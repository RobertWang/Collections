> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@vs28031996/remove-git-history-with-bfg-repo-cleaner-866808826eea)

> BFG is faster and simpler way for Removing Big Files, Passwords, Credentials & other private data. It......

![](https://miro.medium.com/max/1400/1*xjoHGUDDW4g8SQmqLIMxSA.jpeg)Remove your GIT History

BFG is faster and simpler way for **Removing Big Files, Passwords, Credentials & other private data.** It will rewrite the complete commit history i.e. all the commit hash, branch/tags refs will get changed without interfering with commit message and branch names.

**Download BFG**

BFG: [https://rtyley.github.io/bfg-repo-cleaner/](https://rtyley.github.io/bfg-repo-cleaner/)

Download the JAR and put it in the same folder where you are going to clone your bare/mirror repo — `/Users/Vinay/your-git-repo.git` (Put it inside folder ‘Vinay’).

You need to have at least **Java 7** installed on your computer in order for this to work.

**Clean latest commit and push**

**The BFG tool does not touch the latest commit**, so you must first remove all the secrets from the repo, such that the latest commit is a clean commit (free of any secrets.)

**Backup your GitHub repository first**

Mirror Clone your repo as backup if things go ugly and rename it as **your-git-repo-backup.git**

**Clone Repository**

Mirror-clone/bare-clone (Learn the difference if you don’t know) your repository.

`git clone --mirror path-to-your-git-repo.git`

![](https://miro.medium.com/max/1400/1*f7FrwAvmqlfXUuGR6SP5dw.png)BFG command options

**To Strip Large blobs**

To delete all files which are more than 10Mb in size

```
java -jar bfg.jar --strip-blobs-bigger-than 10M your-git-repo.git

```

**For password text cleanup**

Create a file called`replacements.txt` and add to it all the passwords you want BFG to remove (one per line) . The `replacements.txt` file should contain all the substitutions you want to do, in a format like this (one entry per line - note the comments shouldn't be included):

```
PASSWORD1 #Replace literal string 'PASSWORD1' with '***REMOVED***' (default)
PASSWORD2==>examplePass         #Replace with 'examplePass' instead
PASSWORD3==>                    #Replace with the empty string
regex:password=\w+==>password=  #Replace, using a regex
regex:\r(\n)==>$1               #Replace Windows newlines with Unix newlines

```

Remove unwanted passwords with BFG:

`java -jar bfg.jar - — replace-text replacements.txt your-git-repo.git`

Remove unwanted passwords on specific files -

`java -jar bfg.jar -fi *.php - — replace-text replacements.txt your-git-repo.git`

**To Remove file from history**

Remove unwanted files with BFG:

`java -jar bfg.jar - — delete-files file-to-be-deleted.php your-git-repo.git`

`java -jar bfg.jar -— delete-files {*.config,*.xml} your-git-repo.git`

Note that you can also specify a wildcard to match multiple files (e.g. file*.txt will match fileA.txt, fileB.txt, file12345.txt, etc.).

**To remove Folder from history**

Remove unwanted folders with BFG:

`java -jar bfg.jar - — delete-folders {OldConfigsFolder} your-git-repo.git`

**Force git to garbage-collect inside the mirror repo**

`cd your-git-repo.git`

`git reflog expire — expire=now — all && git gc — prune=now — aggressive`

**After this, you need to run**

`git push -f`

```
These are your protected commits, and so their contents will NOT be altered:commit 0afd3c44 (protected by ‘HEAD’) — contains 1 dirty files :dr/content/some-file.csv (198 KB)

```

*   If you see something like this while running any BFG command, it means that you don’t have a clean HEAD branch. Best thing would be to make a manual commit of clean that file(remove credentials from that file) OR you can use — `--no-blob-protection` option with the BFG command.
*   If you’re wondering why BFG is not cleaning files larger than 1MB in size, the reason is **BFG only looks at text files under 1MB by default**. You can use `-fs <size in MB,KB,GB>`option with the command.

> -fs, --filter-content-size-threshold <size>  
> only do file-content filtering on files smaller than <size> (default is 1048576 bytes)

Hope, this will help someone. Cheers!!

Connect with **me** on [LinkedIn](https://www.linkedin.com/in/vinayrvsharma)