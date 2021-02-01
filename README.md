# Guides for projects related to OSBuild

Notes on the format:
 * The guides use simple markdown files, which are rendered using mdBook: https://github.com/rust-lang/mdBook/.
 * To render the guides locally run in one of the directories:
```
$ mdbook serve
```

Github Action:
 * The "build-guides" workflow builds the book and stores the resulting html files in `gh-pages` branch in docs folder.
 * Currently works only for the main branch.



*Tip: If you prefer another markup language, use pandoc for translation. For example:*

```
pandoc -f markdown -t asciidoc
```

