| [Home](README.md)|
|-----|

# Index
- [Git Workflow Strategy](gitflow.md)
- [Creating a Project Page](gitpage.md)
- [Git Commands](gitcmds.md)
- [Reference](reference.md)

# Branching Strategy

This document describes a Git workflow used with a release strategy.  It 
provides a brief overview of the release model that I use
[release deployment](develop-release-deploy.md) 

This workflow tries to make things as simple as possible while still being
flexible enough to work for all contributors.

## What is the purpose of this document?

As a community project continues to grow, consistency across all teams, 
partners and contributors is what will enable contributors to add features.
A ![synchronous workflow](develop-release-deploy.md) ensures we spend our time
developing and not managing code delivery.

### What if I think it should be done another way?

This document is not final; processes can change, ideas can inspire change and
evolution of a product can change any part of this document. 

* Pull request are reviewed and encouraged, we never stop evolving. 
* The scope of this document can't possibly cover every permutation on how to
do something in Git
* There exists more than one way to do the tasks I describe here and recognize
our process is not the only way to do something, nor perfect.

### Release Deployment

This is the strategy, features are bundled into a release and then released
with a [semantic version](https://semver.org/). A feature generally corresponds
to a user story when several features have been accepted into the `dev` branch, 
those features will be pushed to our release master branch and versioned. 

A release may not always include only features, it could include improvements
to the CI/CD, sharable code, updates, hot fixes and documentation changes. 

[Follow this link for more information on release deployment](./develop-release-deploy.md)


