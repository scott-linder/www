---
title: CSAW 2015 Sharpturn
date: 2015-09-21
---

The 400 point Forensics problem in the 2015 CSAW CTF looked lonely with only 4
solves. It was named `sharpturn` and the description was "I think my SATA
controller is dying."

## Taking a look around ##

I downloaded and unpacked the archive, which contained a single `sharpturn`
directory. I changed into the directory, and immediately noticed it was a bare
git repo (it is obvious from the contents, but my shell prompt also parses `git
status` if it determines I'm in a repo).

I backed out and cloned the bare repo to get a working directory. The HEAD
contained two files: `Makefile` and `sharp.cpp`. Naturally I tried to `make`,
but was (not unexpectedly) met with errors. To my surprise there was only one.
It was seemingly a typo of `flag`, which was a valid identifier, so I tried
correcting it. It compiled and ran. I followed the prompts without incident,
until I came to the question: `Part5: Input the two prime factors of the number
270031727027.` A quick wolfram alpha search for `prime factorization of
270031727027` produced four numbers. Clearly we are not there yet. For 400
points I can only expect as much.

    $ cd csaw2015/sharpturn
    $ ls
    sharpturn.tar.xz
    $ unxz sharpturn.tar.xz
    $ tar xf sharpturn.tar
    $ ls
    sharpturn/  sharpturn.tar
    $ cd sharpturn/
    $ ls
    branches/  config  description  HEAD  hooks/  info/  objects/  refs/
    $ cd ..
    $ git clone sharpturn local
    Cloning into 'local'...
    done.
    $ cd local/
    $ ls
    Makefile  sharp.cpp
    $ make
    g++ -O2 -g -Wall -Wextra -Wshadow -std=c++11 -lcrypto -o sharp sharp.cpp
    sharp.cpp: In function ‘int main(int, char**)’:
    sharp.cpp:95:11: error: ‘lag’ was not declared in this scope
      cout << &lag;
               ^
    Makefile:6: recipe for target 'ALL' failed
    make: *** [ALL] Error 1
    $ sed -i 's/&lag/flag/' sharp.cpp
    $ make
    g++ -O2 -g -Wall -Wextra -Wshadow -std=c++11 -lcrypto -o sharp sharp.cpp
    $ ./sharp
    Part1: Enter flag:
    flag
    Part2: Input 51337:
    51337
    Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194
    dope
    Part4: C.R.E.A.M. Get da _____:
    money
    Part5: Input the two prime factors of the number 270031727027.
    gtfo
    fish: “./sharp” terminated by signal SIGFPE (Floating point exception)
    $

## Git schwifty ##

The only hints we have so far are:

1. The problem involves some sort of corruption. The description complains of a
faulty SATA controller, and we had to fix a "typo" to even get the source to
compile.
2. We are working out of a git repo. The problem was certain to make this
obvious by giving us a bare repo.
3. The last prompt is obviously corrupted in some way, as 270031727027 does not
have only two prime factors.

When speaking of corruption and git the first thing that springs to mind
is that git addresses content by its SHA-1 hash, so corruption can never go
unnoticed.

A quick `git fsck` sheds some light on the situation:

    $ git fsck
    Checking object directories: 100% (256/256), done.
    error: sha1 mismatch 354ebf392533dce06174f9c8c093036c138935f3
    error: 354ebf392533dce06174f9c8c093036c138935f3: object corrupt or missing
    error: sha1 mismatch d961f81a588fcfd5e57bbea7e17ddae8a5e61333
    error: d961f81a588fcfd5e57bbea7e17ddae8a5e61333: object corrupt or missing
    error: sha1 mismatch f8d0839dd728cb9a723e32058dcc386070d5e3b5
    error: f8d0839dd728cb9a723e32058dcc386070d5e3b5: object corrupt or missing
    missing blob 354ebf392533dce06174f9c8c093036c138935f3
    missing blob f8d0839dd728cb9a723e32058dcc386070d5e3b5
    missing blob d961f81a588fcfd5e57bbea7e17ddae8a5e61333

Now we want to know what kind of objects are corrupt,

    $ git rev-list --objects --all | grep \
        -e 354ebf392533dce06174f9c8c093036c138935f3 \
        -e d961f81a588fcfd5e57bbea7e17ddae8a5e61333 \
        -e f8d0839dd728cb9a723e32058dcc386070d5e3b5
    f8d0839dd728cb9a723e32058dcc386070d5e3b5 sharp.cpp
    d961f81a588fcfd5e57bbea7e17ddae8a5e61333 sharp.cpp
    354ebf392533dce06174f9c8c093036c138935f3 sharp.cpp

By some stroke of luck the SATA controller only dislikes blobs for `sharp.cpp`.

## Flipping bits ##

At this point I realized it should be as simple as modifying these blobs
in-place until their hash actually matches their name. The only question is
what errors are present. I assume, for the sake of the problem being tractable,
that each commit introduces one new error, which effects only a single
character (a single byte, in this case).

From here there is some uninteresting inspection of commit and tree objects
to determine in what order these commits appear in the current tree of `master`
as I suspect each subsequent commit adds errors, so it is advantageous to begin
with the first and proceed from there.

While looking around I note that there is one more object containing
`sharp.cpp`: `efda2f556de36b9e9e1d62417c5f282d8961e2f8`. This is the first
commit of the file, and is not corrupt.

## "It's getting better!" ##

The first corrupt blob is as follows:

    $ git cat-file -p 354ebf392533dce06174f9c8c093036c138935f3
    #include <iostream>
    #include <string>
    #include <algorithm>

    using namespace std;

    int main(int argc, char **argv)
    {
        (void)argc; (void)argv; //unused

        std::string part1;
        cout << "Part1: Enter flag:" << endl;
        cin >> part1;

        int64_t part2;
        cout << "Part2: Input 51337:" << endl;
        cin >> part2;

        std::string part3;
        cout << "Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194" << endl;
        cin >> part3;

        std::string part4;
        cout << "Part4: C.R.E.A.M. Get da _____: " << endl;
        cin >> part4;

        return 0;
    }

If we diff this with it's (uncorrupt) ancestor we see the lines which could
contain the error:

    $ diff (git cat-file -p efda2f556de36b9e9e1d62417c5f282d8961e2f8 | psub) \
           (git cat-file -p 354ebf392533dce06174f9c8c093036c138935f3 | psub)
    14a15,17
    >       int64_t part2;
    >       cout << "Part2: Input 51337:" << endl;
    >       cin >> part2;
    15a19,25
    >       std::string part3;
    >       cout << "Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194" << endl;
    >       cin >> part3;
    >
    >       std::string part4;
    >       cout << "Part4: C.R.E.A.M. Get da _____: " << endl;
    >       cin >> part4;

I was stuck on this for longer than I care to admit. My friend flay leaned over
and told me 51337 isn't leet enough.

    $ git cat-file -p 354ebf392533dce06174f9c8c093036c138935f3 \
        | sed 's/51337/31337/' \
        | git hash-object --stdin -w
    354ebf392533dce06174f9c8c093036c138935f3

Thank you flay <3

    $ git cat-file -p 354ebf392533dce06174f9c8c093036c138935f3 \
        | sed 's/51337/31337/' > sharp.cpp
    $ rm -f .git/objects/35/4ebf392533dce06174f9c8c093036c138935f3
    $ cat sharp.cpp | git hash-object --stdin -w
    354ebf392533dce06174f9c8c093036c138935f3

## "There's only two factors. Don't let your calculator lie." ##

The second corrupt blob is as follows:

    $ git cat-file -p d961f81a588fcfd5e57bbea7e17ddae8a5e61333
    #include <iostream>
    #include <string>
    #include <algorithm>

    using namespace std;

    int main(int argc, char **argv)
    {
        (void)argc; (void)argv; //unused

        std::string part1;
        cout << "Part1: Enter flag:" << endl;
        cin >> part1;

        int64_t part2;
        cout << "Part2: Input 51337:" << endl;
        cin >> part2;

        std::string part3;
        cout << "Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194" << endl;
        cin >> part3;

        std::string part4;
        cout << "Part4: C.R.E.A.M. Get da _____: " << endl;
        cin >> part4;

        uint64_t first, second;
        cout << "Part5: Input the two prime factors of the number 270031727027." << endl;
        cin >> first;
        cin >> second;

        uint64_t factor1, factor2;
        if (first < second)
        {
            factor1 = first;
            factor2 = second;
        }
        else
        {
            factor1 = second;
            factor2 = first;
        }

        return 0;
    }

A diff with its ancestor is:

    $ diff (git cat-file -p 354ebf392533dce06174f9c8c093036c138935f3 | psub) \
           (git cat-file -p d961f81a588fcfd5e57bbea7e17ddae8a5e61333 | psub)
    26a27,43
    >       uint64_t first, second;
    >       cout << "Part5: Input the two prime factors of the number 270031727027." << endl;
    >       cin >> first;
    >       cin >> second;
    >
    >       uint64_t factor1, factor2;
    >       if (first < second)
    >       {
    >               factor1 = first;
    >               factor2 = second;
    >       }
    >       else
    >       {
    >               factor1 = second;
    >               factor2 = first;
    >       }
    >

As a rough estimate of how difficult this would be if we did not assume only
one character has been corrupted, we would have to test every 12 digit number,
or somewhere close to 10^12 possibilities.

If we consider only one digit has changed, it is as simple as 10×12
possibilities. I did not get clever here:

    $ for f in (echo {1,2,3,4,5,6,7,8,9}70031727027 \
                     2{1,2,3,4,5,6,7,8,9}0031727027 \
                     27{1,2,3,4,5,6,7,8,9}031727027 \
                     270{1,2,3,4,5,6,7,8,9}31727027 \
                     2700{1,2,3,4,5,6,7,8,9}1727027 \
                     27003{1,2,3,4,5,6,7,8,9}727027 \
                     2700312{1,2,3,4,5,6,7,8,9}7027 \
                     27003172{1,2,3,4,5,6,7,8,9}027 \
                     270031727{1,2,3,4,5,6,7,8,9}27 \
                     2700317270{1,2,3,4,5,6,7,8,9}7 \
                     27003172702{1,2,3,4,5,6,7,8,9} | tr ' ' \n); \
        if git cat-file -p d961f81a588fcfd5e57bbea7e17ddae8a5e61333 \
                | sed -e 's/51337/31337/' -e "s/270031727027/$f/" \
                | git hash-object --stdin \
                | grep -q d961f81a588fcfd5e57bbea7e17ddae8a5e61333;
            echo $f;
        end;
    end
    272031727027

We have a winner. Wolfram alpha reports the factors are 31357 and 8675311.

    $ git cat-file -p d961f81a588fcfd5e57bbea7e17ddae8a5e61333 \
        | sed -e 's/51337/31337/' -e "s/270031727027/272031727027/" >sharp.cpp
    $ rm -f .git/objects/d9/61f81a588fcfd5e57bbea7e17ddae8a5e61333
    $ cat sharp.cpp | git hash-object --stdin -w
    d961f81a588fcfd5e57bbea7e17ddae8a5e61333

## "All done now! Should calculate the flag..assuming everything went okay." ##

We already know the final correction.

    $ git cat-file -p f8d0839dd728cb9a723e32058dcc386070d5e3b5 \
        | sed -e 's/51337/31337/' \
              -e "s/270031727027/272031727027/" \
              -e 's/&lag/flag/' \
        | git hash-object --stdin
    f8d0839dd728cb9a723e32058dcc386070d5e3b5
    $ git cat-file -p f8d0839dd728cb9a723e32058dcc386070d5e3b5 \
    | sed -e 's/51337/31337/' \
          -e "s/270031727027/272031727027/" \
          -e 's/&lag/flag/' >sharp.cpp
    $ rm -f .git/objects/f8/d0839dd728cb9a723e32058dcc386070d5e3b5
    $ cat sharp.cpp | git hash-object --stdin -w
    f8d0839dd728cb9a723e32058dcc386070d5e3b5

A quick sanity check of `git fsck` returns no errors.

## The flag ##

    $ make
    g++ -O2 -g -Wall -Wextra -Wshadow -std=c++11 -lcrypto -o sharp sharp.cpp
    $ ./sharp
    Part1: Enter flag:
    flag
    Part2: Input 31337:
    31337
    Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194
    sick
    Part4: C.R.E.A.M. Get da _____:
    money
    Part5: Input the two prime factors of the number 272031727027.
    31357
    8675311
    flag{3b532e0a187006879d262141e16fa5f05f2e6752}
    $
