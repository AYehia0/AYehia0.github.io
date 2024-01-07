# Git for hackers


A practical guide for git with real life scenarios which can make your git experience better.
<!--more-->

# Git for hackers

{{< admonition quote "Once a wise man said:" true >}}
"There are two types of hackers in this world: those who know git, and those who still don't git it" 
{{< /admonition >}}

When I refer to [`hackers`](https://en.wikipedia.org/wiki/Hacker), I mean those who love to go behind the things to discover how things work and break them (somehow). Originally, hacker simply meant advanced computer technology enthusiast (both hardware and software) and adherent of programming subculture who used their skill to achieve a goal or overcome an obstacle.

If you don't use git, you should start learning it now!

## Disclaimer
This isn't a tutorial on how to use git or it's going to make to the best git user but will inspire you to make your git experience better.

If you don't know what's git or version control, you should read this book : [Pro Git](https://git-scm.com/book/en/v2) from git's offical website. 

I also assume you have git installed on your machine, and you have some basic experience with Linux Shell.

{{< admonition note Note >}}
I will update this blog whenever I find something useful to share.
{{< /admonition >}}

## Introduction
In this tutorial, I am going to share some tips and tricks that I have tested that would make it easy to git.

1. Uncommon git commands.
2. Git & Bash aliases.
3. Github Command line tools : `gh`.
4. Markdown everywhere.


### Uncommon git commands
Anyone who ever used git, will know this flow (and yes, with deep understanding of how git works, you won't use other commands).

{{< mermaid >}}
sequenceDiagram
    participant Working Tree
    participant Staging Area
    participant Local Repo
    Working Tree ->>Staging Area: git add
    Staging Area ->>Local Repo: git commit
    Local Repo ->>Staging Area: git reset
    Staging Area ->>Working Tree: git restore --staged
    Local Repo ->>Working Tree: git checkout
    participant Remote
    Local Repo ->>Remote: git push
    Remote ->>Local Repo: git pull
{{< /mermaid >}}

Even though there are some hidden gems that will make your life easier:

#### <b>1. `git stash`</b>
Stash the changes in a dirty working directory away

when you want to record the current state of the working directory and the index, but want to go back to a clean working directory. The command saves your local modifications away and reverts the working directory to match the HEAD commit.

The modifications stashed away by this command can be listed with `git stash list`, inspected with `git stash show`, and restored (potentially on top of a different commit) with `git stash apply`. Calling git stash without any arguments is equivalent to `git stash push`

{{< admonition tip Tip >}}
You can interactively choose which changes to stash when using `git stash -p`
{{< /admonition >}}

#### <b>2. `git cherry-pick`</b>

Apply the changes introduced by some existing commits to your repo.

`git cherry-pick` is a very useful command to use in case you want to have some commits to be picked by reference and added to the current working HEAD.

`git cherry-pick` can be useful for:
- <b>Applying hotfixs</b>: When you want to apply a hotfix in one branch to other branches.
- <b>Selectively merging changes</b>: When you want to merge only specific commits from one branch into another, rather than merging the entire branch.
- <b>Reordering commits</b>: You can use git cherry-pick to reorder commits in the target branch or even apply commits from different parts of the source branch to achieve a specific commit order.

<br>
<p align="center">
    <img src="one-simply-cherry-pick.jpg" width="100%">
</p>

{{< admonition warning "Don't use it if:-" >}}
- <b>When commits depend on each other</b>: If the commits you want to cherry-pick have dependencies on other commits in the source branch, cherry-picking them individually can lead to errors or missing context
- <b>Maintaining a clean commit history</b>: Cherry-picking can lead to a less clean and linear commit history, as it introduces commits from another branch with different commit messages and timestamps.
{{< /admonition >}}


#### <b>2. `git blame`</b>

View the revision history of a file, showing who last modified each line and when. It's a useful tool for tracking changes and identifying the author and commit associated with each line of code in a file.

When `git blame` can be useful:
- <b>Bug Tracking</b>: When trying to identify the source of a bug, you can trace back commits.

<br>
<p align="center">
    <img src="blame.gif" width="50%">
</p>

- <b>Security Auditing</b>: For security-conscious projects, git blame can be employed to trace when and by whom security-related changes were made, aiding in security audits and investigations.

##### Hacks
When you are interested in finding the origin for lines 40-60 for file `foobar`:

```bash
git blame -L 40,+21 foo
```

A particularly useful way is to see if an added file has lines created by copy-and-paste from existing files. Sometimes this indicates that the developer was being sloppy and did not refactor the code properly. You can first find the commit that introduced the file with:

```bash
git log --diff-filter=A --pretty=short -- foo
```
and then annotate the change between the commit and its parents, using commit^! notation:

```bash
git blame -C -C -f $commit^! -- foo
```
{{< admonition info `git annotate`>}}
The only difference between this command and `git blame` is that they use slightly different output formats.
{{< /admonition >}}


#### <b>3. `git bisect`</b>
Use binary search to find the commit that introduced a bug in your code.

This command uses a binary search algorithm to find which commit in your project’s history introduced a bug. You use it by first telling it a "bad" commit that is known to contain the bug, and a "good" commit that is known to be before the bug was introduced. Then `git bisect` picks a commit between those two endpoints and asks you whether the selected commit is "good" or "bad". It continues narrowing down the range until it finds the exact commit that introduced the change.

##### Automating `git bisect`
Suppose we want to find the commit which first intruduced a security flaw in our codebase.
The bug happens in `feat/master` (Bad) and version `v3.4` (Good)

```bash
git bisect feat/master v3.4
```

To check for the bug we (somehow) know that making a `POST` request to our backend with `foobar=admin` allows any user to login, so returning <b>200</b> is a successful login.

```bash
# login.sh
#!/bin/bash

# URL of your backend where you want to make the POST request
url="localhost:8080/login"

data="foobar=admin"

response=$(curl -s -o /dev/null -w "%{http_code}" -d "$data" -X POST "$url")

# Check the HTTP response code
if [ "$response" -eq 200 ]; then
    # Response code is 200, indicating the bug is present (return false to the shell)
    exit 1
else
    # Response code is not 200, indicating the bug is not present (return true to the shell)
    exit 0
fi
```

The automation is then done by just passing the script to `git bisect`:

    git bisect run login.sh

{{< admonition info "Read more about automating bisect" false>}}
[Fully automated bisecting with "git bisect run"](https://lwn.net/Articles/317154/)
{{< /admonition >}}


#### <b>4. `git archive`</b>

Creates an archive of the specified format containing the tree structure for the named tree, and writes it out to the standard output. If `<prefix>` is specified it is prepended to the filenames in the archive.

`git archive` can be useful if you want to create a compressed tarball for `v1.4.0` release and upload somewhere or do something with it.
```bash
git archive --format=tar --prefix=git-1.4.0/ v1.4.0 | gzip >build-1.4.0.tar.gz

# upload somewhere
echo "Uploading to server ..."
```
### Useful git scenarios

#### Creating an orphan branch
An orphan branch is a branch that starts with no commit history but exists alongside your main development branch. It can be useful in specific situations:
1. Separate Documentation or Website: like this blog website, it has 2 branches one for writing markdown files, and the other orphan branch for the generated website.
2. Starting a New Project: if for example you're creating a full stack application and you want to make the server side and front side in one big repo (which I don't recommend) you can have 2 branches, main one and the orphan frontend (coz you know backend comes first).

```bash
$ git checkout --orphan gh-pages

# preview files to be deleted
$ git rm -rf --dry-run .

# actually delete the files
$ git rm -rf .
```

