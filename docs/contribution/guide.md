## Overview
We'd love your help make rk-docs the very best documentation to helper users to understand RK products.

If you hope to add or make changes on rk-docs, please open an issue first with your proposal in order to discuss the changes.

In your issue, pull request, and any other communications, please remember to treat your fellow contributors with respect! We take our [code of conduct](/docs/contribution/code-of-conduct/) seriously.

## Setup
We did not use automated CD procedure in rk-docs projects like netlify.

As a result, we need to follow traditional way to make changes on github.

### 1.Fork
[fork](https://github.com/rookie-ninja/rk-docs/fork)

### 2.Clone
```bash
git clone https://github.com/rookie-ninja/rk-docs.git
```

### 3.Install dependencies
rk-docs is using [mkdocs-material](https://github.com/squidfunk/mkdocs-material/) to generate web pages.

As a result, we need to install bellow tools for local testing.

```bash
pip install mkdocs-material
```

### 4.Running hugo
```bash
mkdocs serve
```

### 5.Validating
[http://localhost:8000](http://localhost:8000)

## Make changes
Please create a new branch:

```bash
git checkout master
git fetch upstream
git rebase upstream/master
git checkout -b cool_new_feature
```

## Commit code
```bash
git push origin cool_new_feature
```

At this point, you're waiting on us to review your changes. We **try** to respond to issues and pull requests within a few business days, and we may suggest some improvements or alternatives. Once your changes are approved, one of the project maintainers will merge them.

We're much more likely to approve your changes if you:

- Add tests for new functionality.
- Write a [good commit message][commit-message].

## Useful resources
- [mkdocs-material](https://github.com/squidfunk/mkdocs-material): All about mkdocs, including how it manages navigation, look and feel, and multi-language support.

