This is a gitbook repo. When push to github, gitbook (already linked to this repo) will trigger a new build.

Published gitbook url: https://book.enginew.cn/

# repo structure
- `SUMMARY.md` is the table of content of the book
- `README.md` in root folder is the Preface of the book
- `GLOSSARY` is not used for now
- `resources/` is where all images live, and is referenced in article like `![](/resources/xxx.png)`

# SUMMARY.md schema

```md
# SUMMARY -- this is not showed

* [page name](file relative path) -- preface

## TopicName -- this is not clickable and displayed as bold
* [section name] -- can with/without page path
    * [page name](file relative path) -- if without url, it will be placeholder

```

# Math Formular Format
It's rendered using KaTex.

- inline formular should be wrapped with `$$`, e.g., `$$(R_i)$$`.
- multi-line formular also use `$$`, but in different lines.



# Misc Format
- Emoji is supported, e.g. `:house:`
- Annotation is supported, refer to https://gitbook.com/docs/creating-content/formatting/inline#create-an-annotation
- Button is supported, e.g. `<a href="https://app.gitbook.com" class="button primary">MyButton</a>`
- Icons are supported, e.g. `<i class="fa-github">:github:</i>`. refer to https://fontawesome.com/