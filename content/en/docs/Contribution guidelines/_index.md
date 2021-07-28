---
title: "Contribution guidelines"
linkTitle: "Contribution guidelines"
weight: 5
---

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
```
git clone --recurse-submodules --depth 1 https://github.com/rookie-ninja/rk-docs.git
```

### 3.Install dependencies
rk-docs is using [hugo](https://gohugo.io/) to generate web pages, and [docsy](https://github.com/google/docsy) used as theme.

As a result, we need to install bellow tools for local testing.

#### Install hugo
Official site: [https://gohugo.io/getting-started/installing/](https://gohugo.io/getting-started/installing/)

#### Install Node.js
> docsy theme is using Node.js, so we need to install it for local testing.

Official site: [https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/)

#### Install dependencies
```shell script
sudo npm install -D --save autoprefixer
sudo npm install -D --save postcss-cli
```

### 4.Running hugo
```shell script
$ hugo serve
```

### 5.Validating
[http://localhost:1313](http://localhost:1313)

## Make changes
Please create a new branch:

```
git checkout master
git fetch upstream
git rebase upstream/master
git checkout -b cool_new_feature
```

## Commit code
```
git push origin cool_new_feature
```

At this point, you're waiting on us to review your changes. We **try** to respond to issues and pull requests within a few business days, and we may suggest some improvements or alternatives. Once your changes are approved, one of the project maintainers will merge them.

We're much more likely to approve your changes if you:

- Add tests for new functionality.
- Write a [good commit message][commit-message].

## Useful resources
- [Docsy user guide](https://www.docsy.dev/docs/): All about Docsy, including how it manages navigation, look and feel, and multi-language support.
- [Hugo documentation](https://gohugo.io/documentation/): Comprehensive reference for Hugo.


