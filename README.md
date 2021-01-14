🚧 Work in progress

# User/Administrator/Contributor Guides for OSBuild and osbuild-composer

Intended audience:
 1. User guide - if you were to use osbuild-composer for the first time, this is what you should read.
 2. Administrator guide - TBD.
 3. Contributor - highlevel overview of osbuild-composer and osbuild, not replicating information that can be found in HACKING.md in osbuild and composer repositories.

Book format:
 * The book uses simple markdown files, which are rendered using mdBook: https://github.com/rust-lang/mdBook/.
 * To render the book locally run:
```
$ mdbook serve
```

Github Action:
 * The "build-guides" workflow builds the book and stores the resulting html files in `gh-pages` branch in docs folder.
 * Currently works only for the main branch.
