# Git for hackers


# Git for hackers

{{< admonition quote "Once a wise man said:" true >}}
"There are two types of hackers in this world: those who know git, and those who still don't git it" 
{{< /admonition >}}

When I refer to [`hackers`](https://en.wikipedia.org/wiki/Hacker), I mean those who love to go be behind the things to discover how things work and break them (somehow). Originally, hacker simply meant advanced computer technology enthusiast (both hardware and software) and adherent of programming subculture who used their skill to achieve a goal or overcome an obstacle.

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

#### 1. `git stash`
Stash the changes in a dirty working directory away

when you want to record the current state of the working directory and the index, but want to go back to a clean working directory. The command saves your local modifications away and reverts the working directory to match the HEAD commit.

The modifications stashed away by this command can be listed with `git stash list`, inspected with `git stash show`, and restored (potentially on top of a different commit) with `git stash apply`. Calling git stash without any arguments is equivalent to `git stash push`

{{< admonition tip Tip >}}
You can interactively choose which changes to stash when using `git stash -p`
{{< /admonition >}}

#### 2. `git cherry-pick`

Apply the changes introduced by some existing commits to your repo.

`git cherry-pick` is a very useful command to use in case you want to have some commits to be picked by reference and added to the current working HEAD.

<br>
<p align="center">
    <img src="one-simply-cherry-pick.jpg" width="100%">
</p>

