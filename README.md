[![build status](https://gitlab.com/bu_cnio/bu_cnio.gitlab.io/badges/master/pipeline.svg)](https://gitlab.com/bu_cnio/bu_cnio.gitlab.io/-/commits/master)

---

This project generates the BU_CNIO docs website located at [https://bu_cnio.gitlab.io]()

---

## GitLab CI

This project's static Pages are built by [GitLab CI][ci], following the steps
defined in [`.gitlab-ci.yml`](.gitlab-ci.yml):

```
image: python:3.8-buster

before_script:
  - pip install mkdocs
  ## Add your custom theme if not inside a theme_dir
  ## (https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes)
  # - pip install mkdocs-material

pages:
  script:
  - mkdocs build
  - mv site public
  artifacts:
    paths:
    - public
  only:
  - master
```

## Building locally

To work locally with this project, you'll have to follow the steps below:

1. Fork, clone or download this project
1. Install MkDocs
1. Preview your project: `mkdocs serve`,
   your site can be accessed under `localhost:8000`
1. Add content
1. Generate the website: `mkdocs build` (optional)

Read more at MkDocs [documentation](https://www.mkdocs.org/).

---
