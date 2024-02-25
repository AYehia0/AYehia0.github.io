# Setup multiple github emails on a single machine


How to handle multiple github emails linked to the same github account, for example if you have a personal email used to manage your personal projects and you have a professional email for work or something else, it's ideal to use each account on the correct place, but sometimes it can be tedious!
<!--more-->

## Directory Separatation

While you could configure the email and username for each repo:
```bash
git config user.name "personal"
git config user.email "personal@email.com"
```
a directory separatation is a better option in this case, what I mean here is that you need to create `/work` and `/personal` or just `/work` and consider any other directory as personal.

Change the global configs in : `~/.gitconfig` to use another `.gitconfig` in the work directory:
```bash
; the global config
[user]
	name = personal
	email = personal@email.com
	editor = 1

; the work dir
[includeIf "gitdir:~/work/"]
    path = ~/work/.gitconfig

[credential]
	helper = cache --timeout=3600

[core]
	editor = nvim
	autocrlf = input
```

Add the specific configs in the `~/work/.gitconfig`: 
```bash
[user]
	name = Work
	email = work@email.com
	editor = 1
    signingkey = 57A1D460DSFMK210
[commit]
	gpgsign = true
```

## Bonus : Signing the commits

Follow the documentations from github to sign your commits and get the verified badge: [here](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)

{{< admonition tip "How to avoid writing the passphrase each time you commit a message ?" false >}}
If you used a passphrase when creating a gpg key, make sure to cache it to avoid writing it each time you commit a message!

Edit the gnupg config : `~/.gnupg/gpg-agent.conf`
```bash
# cache for 2 days
default-cache-ttl 172800
max-cache-ttl 172800
```

And reload : `gpgconf --reload gpg-agent`
{{< /admonition >}}

