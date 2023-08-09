# Guides for projects related to Image Builder

The rendered guide can be [viewed here](https://www.osbuild.org/guides/introduction.html)

Notes on the format:
 * The guides use simple markdown files, which are rendered using mdBook: https://github.com/rust-lang/mdBook/.
 * To render the guides locally run in one of the directories:
```
$ mdbook serve
```

You can also use podman to build and serve up the book during editing:

```
podman pull peaceiris/mdbook
podman run -it --rm -p 8080:3000 -v `pwd`:/book --security-opt label=disable peaceiris/mdbook serve -n 0.0.0.0 -d /tmp/book /book
```


Github Action:
 * The "build-guides" workflow builds the book and stores the resulting html files in `gh-pages` branch in docs folder.
 * Currently works only for the main branch.



*Tip: If you prefer another markup language, use pandoc for translation. For example:*

```
pandoc -f markdown -t asciidoc
```

