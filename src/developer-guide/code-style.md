# Code style guidelines

This depends a little bit on the project and the language. Most of our projects have linters
available, so do make use of those.

If unsure on how to format a specific statement, try to look for examples in the code.

## General

- No trailing whitespace

- Avoid really long lines where possible (>120 characters)

- Single newline at the end of each file

### Golang

This is easy, simply use [Gofmt](https://blog.golang.org/gofmt).

### Python

Python code should follow [the PEP 8 style guide](https://www.python.org/dev/peps/pep-0008/).

### Shell

[ShellCheck](https://www.shellcheck.net/) is used to lint shell code.

### Javascript

Projects like [Cockpit Composer](https://github.com/osbuild/cockpit-composer/) use eslint to enforce
style.
