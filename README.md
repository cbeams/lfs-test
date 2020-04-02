## Git LFS sandbox

Per the instructions at https://git-lfs.github.com:

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
    Counting objects: 5, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (5/5), 1.31 KiB | 445.00 KiB/s, done.
    Total 5 (delta 0), reused 2 (delta 0)
    To github.com:cbeams/lfs-test.git
       edce04c..e723962  master -> master

Now, let's replace the 180K sample.pdf file with a larger 14M version:

    $ du -shx sample.pdf
     180K   sample.pdf
    $ cp /some/large.pdf sample.pdf
    $ du -shx sample.pdf
     14M    sample.pdf
    $ git add sample.pdf
    $ git push
