## What is Git?

* Distributed Version Control System (DVCS)
* Stream of snapshots
* Operations are local, well almost 

## Distributed v/s Centralised VCS

One of the most popular Centralised VCS used at present is Perforce. Whilst it's easier to maintain than local VCS, the server quite literally is the single point of failure. 
What if the central sever is down for some time? Or worse, what if the hard disk the central DB is on gets crashed? You're left with absolutely nothing but the single snapshots that people might have on their local machines.

![DVCS](https://content.intland.com/hubfs/Imported_Blog_Media/version-control-repository-codeBeamer-ALM-software-336x336.png)

Distributed VCS to the rescue. With DVCS such as Git, we mirror the repository (entire history) and not just the latest snapshot of the files, thus, tomorrow if the server (via which the collaboration is being done) were to go down it's as easy as uploading any of the client repo to the server to restore it.

## Snapshots v/s Differences

What sets Git apart is the way it looks at the data. Most of the VCSs out there store information as a set of files and changes made to them over time. This is technically referred to as delta-based VC.

Git does it differently and in a very intelligent way. It saves a reference to the snapshot (picture of what all the files look like) each time the changes are committed and if it finds that a particular file hasn't changed it doesn't store the file again rather it stores a link to the previous identical version. Data to git is like stream of snapshots.
How does Git store data?

Git makes use of [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1) (a 40-character string composed of hexadecimal characters (0–9 and a–f) and calculated based on the contents of a file or directory structure in Git) to ensure data integrity. It stores changes in DB by hash value of its content and not the file name. Worried of SHA collision? Give [this](https://www.git-scm.com/book/en/v2/Git-Tools-Revision-Selection) article a read and the very thought would cease to exist.

## Everything is Local

A small thing such as having project history at your disposal locally can make things drastically different. No need to resort to the server in order to see how a file looked like 2 months down the line.  Great, right? How about when you're offline? Well, Git unlike other VCSs has you covered. Work stress free and save all your work locally and upload it to the server when you're back online. This is definitely a big respite coming from the Perforce world where you literally can't do much unless you're connected to the server.

## The Three States

At any given time a file in Git could be in either of these states:
* committed => Data is safely stored in the local DB
* modified => file changed, but not committed
* staged => marked the current version of the modified file to be pushed in the commit snapshot

![Figure](https://r-bio.github.io/img/git_areas.png)

The fig. above captures the 3 main sections of a Git project namely,

**Git Directory:**
* copied when we clone a repository
* contains metadata and object DB of the project

**Working Tree:**
* single checkout of one version of the project
* files are pulled out of Git directory and placed on disk for us to modify

**Staging Area:**
* A file that resides in Git Directory and stores info about what needs to be pushed in the next commit

## Git Basics

### Initialising a Git Repository

cd to the directory you wanna start tracking and then run "git init"
What does this do?
* creates a sub directory named .git which has various folders inside it:
  * **objects:** stores different Git Objects namely, 
    * blob: similar to a file in unix parlance
    * tree: similar to folder
    * commit: specifies top level tree of the snapshot, commit author, timestamp and commit message
    * tag: generally points to a commit; never moves, points to the same commit (refs/tags)
  * **refs:** help us track history from a particular commit. 3 main refs are there:
    * branch: reference to the head (branch: refs/heads/<branch_name>) 
    * tag
    * remote: refs/remote/<remote_name>/<branch_name>; stores the value we last pushed in the remote for each branch
  * **HEAD:** symbolic reference to the branch we're currently on
  * **Packfiles:** created when "git gc" command is run (could be done manually) or when we push to the remote server; saves the
  difference between 2 almost similar file versions as delta rather than maintaining separate versions each time. Latest
  version is kept intact and the earlier version stored as delta to ensure faster access to the revised version.
  * **config:**
    * Refspec: used for fetching content from remote to local repo
      * Pattern: +<src>:<dst>; <src> => pattern for references on remote side; <dst> => where those references will be tracked locally; "+" tells git to update the ref even if it's not fast-forward
  
### Basic Git Commands:

**Clone a Repo:** command => git clone <url> <target_directory>

**Track status of a file:**

![file state](https://git-scm.com/book/en/v2/images/lifecycle.png)
* A file could be either tracked or untracked. A tracked file could be either modified or unmodified.
command => "git status" 
* To add an untracked file => git add <file_name>
* "git add" can be used to start tracking new files, to stage modified files and to mark merge-conflicted files as resolved
* Note: If you change a file after running git add you need to run git add again, otherwise, the file would show up in both staged and unstaged state.

**Ignoring Files:** 
Create a .gitignore file. We can use standard [glob patterns](http://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm) to specify the files we want to ignore. [Gitignore examples](https://github.com/github/gitignore).

**Viewing Changes:**
Two kind of changes that one could be interested in:
Staged but not committed. Command => git diff --staged
Not yet staged. Command => git diff
While the former would compare the staged changes to the last commit (what's gonna go in the next commit) the latter would compare what's in the working directory against the staging area. 

**Committing changes:**
Every time we make a commit we essentially take a snapshot of the project which we can revert to or compare to later.
Command => git commit -m "commit_message"

Skipping the staging area: If we don't want to explicitly specify which files should make it to the next commit and simply commit the files that have been staged since the last commit we can skip the staging step by replacing -m with -am in the above command. -a option automatically stages every tracked file.
git commit -am "commit_message" 

**Removing Files**

* From staging area: git rm <file_name>
* From staging area, but not the working directory: git rm --cached <file_name>

**Moving Files:**
"git rm" command helps us take care of file renaming. What would usually take 3 commands (mv <old_filename> <new_filename>, git rm <old_filename>, git add <new_filename>) is done single handedly via git rm <old_filename> <new_filename> 

**Viewing Commit History:**
* git log
* git log --pretty=oneline
* git log -p => Would show diff below each commit; p stands for patch

**Undoing Things:**
git commit --amend (To add changes to the last commit like a file you forgot to stage; to change commit message of the last commit)

Unstage a staged file: git reset HEAD <file_name>
How does reset work?
It modifies the three trees namely, HEAD (last commit snapshot), staging/index and working directory in a specific order and stops where we instruct it to.
* Move the branch HEAD points to (git reset --soft)
* Make the index look like HEAD (git reset --mixed)
* Make the working dir look like index (git reset --hard)

Unmodify a modified file: git checkout -- <file_name>
This replaces file with the recent committed version.

**Working with Remotes:**

To pull changes from a remote: git pull <remote_name> <branch_name>
To push changes to remote: git push <remote_name> <branch_name>

**Tagging:**

To list all the existing tags: git tag
With wildcard: git tag -l "v1.*"

Tags could be of following 2 types:

* **Lightweight:** pointer to a specific commit (like a branch that doesn't change)
* **Annotated:** stored as objects in git DB
git tag -a v1.2 <commit_id>
Tags are not pushed by default to the remote server. The command to do the same is:
git push <remote_name> --tags
This would push all the tags that are not there in the central repo immaterial of their type.

Delete the tag: git tag -d <tag_name> (doesn't remove from the remote server)
Delete a remote tag: git push origin --delete <tag_name>

## Git Aliases

Git doesn't automatically infer our command if we type them in partially. We can setup alias for each command using git config. Below are few examples:

git config --global alias.co checkout
git config --global alias.st status

To see the last commit you could use something like:
git config --global alias.last 'log -1 HEAD'


### References:

* [Git](https://git-scm.com/) website
* [Pro Git](https://git-scm.com/book/en/v2) by Scott Chacon and Ben Straub

