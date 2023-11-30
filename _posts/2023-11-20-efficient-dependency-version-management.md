---
layout: post
title: Efficient dependency version management
date: 2023-11-20 01:45:42 +0100
categories: ["Software Development", "DevOps"]
---

When working with dependencies one commonly asked question is how to specify the dependencies in the package files (`pyproject.toml`, `Gemfile`, `package.json`, etc.), and why one would need lock files (`poetry.lock`, `Gemfile.lock`, `package-lock.json`). In this article, we will explore how dependency management can be easy and painless. Let's dive in!

![header](/images/header1.jpg)

## Version constraints

First, let's take a look at the different options available to define version constraints:

* **Exact (==) requirements** specify the exact version to be installed, with no possibility of updating.
* **Tilde (~) requirements** define the minimal version to be used with some ability to update. When the major+minor+patch or major+minor versions are specified, then the patch version can be changed; when the major version is specified then the minor version can be changed.
* **Caret (^) requirements** define a [SemVer](https://semver.org/) compatible update. The update is possible when the leftmost non-zero digit is not modified.
* **Wildcard (\*) requirements** allow the usage of the latest dependencies. Here the update is allowed for the versions where the wildcard is specified.

| Specified version constraint | Allowed new versions |
| ---------------------------- | -------------------- |
| ==1.2.3 | NEW_VERSION == 1.2.3
| ^1.2.3 | 1.2.3 <= NEW_VERSION < 2.0.0 |
| ^0.2.3 | 0.2.3 <= NEW_VERSION < 0.3.0 |
| ^0.0 | 0.0.0 <= NEW_VERSION < 0.1.0 |
| ^0 | 0.0.0 <= NEW_VERSION < 1.0.0 |
| ~1.2.3 | 1.2.3 <= NEW_VERSION < 1.3.0 |
| ~1.2 | 1.2.0 <= NEW_VERSION < 1.3.0 |
| ~1 | 1.0.0 <= NEW_VERSION < 2.0.0 |
| 1.2.* | 1.2.0 <= NEW_VERSION < 1.3.0 |
| 1.* | 1.0.0 <= NEW_VERSION < 2.0.0 |
| * | 0.0.0 <= NEW_VERSION |

In theory, the tilde requirement usage would be enough to receive the necessary bug fixes. However, in practice, that is not sufficient, because security fixes are often not backported to previous minor releases, and new minor releases could also contain bug fixes.

This suggests using caret requirements by default. Meanwhile, we can decide to enable a more aggressive dependency update policy by using the wildcard requirements, which let us update to major versions as well. In rare cases, like for alpha versions or with libraries, which do not follow the semantic versioning system, defining exact dependency versions is also considerable.

## About version locking/pinning

According to my experience, many developers choose to use exact versions (this is called version locking/pinning) for safety reasons. Although I think that locking does more harm than good. The following table summarizes the consequences of locking dependencies to specific versions.

| Advantages | Disadvantages |
| ---------- | ------------- |
| **Certainty**: you can always know by looking into the package file what versions are used. | **Locking makes the version compatibility narrow**, which causes frequent dependency conflicts or even unresolvable dependencies. Furthermore, this narrow version compatibility leads to duplicated installed libraries. |
| &nbsp; | **Giving a false sense of security**: you think you are protected, but in reality, only the direct dependencies are locked. A better approach is to use lock files, that will lock down dependencies and implicit dependencies as well. |
| &nbsp; | **Upgrades are more difficult** because the package file has to be adapted every time. When you manually update the libraries this alone often hinders the recurring dependency updates. On the other side when you choose to automate the process, this triggers a package update for each new version, inducing an increased _upgrade noise_. |

As you can see, the benefits of using unlocked versions outweigh the downsides. Using proper version constraints and lock files can bring the desired stability into the application.

## Automated dependency updates to the rescue

There is however an option that further optimizes the process: we can use a bot in our CI/CD tool to update the dependencies. The two most commonly used solutions are the GitHub native [Dependabot](https://docs.github.com/en/code-security/dependabot) and the little bit more comprehensive [Renovatebot](https://docs.renovatebot.com/). Both of them are decent choices, I recommend experimenting with them to figure out which one fits better your needs.

### Package update strategies in Apps vs Libraries

One common property of the dependency updater tools is that they can only work when you specify the version constraints with ranges (tilde/caret requirements). However in the long run as new package versions arrive we will need to decide how these ranges should be updated. This depends on the type of our software:

* **Apps** we would like to keep as up-to-date as possible to avoid possible future [dependency hells](https://en.wikipedia.org/wiki/Dependency_hell). This means the bot will **increase the lower bound** of the ranges to match the current version.
* **Libraries** are different because there we have multiple clients possibly who have different version requirements, hence we need to support ideally the widest possible version ranges. Typically, this calls for **increasing only the upper bound** of the ranges.

### Solutions for the increased update noise

With automating the updates we need to be careful as these automations can quickly overwhelm us and generate a large amount of Pull Requests for us.

Features to reduce the upgrade noise:
* **Limit the number of active PRs**: Start small, gradually increasing the number of PRs created by the updater tool.
* **Scheduled updates**: One often overlooked aspect of automated PR creation is that someone needs to review them and deploy them into production later. Finding a good schedule to bring the new library versions live is also an important step in the process.
* **Dependency grouping**: The most effective way to decrease the number of recurring PRs is to merge updates of similar packages into one.
* **PR automerging**: Low-impact updates can be automatically set to be merged to reduce the manual work further.
* **Branch automerging (without PRs)**: An ultimate step in diminishing noise is to directly merge the changes into the target branch. This will spare us the notifications about the created and merged PRs.

## Summary

Efficient dependency management is crucial for ensuring the proper functioning of software applications. By using up-to-date version constraints, and by leveraging automated dependency updates, software developers can maximize stability while minimizing software issues.



