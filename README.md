This repository generates a history of changes to binary files to test disk consumption for git repositories with and without Git LFS installed.

## Context

This repository and the resulting tests were created in support of a [blog post](http://blog.testdouble.com/posts/2017-01-25-how-and-why-to-use-git-lfs.html) explaining the how and why of using Git LFS.

## Conclusions

- Pro-LFS
  - Local disk usage with LFS installed is _significantly lower_. This effect becomes more extreme as files tracked by Git LFS are changed.
  - Pushing histories with multiple updates to files tracked by Git LFS is _much faster_ with LFS installed.
- Anti-LFS
  - Local disk usage with LFS installed is _higher_ for the machine making the changes. This was unexpected, and time prevented exploring this further.
  - Committing binary files is _slower_ with LFS installed. This is to be expected, considering the time required for the Git LFS filters to run.

## Method

The `run` script is a Ruby program that generates:

- A specific number of binary files...
- Containing a specific number of bytes (of noise)...
- Regenerating them a specific number of times.

After each of these changes, the script creates an all-emcompassing git commit. By the time the script is finished, there should be a precise history of changed binary files we can (outside the `run` script):

- Upload to an Git-LFS-enabled server (e.g. GitHub).
- Clone that repo to a fresh client.
- Measure the resulting fresh clone's disk utilization, and compare.

This method was used with and without Git LFS installed (Using a `git reset` to wipe out the Git LFS `.gitattributes` file). The exact steps and results are included below.

For posterity, the two target repositories are [here](https://github.com/Schoonology/git-lfs-size-test) and [here](https://github.com/Schoonology/git-no-lfs-size-test).

## Steps and results for 10 2KB files, committed 40 times

In summary:

- Disk utilization:
  - Savings: 700KB, a reduction of 72%.
  - Without Git LFS, a fresh clone uses 928KB in the `.git` directory, and 972KB in total.
  - With Git LFS, a fresh clone uses 224KB in the `.git` directory, and 272KB in total.
- Time:
  - Cost: 102.4s, an increase of 1013%.
  - With Git LFS, generating history took 19.9s, uploading took 1m9.4s, and downloading took 23.2s. 112.5s total.
  - Without Git LFS, generating history took 3.1s, uploading took 3.9s, and downloading took 3.1s. 10.1s total.

### Generating a history with LFS

```
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git pull --tags git@github.com:Schoonology/git-lfs-size-test-generator.git
remote: Counting objects: 18, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
Unpacking objects: 100% (18/18), done.
From github.com:Schoonology/git-lfs-size-test-generator
 * branch            HEAD       -> FETCH_HEAD
 * [new tag]         lfs        -> lfs
 * [new tag]         no-lfs     -> no-lfs
✔ (master) git-lfs-size-child$ time ./run 10 2048 40

real  0m19.902s
user  0m5.776s
sys 0m13.158s
✔ (master) git-lfs-size-child$ time git push -f git@github.com:Schoonology/git-lfs-size-test.git
Git LFS: (400 of 400 files) 800.00 KB / 800.00 KB
Counting objects: 538, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (535/535), done.
Writing objects: 100% (538/538), 73.31 KiB | 0 bytes/s, done.
Total 538 (delta 6), reused 0 (delta 0)
remote: Resolving deltas: 100% (6/6), done.
To github.com:Schoonology/git-lfs-size-test.git
 + 80fe4aa...75d4794 master -> master (forced update)

real  1m9.384s
user  0m2.443s
sys 0m7.366s
```

### Generating a history without LFS

```
✔ (master) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git pull --tags git@github.com:Schoonology/git-lfs-size-test-generator.git
remote: Counting objects: 18, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
Unpacking objects: 100% (18/18), done.
From github.com:Schoonology/git-lfs-size-test-generator
 * branch            HEAD       -> FETCH_HEAD
 * [new tag]         lfs        -> lfs
 * [new tag]         no-lfs     -> no-lfs
✔ (master) git-lfs-size-child$ git reset --hard no-lfs
HEAD is now at 3173d31 Trim script to generating files.
✔ (master) git-lfs-size-child$ time ./run 10 2048 40

real  0m3.057s
user  0m1.248s
sys 0m2.308s
✔ (master) git-lfs-size-child$ time git push -f git@github.com:Schoonology/git-no-lfs-size-test.git
Counting objects: 535, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (532/532), done.
Writing objects: 100% (535/535), 830.69 KiB | 0 bytes/s, done.
Total 535 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), done.
To github.com:Schoonology/git-no-lfs-size-test.git
 + f89d87c...86fc247 master -> master (forced update)

real  0m3.864s
user  0m0.146s
sys 0m0.080s
```

### Pulling and measuring disk utilization without Git LFS

```
✔ (master) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ time git pull git@github.com:Schoonology/git-no-lfs-size-test.git
remote: Counting objects: 535, done.
remote: Compressing objects: 100% (528/528), done.
remote: Total 535 (delta 4), reused 535 (delta 4), pack-reused 0
Receiving objects: 100% (535/535), 830.69 KiB | 420.00 KiB/s, done.
Resolving deltas: 100% (4/4), done.
From github.com:Schoonology/git-no-lfs-size-test
 * branch            HEAD       -> FETCH_HEAD

real  0m3.138s
user  0m0.071s
sys 0m0.124s
✔ (master) git-lfs-size-child$ du -hc
 44K  ./.git/hooks
4.0K  ./.git/info
4.0K  ./.git/logs/refs/heads
4.0K  ./.git/logs/refs
8.0K  ./.git/logs
  0B  ./.git/objects/info
848K  ./.git/objects/pack
848K  ./.git/objects
4.0K  ./.git/refs/heads
  0B  ./.git/refs/tags
4.0K  ./.git/refs
928K  ./.git
 40K  ./binaries
972K  .
972K  total
```

### Pulling and measuring disk utilization with Git LFS

```
✔ (master #%) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git remote add origin git@github.com:Schoonology/git-lfs-size-test.git
✔ (master #) git-lfs-size-child$ time git pull origin master
remote: Counting objects: 538, done.
remote: Compressing objects: 100% (529/529), done.
remote: Total 538 (delta 6), reused 538 (delta 6), pack-reused 0
Receiving objects: 100% (538/538), 73.31 KiB | 0 bytes/s, done.
Resolving deltas: 100% (6/6), done.
From github.com:Schoonology/git-lfs-size-test
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
Downloading binaries/0.dat (2.00 KB)
Downloading binaries/1.dat (2.00 KB)
Downloading binaries/2.dat (2.00 KB)
Downloading binaries/3.dat (2.00 KB)
Downloading binaries/4.dat (2.00 KB)
Downloading binaries/5.dat (2.00 KB)
Downloading binaries/6.dat (2.00 KB)
Downloading binaries/7.dat (2.00 KB)
Downloading binaries/8.dat (2.00 KB)
Downloading binaries/9.dat (2.00 KB)
Checking out files: 100% (14/14), done.

real  0m23.206s
user  0m0.965s
sys 0m0.668s
✔ (master) git-lfs-size-child$ du -hc
 48K  ./.git/hooks
4.0K  ./.git/info
4.0K  ./.git/lfs/objects/0f/09
4.0K  ./.git/lfs/objects/0f
4.0K  ./.git/lfs/objects/1d/8f
4.0K  ./.git/lfs/objects/1d
4.0K  ./.git/lfs/objects/3d/3b
4.0K  ./.git/lfs/objects/3d
4.0K  ./.git/lfs/objects/4c/59
4.0K  ./.git/lfs/objects/4c
4.0K  ./.git/lfs/objects/51/1c
4.0K  ./.git/lfs/objects/51
4.0K  ./.git/lfs/objects/8d/f4
4.0K  ./.git/lfs/objects/8d
4.0K  ./.git/lfs/objects/a1/de
4.0K  ./.git/lfs/objects/a1
4.0K  ./.git/lfs/objects/b5/81
4.0K  ./.git/lfs/objects/b5
4.0K  ./.git/lfs/objects/b7/2e
4.0K  ./.git/lfs/objects/b7
4.0K  ./.git/lfs/objects/da/e0
4.0K  ./.git/lfs/objects/da
  0B  ./.git/lfs/objects/incomplete
  0B  ./.git/lfs/objects/logs
 40K  ./.git/lfs/objects
  0B  ./.git/lfs/tmp/objects
  0B  ./.git/lfs/tmp
 40K  ./.git/lfs
4.0K  ./.git/logs/refs/heads
4.0K  ./.git/logs/refs/remotes/origin
4.0K  ./.git/logs/refs/remotes
8.0K  ./.git/logs/refs
 12K  ./.git/logs
  0B  ./.git/objects/info
 92K  ./.git/objects/pack
 92K  ./.git/objects
4.0K  ./.git/refs/heads
4.0K  ./.git/refs/remotes/origin
4.0K  ./.git/refs/remotes
  0B  ./.git/refs/tags
8.0K  ./.git/refs
224K  ./.git
 40K  ./binaries
272K  .
272K  total
```

## Steps and results for 1 100MiB file, committed only 1 time

In summary:

- Disk utilization:
  - Savings: 0B, no change. This was, thankfully, expected—the binary file never changed.
  - Without Git LFS, a fresh clone uses 96MB in the `.git` directory, and 191MB in total.
  - With Git LFS, a fresh clone uses 96MB in the `.git` directory, and 191MB in total.
- Time:
  - Cost: 2.5s, an increase of 2%.
  - With Git LFS, generating history took 10s, uploading took 1m23s, and downloading took 29.1s. 122.1s total.
  - Without Git LFS, generating history took 12.9s, uploading took 1m14.7s, and downloading took 32s. 119.6s total.

### Generating a history with LFS

```
✔ (master *) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git pull --tags git@github.com:Schoonology/git-lfs-size-test-generator.git
remote: Counting objects: 18, done.
remote: Compressing objects: 100% (11/11), done.
Unpacking objects: 100% (18/18), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
From github.com:Schoonology/git-lfs-size-test-generator
 * branch            HEAD       -> FETCH_HEAD
 * [new tag]         lfs        -> lfs
 * [new tag]         no-lfs     -> no-lfs
✔ (master) git-lfs-size-child$ time ./run 1 100000000 1

real  0m9.951s
user  0m0.869s
sys 0m9.429s
✔ (master) git-lfs-size-child$ time git push -f git@github.com:Schoonology/git-lfs-size-test.git
Git LFS: (1 of 1 files) 95.37 MB / 95.37 MB
Counting objects: 22, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (19/19), done.
Writing objects: 100% (22/22), 3.00 KiB | 0 bytes/s, done.
Total 22 (delta 5), reused 0 (delta 0)
remote: Resolving deltas: 100% (5/5), done.
To github.com:Schoonology/git-lfs-size-test.git
 + 75d4794...09f33e3 master -> master (forced update)

real  1m23.113s
user  0m1.834s
sys 0m1.987s
```

### Generating a history without LFS

```
✔ (master) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git pull --tags git@github.com:Schoonology/git-lfs-size-test-generator.git
remote: Counting objects: 18, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
Unpacking objects: 100% (18/18), done.
From github.com:Schoonology/git-lfs-size-test-generator
 * branch            HEAD       -> FETCH_HEAD
 * [new tag]         lfs        -> lfs
 * [new tag]         no-lfs     -> no-lfs
✔ (master) git-lfs-size-child$ git reset --hard no-lfs
HEAD is now at 3173d31 Trim script to generating files.
✔ (master) git-lfs-size-child$ time ./run 1 100000000 1

real  0m12.852s
user  0m3.657s
sys 0m9.368s
✔ (master) git-lfs-size-child$ time git push -f git@github.com:Schoonology/git-no-lfs-size-test.git
Counting objects: 19, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (16/16), done.
Writing objects: 100% (19/19), 95.40 MiB | 1.48 MiB/s, done.
Total 19 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), done.
remote: warning: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
remote: warning: See http://git.io/iEPt8g for more information.
remote: warning: File binaries/0.dat is 95.37 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
To github.com:Schoonology/git-no-lfs-size-test.git
 + 86fc247...c7855c4 master -> master (forced update)

real  1m14.696s
user  0m4.190s
sys 0m3.372s
```

### Pulling and measuring disk utilization without Git LFS

```
✔ (master) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ time git pull git@github.com:Schoonology/git-no-lfs-size-test.git
remote: Counting objects: 19, done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 19 (delta 4), reused 19 (delta 4), pack-reused 0
Unpacking objects: 100% (19/19), done.
From github.com:Schoonology/git-no-lfs-size-test
 * branch            HEAD       -> FETCH_HEAD

real  0m32.012s
user  0m5.309s
sys 0m4.962s
✔ (master) git-lfs-size-child$ du -hc
 44K  ./.git/hooks
4.0K  ./.git/info
4.0K  ./.git/logs/refs/heads
4.0K  ./.git/logs/refs
8.0K  ./.git/logs
4.0K  ./.git/objects/03
4.0K  ./.git/objects/11
4.0K  ./.git/objects/1f
4.0K  ./.git/objects/2c
4.0K  ./.git/objects/31
4.0K  ./.git/objects/33
4.0K  ./.git/objects/4a
4.0K  ./.git/objects/6d
4.0K  ./.git/objects/70
4.0K  ./.git/objects/94
 95M  ./.git/objects/9a
4.0K  ./.git/objects/ae
4.0K  ./.git/objects/c7
4.0K  ./.git/objects/d5
4.0K  ./.git/objects/db
4.0K  ./.git/objects/e6
4.0K  ./.git/objects/e8
4.0K  ./.git/objects/f6
4.0K  ./.git/objects/f9
  0B  ./.git/objects/info
  0B  ./.git/objects/pack
 95M  ./.git/objects
4.0K  ./.git/refs/heads
  0B  ./.git/refs/tags
4.0K  ./.git/refs
 96M  ./.git
 95M  ./binaries
191M  .
191M  total
```

### Pulling and measuring disk utilization with Git LFS

```
✔ (master #%) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git remote add origin git@github.com:Schoonology/git-lfs-size-test.git
✔ (master #) git-lfs-size-child$ time git pull origin master
remote: Counting objects: 22, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 22 (delta 5), reused 22 (delta 5), pack-reused 0
Unpacking objects: 100% (22/22), done.
From github.com:Schoonology/git-lfs-size-test
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
Downloading binaries/0.dat (95.37 MB)

real  0m29.057s
user  0m1.410s
sys 0m2.338s
✔ (master) git-lfs-size-child$ du -hc
 48K  ./.git/hooks
4.0K  ./.git/info
 95M  ./.git/lfs/objects/5e/97
 95M  ./.git/lfs/objects/5e
  0B  ./.git/lfs/objects/incomplete
  0B  ./.git/lfs/objects/logs
 95M  ./.git/lfs/objects
  0B  ./.git/lfs/tmp/objects
  0B  ./.git/lfs/tmp
 95M  ./.git/lfs
4.0K  ./.git/logs/refs/heads
4.0K  ./.git/logs/refs/remotes/origin
4.0K  ./.git/logs/refs/remotes
8.0K  ./.git/logs/refs
 12K  ./.git/logs
4.0K  ./.git/objects/09
4.0K  ./.git/objects/11
4.0K  ./.git/objects/1f
4.0K  ./.git/objects/2c
4.0K  ./.git/objects/31
4.0K  ./.git/objects/44
4.0K  ./.git/objects/4a
4.0K  ./.git/objects/57
4.0K  ./.git/objects/6d
4.0K  ./.git/objects/70
4.0K  ./.git/objects/7e
4.0K  ./.git/objects/86
4.0K  ./.git/objects/94
4.0K  ./.git/objects/a6
8.0K  ./.git/objects/ae
4.0K  ./.git/objects/d5
4.0K  ./.git/objects/db
4.0K  ./.git/objects/e6
4.0K  ./.git/objects/e8
4.0K  ./.git/objects/f6
4.0K  ./.git/objects/f9
  0B  ./.git/objects/info
  0B  ./.git/objects/pack
 88K  ./.git/objects
4.0K  ./.git/refs/heads
4.0K  ./.git/refs/remotes/origin
4.0K  ./.git/refs/remotes
  0B  ./.git/refs/tags
8.0K  ./.git/refs
 96M  ./.git
 95M  ./binaries
191M  .
191M  total
```

## Steps and results for 50 3MiB files, committed 25 times each

This test completely failed due to data usage in GitHub. Unbeknownst to me at the time (and contrary to what I understood from [this document](https://help.github.com/articles/what-is-my-disk-quota/)), all GitHub users have a limit on their Git LFS storage. Fortunately, the fix for going over the limit was to delete the offending repository (and, so testing could continue, recreate it), though I did reach out to Support for help.

```
✔ (master) git-lfs-size-child$ caffeinate time git push -f git@github.com:Schoonology/git-lfs-size-test.git
Git LFS: (286 of 1250 files) 824.38 MB / 3.49 GB                                         Connection to github.com closed by remote host.
Git LFS: (766 of 1250 files) 2.51 GB / 3.49 GB
http: Git LFS is disabled for this repository.
Docs: https://github.com/contact
...
http: Git LFS is disabled for this repository.
Docs: https://github.com/contact
batch response: http: This repository is over its data quota. Purchase more data packs to restore access.
Docs: https://help.github.com/articles/purchasing-additional-storage-and-bandwidth-for-a-personal-account/
http: Git LFS is disabled for this repository.
Docs: https://github.com/contact
...
http: Git LFS is disabled for this repository.
Docs: https://github.com/contact
error: failed to push some refs to 'git@github.com:Schoonology/git-lfs-size-test.git'
Command terminated abnormally.
     1862.09 real        62.06 user        94.28 sys
```

## Steps and results for 1 1MiB file, committed 1000 times

In summary:

- Disk utilization:
  - Savings: 188.4MB, a reduction of 98.6%.
  - Without Git LFS, a fresh clone uses 96MB in the `.git` directory, and 191MB in total.
  - With Git LFS, a fresh clone uses 1.6MB in the `.git` directory, and 2.6MB in total.
- Time:
  - Cost: 81.3s, an increase of 68%.
  - With Git LFS, generating history took 2m49.8s, uploading took 24.2s, and downloading took 6.9s. 200.9s total.
  - Without Git LFS, generating history took 12.9s, uploading took 1m14.7s, and downloading took 32s. 119.6s total.

### Generating a history with LFS

I tried multiple times to get a clean `git push`, but it never happened. GitHub regularly dropped the connection while uploading, and even a _single dropped object_ resulted in the entire operation failing. A second `git push` was required, and I've added the two times together in the summary.

```
✔ (master *) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git pull --tags git@github.com:Schoonology/git-lfs-size-test-generator.git
remote: Counting objects: 18, done.
remote: Compressing objects: 100% (11/11), done.
Unpacking objects: 100% (18/18), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
From github.com:Schoonology/git-lfs-size-test-generator
 * branch            HEAD       -> FETCH_HEAD
 * [new tag]         lfs        -> lfs
 * [new tag]         no-lfs     -> no-lfs
✔ (master) git-lfs-size-child$ time ./run 1 1000000 1000

real  2m49.772s
user  0m30.807s
sys 2m22.992s
✔ (master) git-lfs-size-child$ time git push -f git@github.com:Schoonology/git-lfs-size-test.git
Git LFS: (837 of 1000 files) 801.39 MB / 953.67 MB                                       Connection to github.com closed by remote host.
Git LFS: (999 of 1000 files) 953.67 MB / 953.67 MB
http: Post https://api.github.com/lfs/Schoonology/git-lfs-size-test/objects/27d337efd28d8f242d36675cf99a63daf73632573610167713cd396d1e5ee8c5/verify: EOF
error: failed to push some refs to 'git@github.com:Schoonology/git-lfs-size-test.git'

real  11m56.418s
user  0m21.662s
sys 0m39.142s
✘ (master) git-lfs-size-child$ time git push git@github.com:Schoonology/git-lfs-size-test.git
Git LFS: (1 of 1 files, 999 skipped) 976.56 KB / 976.56 KB, 952.72 MB skipped
Counting objects: 4018, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4015/4015), done.
Writing objects: 100% (4018/4018), 497.57 KiB | 0 bytes/s, done.
Total 4018 (delta 40), reused 0 (delta 0)
remote: Resolving deltas: 100% (40/40), done.
To github.com:Schoonology/git-lfs-size-test.git
 * [new branch]      master -> master

real  0m24.216s
user  0m0.723s
sys 0m1.110s
```

### Generating a history without LFS

```
✔ (master) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git pull --tags git@github.com:Schoonology/git-lfs-size-test-generator.git
remote: Counting objects: 18, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
Unpacking objects: 100% (18/18), done.
From github.com:Schoonology/git-lfs-size-test-generator
 * branch            HEAD       -> FETCH_HEAD
 * [new tag]         lfs        -> lfs
 * [new tag]         no-lfs     -> no-lfs
✔ (master) git-lfs-size-child$ git reset --hard no-lfs
HEAD is now at 3173d31 Trim script to generating files.
✔ (master) git-lfs-size-child$ time ./run 1 100000000 1

real  0m12.852s
user  0m3.657s
sys 0m9.368s
✔ (master) git-lfs-size-child$ time git push -f git@github.com:Schoonology/git-no-lfs-size-test.git
Counting objects: 19, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (16/16), done.
Writing objects: 100% (19/19), 95.40 MiB | 1.48 MiB/s, done.
Total 19 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), done.
remote: warning: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
remote: warning: See http://git.io/iEPt8g for more information.
remote: warning: File binaries/0.dat is 95.37 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
To github.com:Schoonology/git-no-lfs-size-test.git
 + 86fc247...c7855c4 master -> master (forced update)

real  1m14.696s
user  0m4.190s
sys 0m3.372s
```

### Pulling and measuring disk utilization without Git LFS

```
✔ (master) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ time git pull git@github.com:Schoonology/git-no-lfs-size-test.git
remote: Counting objects: 19, done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 19 (delta 4), reused 19 (delta 4), pack-reused 0
Unpacking objects: 100% (19/19), done.
From github.com:Schoonology/git-no-lfs-size-test
 * branch            HEAD       -> FETCH_HEAD

real  0m32.012s
user  0m5.309s
sys 0m4.962s
✔ (master) git-lfs-size-child$ du -hc
 44K  ./.git/hooks
4.0K  ./.git/info
4.0K  ./.git/logs/refs/heads
4.0K  ./.git/logs/refs
8.0K  ./.git/logs
4.0K  ./.git/objects/03
4.0K  ./.git/objects/11
4.0K  ./.git/objects/1f
4.0K  ./.git/objects/2c
4.0K  ./.git/objects/31
4.0K  ./.git/objects/33
4.0K  ./.git/objects/4a
4.0K  ./.git/objects/6d
4.0K  ./.git/objects/70
4.0K  ./.git/objects/94
 95M  ./.git/objects/9a
4.0K  ./.git/objects/ae
4.0K  ./.git/objects/c7
4.0K  ./.git/objects/d5
4.0K  ./.git/objects/db
4.0K  ./.git/objects/e6
4.0K  ./.git/objects/e8
4.0K  ./.git/objects/f6
4.0K  ./.git/objects/f9
  0B  ./.git/objects/info
  0B  ./.git/objects/pack
 95M  ./.git/objects
4.0K  ./.git/refs/heads
  0B  ./.git/refs/tags
4.0K  ./.git/refs
 96M  ./.git
 95M  ./binaries
191M  .
191M  total
```

### Pulling and measuring disk utilization with Git LFS

```
✔ (master #%) git-lfs-size-child$ rm -rf .git* *
✔  git-lfs-size-child$ git init
Initialized empty Git repository in /Users/schoon/Projects/sandbox/git-lfs-size-child/.git/
✔ (master #) git-lfs-size-child$ git remote add origin git@github.com:Schoonology/git-lfs-size-test.git
✔ (master #) git-lfs-size-child$ time git pull origin master
remote: Counting objects: 4018, done.
remote: Compressing objects: 100% (3975/3975), done.
remote: Total 4018 (delta 40), reused 4018 (delta 40), pack-reused 0
Receiving objects: 100% (4018/4018), 497.58 KiB | 0 bytes/s, done.
Resolving deltas: 100% (40/40), done.
From github.com:Schoonology/git-lfs-size-test
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
Downloading binaries/0.dat (976.56 KB)

real  0m6.896s
user  0m0.199s
sys 0m0.227s
✔ (master) git-lfs-size-child$ du -hc
 48K  ./.git/hooks
4.0K  ./.git/info
980K  ./.git/lfs/objects/90/14
980K  ./.git/lfs/objects/90
  0B  ./.git/lfs/objects/incomplete
  0B  ./.git/lfs/objects/logs
980K  ./.git/lfs/objects
  0B  ./.git/lfs/tmp/objects
  0B  ./.git/lfs/tmp
980K  ./.git/lfs
4.0K  ./.git/logs/refs/heads
4.0K  ./.git/logs/refs/remotes/origin
4.0K  ./.git/logs/refs/remotes
8.0K  ./.git/logs/refs
 12K  ./.git/logs
  0B  ./.git/objects/info
612K  ./.git/objects/pack
612K  ./.git/objects
4.0K  ./.git/refs/heads
4.0K  ./.git/refs/remotes/origin
4.0K  ./.git/refs/remotes
  0B  ./.git/refs/tags
8.0K  ./.git/refs
1.6M  ./.git
980K  ./binaries
2.6M  .
2.6M  total
```

## Addendum: Earlier version

An earlier version of this script analyzed disk utilization on the local repo _before_ pushing to GitHub. It produced its own interesting results: disk utilization is _slightly higher_ with Git LFS enabled. Exploring these results further informed the final method, described above.

Annotated output (ellipses replacing verbose git output resulting from using git in a "detatched HEAD" state and running `git gc` before all size checks):

```
$ ./run 10 2048 40
Note: checking out 'no-lfs'.
...
HEAD is now at ab53780... Update script to run both tests.
...
# This is the "without LFS" local growth. The .git folder grew 4.1K.
{:repo=>4176, :working=>80}

Warning: you are leaving 40 commits behind, ...
HEAD is now at 1368c3d... Add LFS integration.
...
# This is the "with LFS" local growth. The .git folder grew 7.3K.
# As expected, the working directory grew the same amount in both tests.
{:repo=>7384, :working=>80}

Warning: you are leaving 40 commits behind,...
Switched to branch 'master'
...
$
```
