---
title: "Defensive Commits"
date: 2022-10-22T21:55:26-04:00
draft: false
toc: false
images:
tags:
  - git
  - ci
  - continuous delivery
---

Defensive Commits are a strategy for pushing commit to upstream repos that runs part of the ci/cd pipeline locally using git hooks. It fails the hook fails. This allows you to catch integration errors and get feedback quickly. This strategy works best with [trunk-based development](https://trunkbaseddevelopment.com) and ci/cd teams, because it helps avoid breaking the build and reduces dumb mistakes from even being pushed up to main.

At [Coqui Health](https://coquihealth.com), we've implemented this strategy successfully and this is now the default way we work. We use a monorepo strategy coupled with ci/cd and trunk-based development, which means we work close to or in trunk and so having a robust ci/cd pipeline is paramount. Defensive commits allows us to catch bugs early and get feedback that our code will work or break things. It's allowed us to move faster and not break things.

## How to Implement
The easiest way to implement Defensive Commits (TM) is to use git hooks, particularly on push or after a commit, literally. We used [husky](https://typicode.github.io/husky/#/) to help us do this. 

First install and configure husky

```bash
npx husky-init && npm install       # npm
```

Then configure the git hooks, like this for pre-push

```bash
npx husky add .husky/pre-push "npm test"
```

## Conclusion

Using this strategy you can save a lot of time and effort from having to wait for ci/cd pipelines and potentially braking the build over stupid little things. I can't guarantee your build will never break, but the probability it will fail is a lot less.