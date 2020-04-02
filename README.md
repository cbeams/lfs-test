## Git LFS sandbox

This repository demonstrates the virtues of using Git LFS in situations where checking in multiple versions of large binary files is unavoidable. If you clone this repository right now, you'll notice that it is very lightweight, about 200K in size, despite the fact that its history contains versions of binary files much larger than that. This is because we're storing these binary objects outside the repository using Git LFS.


Here's how this repository was created per the instructions at https://git-lfs.github.com:

    brew install git-lfs
    git lfs install
    git lfs track '*.pdf'
    git add .gitattributes
    git add README.md
    git add sample.pdf
    git commit -m"Add 180K Git LFS-tracked PDF"
    git push

Notice the `Uploading LFS objects` line in the output of `git push`. The 180K sample.pdf file is being uploaded to GitHub's LFS server here:

    $ git push
    Uploading LFS objects: 100% (1/1), 184 KB | 0 B/s, done.
    [...]

Now, let's replace the 180K sample.pdf file with a larger 14M version:

    $ du -shx sample.pdf
     180K   sample.pdf
    $ cp /some/large.pdf sample.pdf
    $ du -shx sample.pdf
     14M    sample.pdf
    $ git add sample.pdf
    $ git push
    Uploading LFS objects: 100% (1/1), 14 MB | 0 B/s, done.
    [..]

Finally, let's restore the old 180K sample.pdf:

    $ du -shx sample.pdf
     14M    sample.pdf
    $ cp /original/small.pdf sample.pdf
    $ du -shx sample.pdf
     180K   sample.pdf
    $ git add sample.pdf
    $ git push
    Uploading LFS objects: 100% (1/1), 184K | 0 B/s, done.
    [..]

Now let's create a fresh clone of the repository in a different directory:

    $ time git clone cbeams/lfs-test
    Cloning into 'lfs-test'...
    remote: Enumerating objects: 20, done.
    remote: Counting objects: 100% (20/20), done.
    remote: Compressing objects: 100% (15/15), done.
    remote: Total 20 (delta 4), reused 19 (delta 3), pack-reused 0
    Receiving objects: 100% (20/20), 6.52 KiB | 3.26 MiB/s, done.
    Resolving deltas: 100% (4/4), done.

    real    0m6.437s

Notice that the size of the repository is very small and the time to clone it is very fast because it includes only the latest, smaller version of the PDF. If we had not used Git LFS to track this file, the repository would contain the complete history of the PDF, balooning the size to more than 14 megabytes, making cloning the repository unnecessarily cumbersome and time-consuming:

    $ du -shx lfs-test
    504K    lfs-test

    $ du -shx lfs-test/.git
    316K    lfs-test/.git

    $ du -shx lfs-test/.git/lfs/
    196K    lfs-test/.git/lfs/

Of course, if the user wants to get back to the larger version of the PDF, they can do so, and they will incur the cost of downloading only at that time (note that the update takes 18 seconds, because it has to actually download the file from LFS):

    $ time git checkout f768e45
    [..]
    HEAD is now at f768e45 Replace 180K sample.pdf with 14M version

    real    0m18.084s

At this point, our Git database includes both versions of the file and is much larger because of it:

    $ du -shx .git
    14M    .git


If we check out master once again, we'll see that the size of the Git database remains the same.

    $ git checkout master
    $ du -shx .git
    14M    .git

In this way, Git LFS allows us to track arbitrarily large binary files without blowing up the size of the repository to unweildy levels. Devs can opt-in to downloading older versions as necessary, but initial clone times are kept as fast as possible, and local Git history operations, e.g. `git log -S` can remain fast because they do not have to process all the old massive history of those binary objects.
