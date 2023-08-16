# Welcome to the void
Here I share my thoughts about the universe.

## Development over codespace
1- Fetch all the remote submodules : `git submodule update --init --recursive`

2- Run the development server : `hugo server --disableFastRender --buildDrafts`

3- Expose/Forward the default port to your localhost: `gh codespace ports forward 1313:1313`

4- Create a new post : `hugo new posts/content_name/index.md`

## TODO

- [X] Static TOC
- [X] Enable Search
    - [ ] Use advanced search algolia
- [X] Configure Post metadata
    - [X] Add data about table of content, author.
