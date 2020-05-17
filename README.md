# latex-action

[![GitHub Actions Status](https://github.com/jeff-tian/latex-action/workflows/Test%20Github%20Action/badge.svg)](https://github.com/jeff-tian/latex-action/actions)

GitHub Action to compile LaTeX documents.

It runs in [a docker image](https://github.com/jeff-tian/latex-docker) with a full [TeXLive](https://www.tug.org/texlive/) environment installed.

If you want to run arbitrary commands in a TeXLive environment, use [texlive-action](https://github.com/xu-cheng/texlive-action) instead.

## Inputs

- `root_file`

    The root LaTeX file to be compiled. This input is required. You can also pass multiple files as a multi-line string to compile multiple documents. For example:
    ```yaml
    - uses: xu-cheng/latex-action@v2
      with:
        root_file: |
          file1.tex
          file2.tex
    ```

- `working_directory`

  The working directory for the LaTeX engine.

- `compiler`

  The LaTeX engine to be invoked. By default, [`latexmk`](https://ctan.org/pkg/latexmk) is used, which automates the process of generating LaTeX documents by issuing the appropriate sequence of commands to be run.

- `args`

  The extra arguments to be passed to the LaTeX engine. By default, it is `-pdf -file-line-error -interaction=nonstopmode`. This tells `latexmk` to use `pdflatex`. Refer to [`latexmk` document](http://texdoc.net/texmf-dist/doc/support/latexmk/latexmk.pdf) for more information.

- `extra_system_packages`

  The extra packages to be installed by [`apk`](https://pkgs.alpinelinux.org/packages) separated by space. For example, `extra_system_packages: "py-pygments"` will install the package `py-pygments` to be used by the `minted` for code highlights.

* `pre_compile`

    Arbitrary bash codes to be executed before compiling LaTeX documents. For example, `pre_compile: "tlmgr update --all"` to update all TeXLive packages.

* `post_compile`

    Arbitrary bash codes to be executed after compiling LaTeX documents. For example, `post_compile: "latexmk -c"` to clean up temporary files.

## Example

```yaml
name: Build LaTeX document
on: [push]
jobs:
  build_latex:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Git repository
        uses: actions/checkout@v2
      - name: Compile LaTeX document
        uses: xu-cheng/latex-action@v2
        with:
          root_file: main.tex
```

## FAQs

### How to use XeLaTeX or LuaLaTeX instead of pdfLaTeX?

By default, this action uses pdfLaTeX. If you want to use XeLaTeX or LuaLaTeX, you can set the `args` to `-xelatex -file-line-error -interaction=nonstopmode` or `-lualatex --file-line-error --interaction=nonstopmode` respectively. Alternatively, you could create a `.latexmkrc` file. Refer to the [`latexmk` document](http://texdoc.net/texmf-dist/doc/support/latexmk/latexmk.pdf) for more information.

### How to enable `--shell-escape`?

To enable `--shell-escape`, you should add it to `args`. For example, set `args` to `-pdf -file-line-error -interaction=nonstopmode -shell-escape` when using pdfLaTeX.

### Where is the PDF file? How to upload it?

The PDF file will be in the same folder as that of the LaTeX source in the CI environment. It is up to you on whether to upload it to some places. Here are some example.

- You can use [`@actions/upload-artifact`](https://github.com/actions/upload-artifact) to upload PDF file to the workflow tab.
- You can use [`@actions/upload-release-asset`](https://github.com/actions/upload-release-asset) to upload PDF file to the Github Release.
- You can use normal shell tools such as `scp`/`git`/`rsync` to upload PDF file anywhere. For example, you can git push to the `gh-pages` branch in your repo, so you can view the document using Github Pages.

### It fails due to `xindy` cannot be found.

This is an upstream issue where `xindy.x86_64-linuxmusl` is currently missing in TeXLive. To work around it, try [this](https://github.com/xu-cheng/latex-action/issues/32#issuecomment-626086551).

### It fails to build the document, how to solve it?

- Try to solve the problem by examining the build log.
- Try to build the document locally.
- You can also try to narrow the problem by creating a minimal working example to reproduce the problem.
- [Open an issue](https://github.com/jeff-tian/latex-action/issues/new) if you need help.

### Run from local machine

```shell
docker build -t jefftian/texlive-full .
# macOS
docker run --name latex-action --rm -v "/var/run/docker.sock":"/var/run/docker.sock" -v "$PWD":/root/workspace jefftian/texlive-full:latest "fduthesis.tex" "/root/workspace/test" "arara" "--verbose" ""
# Windows
docker run --name latex-action --rm -v "/var/run/docker.sock":"/var/run/docker.sock" -v "%cd%":/root/workspace jefftian/texlive-full:latest "fduthesis.tex" "/root/workspace/test" "arara" "--verbose" ""
```

### MWE 

[mwe]: https://tex.meta.stackexchange.com/questions/3300/minimum-working-example-mwe

## License

MIT
