---
layout: post
title: How to build Python packages reproducibly with Poetry
date: 2024-02-19 23:37:42 +0100
categories: ["Software Development", "DevOps", "Python", "Poetry"]
---

For handling dependencies and creating Python packages, [Poetry](https://python-poetry.org/) is a great choice. Poetry's build command can generate _source_ and _wheel_ distributions. The [wheel](https://packaging.python.org/en/latest/glossary/#term-Wheel) is a pre-built distribution format containing files and metadata, which only need to be moved to the target system to be installed. On the other hand, source (or _sdist_) distribution still requires a build step before it is usable. But can these formats be directly utilized in production?

![header](/images/02_header.png)

## Poetry's design issue

It turns out there is a problem with Poetry’s package building functionality; it does not take into account the lock file while executing the build command. This design flaw leads to multiple issues with the built wheel:

* **No exact dependency versions** are specified in the distribution for explicit dependencies if no [exact requirements](https://thesnapdragon.github.io/efficient-dependency-version-management/#version-constraints) are used.
* **Implicit dependencies are missing** from the distribution's dependency specification.

![Unsafe dependencies](/images/02_unsafe_dependencies.png)

Ultimately, these issues make the built wheel distribution unreliable and practically unusable in production as explicit and implicit dependencies are installed non-deterministically.

## Locked versions in distributions

The Poetry team is [aware of the issue](https://github.com/python-poetry/poetry/issues/2778), but making this feature available in a maintainable way is not easy. Until this long-awaited feature is ready, I created a plugin to solve the problem. The plugin extends the building process by reading up the lock file and putting the locked versions into the distribution's metadata:

![Correct dependencies](/images/02_correct_dependencies.png)

### Installation & usage

1. The easiest way to install the lockedbuild plugin is via Poetry:
```bash
poetry self add poetry-plugin-lockedbuild
```

2. Now, the plugin provides a new command to build a wheel file with locked packages:
```bash
poetry lockedbuild
```

For further details, check out the project:

[![thesnapdragon/poetry-plugin-lockedbuild - GitHub](https://gh-card.dev/repos/thesnapdragon/poetry-plugin-lockedbuild.svg)](https://github.com/thesnapdragon/poetry-plugin-lockedbuild)
