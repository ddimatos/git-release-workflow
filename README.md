```
               _____          _   _    _       _                        _ _     _       
              |  __ \        | | | |  | |     | |       /\             (_) |   | |      
              | |__) |___  __| | | |__| | __ _| |_     /  \   _ __  ___ _| |__ | | ___  
              |  _  // _ \/ _` | |  __  |/ _` | __|   / /\ \ | '_ \/ __| | '_ \| |/ _ \ 
              | | \ \  __/ (_| | | |  | | (_| | |_   / ____ \| | | \__ \ | |_) | |  __/ 
              |_|  \_\___|\__,_| |_|  |_|\__,_|\__| /_/    \_\_| |_|___/_|_.__/|_|\___| 
              _____          _   _  __ _          _    _____            _             _   
             / ____|        | | (_)/ _(_)        | |  / ____|          | |           | |  
            | |     ___ _ __| |_ _| |_ _  ___  __| | | |     ___  _ __ | |_ ___ _ __ | |_ 
            | |    / _ \ '__| __| |  _| |/ _ \/ _` | | |    / _ \| '_ \| __/ _ \ '_ \| __|
            | |___|  __/ |  | |_| | | | |  __/ (_| | | |___| (_) | | | | ||  __/ | | | |_ 
             \_____\___|_|   \__|_|_| |_|\___|\__,_|  \_____\___/|_| |_|\__\___|_| |_|\__|
                                            __           
                                           / _|           
                                          | |_ ___  _ __  
                                          |  _/ _ \| '__| 
                                          | || (_) | |    
                                          |_| \___/|_|            
                                 _____ ____  __  __      ______
                                |_   _|  _ \|  \/  |    |___  /
                                  | | | |_) | \  / |       / / 
                                  | | |  _ <| |\/| |      / /  
                                 _| |_| |_) | |  | |     / /__ 
                                |_____|____/|_|  |_|    /_____|                     
```

# Branching Strategy

This document describes the community project __Red Hat Ansible Certified Content for IBM Z__ branching and release strategy.  

It provides a brief overview of the release model that we use:
[release deployment](release-deployment.md) 

This workflow tries to make things as simple as possible while still being flexible enough to work for all contributors, IBM and the community.

## What is the purpose of this document?

As our community project continues to grow and expand, consistency across all teams, partners and contributors is what will ensure we can focus on developing source code and not managing the contribution. Aligning ourselves with an agreed delivery process ensures we spend our time expanding the collection not managing code delivery.

Thtis document outlines:
* How we develop code in a GitFlow process
* When and how we create a collection release
* How to handle a collection hot fix
* Anything else related to our branching strategy

### What this document is not

* __Final...__ 
    * This document continues to evolve and improve over time with community feedback. 
    * Pull request are reviewed and encouraged, we never stop evolving. 

* __The only way to work...__
    * The scope of this document can't possibly cover every sceanrio and every variation on how to do something in Git
    * There exists more than one way to do the tasks we describe here, we recognize our recommended way is not the only way to perform a given task, its the way we are comfortable doing so and have outlined that process here. 

### Release Deployment

This is the strategy for __Red Hat Ansible Certified Content for IBM Z__ where features are bundled into a release and then released with a [semantic version](https://semver.org/). A feature generally corresponds to a user story and that usually maps to an Ansible Role, module, plugin or playbook. When we have completed serveral features usually at the end of a 12 week cycle, those features will be pushed to our release master branch and versioned. 

In the release, it may not always included features, we could have improvments to the CI/CD, sharable code, updates, hot fixes and documentation changes. 

[Follow this link for more information on release deployment](./release-deployment.md)

## Branch Naming Conventions
Branch naming is left mostly up to the discretion of the person creating the branch
with a few exceptions. `master` and `dev` are always named exactly that. 

Branch names should use dashes to separate words of the name and should avoid any
uppercase letters.


Other than that, choose names that are descriptive and concise. You don't need a branch
name that is a novel because most branches should be relatively short-lived (hours to
days, not weeks).

name/reason/issue-tracker-number/short-description
* name
  * username associated to git
* reason
  * feature - refers to a user story which usually relates to a module, plugin, role, playbook
  * content - refers to WIKI page, doc, readme(s)
  * enhancement
  * refactor
  * delete
  * update
  * fix
  * hot-fix
  * patch
* issue-tracker-number
  * git issue number
  * jira tracker number
* short-description
  * short description of what is being committed to the branch
  
 Some examples:
 * ddimatos/feature/nazare-21/ansible-zos-raw-module
 * ddimatos/content/nazare-55/change-data-set-ansible-doc
 * ddimatos/enhancement/nazare-66/add-copy-module-VSAM-support
 * ddimatos/fix/nazare-47/data-set-write-corruption-vX.Y.Z
 * ddimatos/fix-x.y.z/nazare-47/data-set-write-corruption-x.y.z

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

## Anti-Patterns

After reading all of the above, none of the [Anti-Patterns](antipatterns.md) should
come as a surprise.
