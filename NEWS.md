# 0.2.0

## Breaking changes

* A different operator, `..()` (instead of `!!`), is now expanded in meta-mode. In normal execution, this operator is not expanded, and is, instead stripped (i.e., `.,(data())` becomes `data()`). See [this wiki page](https://github.com/rstudio/shinymeta/wiki/Syntax-changes-for-shinymeta-0.2.0) for more information. ([#59](https://github.com/rstudio/shinymeta/pull/59))

## New features

* New `metaAction` function, intended for executing code for its side effects while also capturing the source for code generation. This is useful for app setup code, such as `library()` calls, `source`-ing of supplemental .R files, loading static data sets, etc. ([#71](https://github.com/rstudio/shinymeta/pull/71))

# 0.1.0 (unreleased)

* Initial version, as presented at useR 2019.
