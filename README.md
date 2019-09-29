# README

This project is a build project for JFR introduction Book.

## Site URL

https://koduki.github.io/docs/book-introduction-of-jfr/site/


## Dev build

```bash
gitbook serve
```

## Build

```bash
docker run -it -v `pwd`/docs/:/srv/gitbook fellah/gitbook gitbook build
```

## Publish

```bash
docker run -it --rm -v `pwd`/docs:/work \
    -e GITHUB_ACTOR=koduki \
    -e GIT_ID=koduki \
    -e SRC_DIR=_book \
    -e DEPLOY_DIR=docs/book-introduction-of-jfr/site/ \
    -e PERSONAL_TOKEN=${{ secrets.PERSONAL_TOKEN }} \
    -e COMMIT_MSG="Publish TEST" \
    koduki/gh-actions_github-pages
```