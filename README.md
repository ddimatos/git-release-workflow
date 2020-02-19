
# Branching Strategy

This document describes the community project __Red Hat Ansible Certified Content for IBM Z__ branching and release strategy.  

It provides a brief overview of the release model that we use:  
[release deployment](develop-release-deploy.md) 

This workflow tries to make things as simple as possible while still being flexible enough to work for all contributors, IBM and the community.

## What is the purpose of this document?

As our community project continues to grow, consistency across all teams, partners and contributors is what will enable
us to add features. A ![synchronous workflow](develop-release-deploy.md) ensures we spend our time expanding the collection 
not managing code delivery.

### What if I think it should be done this way?

This document is not final; processes can change, ideas ca inspire change and evolution of a product can change any part
of this document. 

* Pull request are reviewed and encouraged, we never stop evolving. 
* The scope of this document can't possibly cover every permutation on how to do something in Git
* There exists more than one way to do the tasks we describe here and recognize our process is not the only way to do something

### Release Deployment

This is the strategy for __Red Hat Ansible Certified Content for IBM Z__ where features are bundled into a release and 
then released with a [semantic version](https://semver.org/). A feature generally corresponds to a user story and that 
usually maps to an Ansible Role, module, plugin or playbook. When we have completed serveral features usually at the end 
of a 12 week cycle, those features will be pushed to our release master branch and versioned. 

In the release, it may not always included features, we could have improvments to the CI/CD, sharable code, updates, hot fixes and documentation changes. 

[Follow this link for more information on release deployment](./develop-release-deploy.md)



### Git Pull Command

Git provides a single command to update your local branch with changes from a remote.
`git pull` is this command. Most of the time it does exactly what you want without
any problems, but you should know that `git pull` is really `git fetch` followed
by `git merge`. So when you pull from a remote, you're actually updating the remote
tracking branch (eg. `origin/mybranch`) and then merging that into your local
branch `mybranch`.

It's good to know that this happens under the hood. Some people prefer to do the
`git fetch` and `git merge` operations separately. Most of the time `git pull` will
do what you want and is an acceptable way to update your local branch with changes
from remote.

### Protected Branches

Branches `master` and `dev` are protected branches. These protected branches
should never be directly committed to. They should only be updated through PR merges.

Protected branches:
- Can't be force pushed
- Can't be deleted
- Can't have changes merged into them until required status checks pass
