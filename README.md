🚧 Work in progress

Book format:
 * The book uses simple markdown files, which are rendered using mdBook: https://github.com/rust-lang/mdBook/.
 * To render the book locally run in one of the directories:
```
$ mdbook serve
```

Github Action:
 * The "build-guides" workflow builds the book and stores the resulting html files in `gh-pages` branch in docs folder.
 * Currently works only for the main branch.
