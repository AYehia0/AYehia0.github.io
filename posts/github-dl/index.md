# How to download files and directories from github


## Using the github website
<br>
Probably this is the easiest way to download files from github, by navigating to the file on github and click on the download button.

Unfortenitaly, it's not the fastest, you will have to move the file to the desired destination and doesn't work for raw files like (txt and .ext).

{{< image src="github_download_page.png" position="center" style="border-radius: 8px;" >}}

## Using wget/curl

Now we talk :D, here's a bash function (add to your alias section) I wrote to automate the download process by using the URL which contains `blob` keyword.

Note: you know what to do to make it work with cURL :D

```bash
#!/bin/bash
gd () {
    SEARCH="\/blob\/"
    replace="\/raw\/"
    result=`echo $1 | sed -e "s/$SEARCH/$replace/g"`
    wget $result
}
```

{{< image src="wget.gif" position="center" style="border-radius: 8px;" >}}

## Downloading directories

GitHub doesn't support git-archive (a git feature to download specific folders). However, it supports a variety of Subversion features, one of which we can use. Subversion is a VC (Version Control) just like `git`.

Make sure you have `subversion` installed, for arch : `sudo pacman -S subversion`

```bash
modify_url () {
    SEARCH="\/tree\/\w\+\/"
    replace="\/trunk\/"
    echo $1 | sed -e "s/$SEARCH/$replace/g"
}

dgd () {
    url=`modify_url $1`
    svn export $url
}
```

{{< image src="svn_github.gif" position="center" style="border-radius: 8px;" >}}


