| [Home](README.md) ▸ **Release Deployment** |
|-----|

# Git Workflow Strategy

  This Git work flow is tailored to projects who's features get bundled into a release and versioned.  

  This work flow is a variation of the [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
  workflow. Readers will be provided various use cases and associated personas that include the commands neccessary to 
  exercise the use case.   
  
## Overview
- [Branching](#branching)
- [Use Cases](#use-cases)
  - [Develop a new feature](#develop-a-new-feature)
  - [Develop multiple features in parallel](#develop-multiple-features-in-parallel)
  - [Create and deploy a release](#create-and-deploy-a-release)
  - [Bugfix the release branch](#bugfix-the-release-branch)
  - [Backout a feature from the release branch](#backout-a-feature-from-the-release-branch)
  - [Supporting old releases](Supporting-old-releases)
  - [Bugfix old releases](#Bugfix-old-releases)
  - [Production hotfix](#production-hotfix)
  - [Production hotfix release](#Production-hotfix-release)
- [Squashing commits](#squashing-commits)
- [Workflow anti-patterns](#workflow-anti-patterns)
- [Branch Naming Conventions](#branch-naming-conventions)
- [Prune Branches](#prune-branches)


## Branching

   <!-- Branching Flows -->
   <img width="521" alt="image" src="https://user-images.githubusercontent.com/25803172/74110947-871ef100-4b45-11ea-8e2a-daf6340ab819.png">
   
   | Ledger          | Branch          | Base        | Description    |
   |-----------------|-----------------|-------------|----------------|
   | <img width="13" alt="image" src="https://user-images.githubusercontent.com/25803172/74109916-a9604100-4b3c-11ea-962b-cd65ee1c3e5f.png">       | `master`        | ----------  | This is __stable__ code that is [semantic](http://semver.org/) versioned that requires a pull request to merge into `master`. |
   | <img width="13" alt="image" src="https://user-images.githubusercontent.com/25803172/74109939-d1e83b00-4b3c-11ea-9932-cc4b2edc9c58.png">          | `dev`           | `master`    | This is the development code that keeps developers in synch that has undegone a review and pull request that can be __unstable__. |
   | <img width="13" alt="image" src="https://user-images.githubusercontent.com/25803172/74109943-d876b280-4b3c-11ea-8150-fe9dc99083ae.png">         | feature         | `dev`       | This is a __temporary__ branch with feature code that is actively being developed thus __unstable__. |
   | <img width="13" alt="image" src="https://user-images.githubusercontent.com/25803172/74109946-dca2d000-4b3c-11ea-9fcf-a49411441c9e.png">| `release-vX.Y.Z`| `dev`       | This is a __temporary__ release branch that following the [semantic version](http://semver.org/) that stabelized the release code, allowing for bugfix's to be made without the risk of feature code slipping into the release  |
   | <img width="13" alt="image" src="https://user-images.githubusercontent.com/25803172/74109950-e0ceed80-4b3c-11ea-97fc-6d2099c9b27c.png">          | bugfix          | `release-vX.Y.Z` |  This is a __temporary__  branch with fixes for a __release branch__. The bugfix branch will be merged into the release branch and cherry-picked into the `dev` branch.|
   | <img width="13" alt="image" src="https://user-images.githubusercontent.com/25803172/74109952-e4fb0b00-4b3c-11ea-9960-bc9c4883a188.png">          | hotfix          | `master`    | This is a __temporary__  branch with a production code fix that should be merged into `master` and cherry-picked into `dev` and release branches.|

## Use Cases

### Develop a new feature
<!-- Development persona icon-->
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer">
<img width="31" alt="image" src="https://user-images.githubusercontent.com/25803172/74109197-36ec6280-4b36-11ea-9850-a48e2b7118c9.png">
</a>
 	
<!-- Git Develop a new feature workflow image -->
<img width="551" alt="image" src="https://user-images.githubusercontent.com/25803172/74111973-aae63500-4b4d-11ea-906a-f0cdce8e7857.png">
   
A developer wants to start a new feature that will ship in a future release. They should branch off of the `dev` branch 
because there could be other features also in the `dev` branch shipping in the next release and you should be in synch
with the rest of the development team.

1. Create a feature branch "feature/21/ansible-zos-raw-module" based off of `dev` and set the default remote branch to the 
current local branch. When setting the default remote branch, then a `git pull` will pull in commits from the remote 
feature branch.

 ```
 $ git checkout dev
 $ git checkout -b feature/21/ansible-zos-raw-module
 $ git push -u origin feature/21/ansible-zos-raw-module
 ```

2. Develop code for the new feature, commit and push your changes often. This allows others to see your changes and make 
suggestions as well as securing your code should your local storage fail.

```
$ ... make changes
$ git add .
$ git commit -m "Documented the zos_raw_module feature options"
$ ... make more changes
$ git add .
$ git commit -m "Fixed hard code counter with a user defined environment variable"
$ git push
```

3. Navigate to the project [ibm_zos_core](https://github.com/ansible-collections/ibm_zos_core) and open a pull 
request with the following branch settings:
* Base: `dev`
* Compare: `feature/21/ansible-zos-raw-module`

4. When the pull request has been reviewed on github:
   * merge pull request
   * comment and close the pull request
   * delete the `feature/21/ansible-zos-raw-module` branch
   
### Develop multiple features in parallel

<!--Development personal icon--> 
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer">
<img width="31" alt="image" src="https://user-images.githubusercontent.com/25803172/74109197-36ec6280-4b36-11ea-9850-a48e2b7118c9.png">
</a>

There is nothing unique about developing multiple features in parallel. Each developer simply needs to follow the same 
process outlined in [Develop a new feature](#develop-a-new-feature) for each feature they are working on.

### Create and deploy a release

<!--Release Manager Icon-->
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#rachel-release-manager">
<img width="46" alt="image" src="https://user-images.githubusercontent.com/25803172/74109181-12908600-4b36-11ea-9b05-be812d199e48.png">
</a>

<img width="557" alt="image" src="https://user-images.githubusercontent.com/25803172/74111865-9e151180-4b4c-11ea-8057-8696015625b1.png">

1. Merge `master` into `dev` to ensure the new release will contain the latest production code. This reduces the 
chance of a merge conflict during the release.

```
$ git checkout dev
$ git merge master
```

2. Create a new `release-v2.0.0` release branch off of `dev`.

```
$ git checkout -b release-v2.0.0
$ git push -u origin release-v2.0.0
```

3. After creating release-v2.0.0, developers should only be bugfixing this branch, no new features should be allowed after this point.
Remember, the release branch is only a temprorary branch and its purpose is to take a snapshot of the `dev` branch such that 
fetures not releasing can continue to be push into `dev`. 

4. When the release bugfix window has closed. Navigate to the project [ibm_zos_core](https://github.com/ansible-collections/ibm_zos_core) 
and open a Pull Request with the following branch settings:
* Base: `master`
* Compare: `release-v2.0.0`

5. When the pull request has been reviewed on github:
   * merge pull request
   * comment and close the pull request
   * DO NOT delete `release-v2.0.0` yet...

   __Important Note__:  Always use pull requests when merging into `master`.

6. Now you are ready to create the actual release since you have merged the release branch `release-vX.Y.Z` into `master`. 
Navigate to the project page on Github and draft a __new release__ with the following settings:
* Tag version: `v2.0.0`
* Target: `master`
* Release title: `release-v2.0.0`
* Description: List the features and capabilities this version will include. 
* __Click__ `Publish release`  

7. Merge `release-v2.0.0` into the `dev` branch because by now, its possible the `dev` branch that may have 
diverged and is lacking bugfix's added to `release-v2.0.0`.

```
$ git checkout dev
$ git merge release-v2.0.0
$ git push
```

8. Complete any additional tasks in the github template as part of the pull requested you opened.
9. Close the pull request and delete the release branch `release-v2.0.0`.

### Bugfix the release branch

<!--Development personal icon--> 
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer">
<img width="31" alt="image" src="https://user-images.githubusercontent.com/25803172/74109197-36ec6280-4b36-11ea-9850-a48e2b7118c9.png">
</a>

1. Release branches should not have any new features added, only bugfixes for currently releasing features.

2. If bugs are found in the release branch, stabilize the release by creating bugfix branches off of the `release-vX.Y.Z` branch.
Remember to prefix your bugfix branch with __bugfix__ to indicate this is a bugfix and not any other type of fix. 

```
$ git checkout release-v2.0.0
$ git checkout -b bugfix/29/zos-raw-module-parse-failure
$ git push -u origin bugfix/29/zos-raw-module-parse-failure
...fix bugs...commit...make more bugs....
$ git commit -m "Fix parse error by correcting EOF"
$ git push
```

3. Navigate to the project [ibm_zos_core](https://github.com/ansible-collections/ibm_zos_core) and open a pull 
request with the following branch settings:
* Base: `release-v2.0.0`
* Compare: `bugfix/29/zos-raw-module-parse-failure`

4. When the pull request has been reviewed:
* merge the pull request
* comment and close the pull request
* delete the `bugfix/29/zos-raw-module-parse-failure` branch


### Backout a feature from the release branch
<!--Release manager persona icon-->
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#rachel-release-manager">
<img width="46" alt="image" src="https://user-images.githubusercontent.com/25803172/74109181-12908600-4b36-11ea-9b05-be812d199e48.png">
</a>

//TODO - will document this at a later date either as a pull request reversal or new branch + pull request reversal. 

### Supporting old releases
<!--Release manager persona icon-->
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#rachel-release-manager">
<img width="46" alt="image" src="https://user-images.githubusercontent.com/25803172/74109181-12908600-4b36-11ea-9b05-be812d199e48.png">
</a>
    
To support a paticular release, we will adopt the support branch model, support branches are created as neeeded for customers who are on older release who can not adopt the lastest release but still need bug fixes, security fixes and ported features. Old releases often become incompatible with the most recent versions of projects thus we should encourage users to adopt new releases.

Support branches are long-lived branches to maintain legacy code without the need to be merged back into master. Support branches 
__do not get merged__ back into __`master`__ or __`dev`__, this __would cause major merge issues__. Instead, commits are cherry-picked 
from the support branch back into `dev` through a pull request. 
   
Support branches can be thought of as the `master` branch for old release. Support branches for major releases should be 
named as `support-v<major>.x`. Support branches for minor releases should be named as `support-v<major>.<minor>.x`. 
The __x__ remains part of the branch name to signify that any number of fixes will continue to append to the support branch.
   
Here is an example of creating a support branch for v1.0.0 assuming the project is currently on v2.0.0.

1. Create the support branch and release branch:
   * Checkout the tag for the 1.0.0 release   
   ```git checkout v1.0.0```
   
   * Create the long living support branch (don't change the __x__ in `support-v1.x`, it represents subsequent added fixes)   
   ```git checkout -b support-v1.x```
   
   * Create the release temporary branch incrementing the minor verson   
   ```git checkout -b release-v1.0.1```

   Note: For subsequent releases (i.e. v1.0.2) the release branch will be branched off the `HEAD` of `support-v1.x`

2. Now that an incremented release branch `release-v1.0.1` has been created a ![developer](https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer) can now follow the instructions on how to ![Bugfix old releases](#bugfix-old-releases) who will also create a pull request to merge `release-v1.0.1` into `support-v1.x`.
   
3. After reviewing  and approving the pull request to merge `release-v1.0.1` into `support-v1.x`, ensure that the associated fix will be cherry-picked into the `dev` branch through a pull request.  

```
git checkout release-v1.0.1
git checkout -b bugfix/45/cherry-pick-release-v1.0.1-fix
git push -u origin bugfix/45/cherry-pick-release-v1.0.1-fix
git cherry-pick -x commit-sha-one
git push
```
  
4. Navigate to the project and open a pull request with the following branch settings:
* Base: `dev`
* Compare: `bugfix/45/cherry-pick-release-v1.0.1-fix`

5. When the pull request has been reviewed on github:
   * merge pull request
   * comment and close the pull request
   * delete the `bugfix/45/cherry-pick-release-v1.0.1-fix` branch

6. Follow the standard release process outlined in ![Create and deploy a release](#create-and-deploy-a-release) 
treating `support-v1.x` as the `master` branch. If you follow the __Create and deploy a release__ process, branch `release-v1.0.1` will 
get __deleted__ and `support-v1.x` will remain in the repository indefinitely to bugfix but incrementing the minor vession for each fix.

7. Mark `support-v1.x` as a ![protected branch](https://help.github.com/en/github/administering-a-repository/configuring-protected-branches)
in Github so that it does not get accidentally deleted.

To summarize, you are either bug fixing or backporting a feature to an older release that a customer could have in production.
If its a bug, it could be inherent in the `dev` branch, thus propagted to future releases; you will want to make sure you 
cherry-pick the fix accordingly, see [Bugfix old releases](#bugfix-old-releases) also ensure that bugfixes are compatible 
with the current state of `dev`.  
   
__Tip__: Try to maintain as few support branches as possible. These branches are expensive to maintain since need to 
cherry-pick applicable bugfixes into __each support branch__ seperately.

### Bugfix old releases
    
<!--Development personal icon--> 
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer">
<img width="31" alt="image" src="https://user-images.githubusercontent.com/25803172/74109197-36ec6280-4b36-11ea-9850-a48e2b7118c9.png">
</a>
    
Create a bugfix branch based on the appropriate release branch for an older release, there should already be a 
base release branch with pattern `release-v*` created for you, see [Supporting old releases](#supporting-old-releases). 

Bugfix the release branch:
```
$ git checkout release-v1.0.1
$ git checkout -b bugfix/79/zos-dat_set-module-validation-failure
$ git push -u origin bugfix/79/zos-dat_set-module-validation-failure
...fix bugs...commit...fix more bugs....
$ git commit -m "Fix parsing by correcting EOF"
$ git push
```

1. Navigate to the project [ibm_zos_core](https://github.com/ansible-collections/ibm_zos_core) and open a pull request with
the following branch settings:
* Base: `release-v1.0.1`
* Compare: `bugfix/79/zos-dat_set-module-validation-failure`

2. When the pull request has been reviewed on github:
   * merge the pull request
   * comment and close the pull request
   * delete the `bugfix/29/zos-raw-module-parse-failure` branch

3. For each bug you fix in a old release, cherry-pick the fix into a bugfix branch based on `dev`. You can not merge
an older release branch into `dev`, the code has diverged to far and be a troublesome merge, therefore cherry-picking
is preferred.

Bugfix the develpment branch:
```
git checkout dev
git checkout -b bugfix/88/zos-dat_set-module-validation-failure
git brancy --set-upstream-to origin/bugfix/88/zos-dat_set-module-validation-failure
git cherry-pick <hash>, git cherry-pick <hash> another, etc
git push
```

4. Navigate to the project [ibm_zos_core](https://github.com/ansible-collections/ibm_zos_core) and open a pull request with
the following branch settings:
* Base: `dev`
* Compare: `bugfix/88/zos-dat_set-module-validation-failure`

5. When the pull request has been reviewed on github:
   * merge the pull request
   * comment and close the pull request
   * delete the `bugfix/88/zos-raw-module-parse-failure` branch
   
### Production hotfix
    
<!--Development persona icon--> 
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer">
<img width="31" alt="image" src="https://user-images.githubusercontent.com/25803172/74109197-36ec6280-4b36-11ea-9850-a48e2b7118c9.png">
</a>

A production hotfix is similar to a release bugfix only that you do your work in a branch based on `master`, this is because
the latest production release is based on the `master` branch. Hotfixes are useful in cases where you want to fix a bug 
in a current release and because `dev` could have new code in it, these two branches likely have diverged. 

1. Create a hotfix branch based off of `master`, fix the issue and include a test case.

```
$ git checkout master
$ git checkout -b hotfix/99/zos_query_job-module-read-failure
$ git push -u origin hotfix/99/zos_query_job-module-read-failure
... fix bug ...commit
... add test ...verify ...commit
$ git add .
$ git commit -m "Fix job read failure"
$ git push
```

2. Navigate to the project and open a pull request with
the following branch settings:
* Base: `master`
* Compare: `hotfix/99/zos_query_job-module-read-failure`

### Production hotfix release
<!--Release Manager persona icon-->
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#rachel-release-manager">
<img width="46" alt="image" src="https://user-images.githubusercontent.com/25803172/74109181-12908600-4b36-11ea-9b05-be812d199e48.png">
</a>
   
After development has performed the [Production hotfix](#Production hotfix) it is time to review the pull request.

1. Review the pull request:
* merge the pull request
* comment and close the pull request
* __DO NOT delete__ `hotfix/99/zos_query_job-module-read-failure` yet...

2. Now that the hotfix code has been merged into the `master` branch, you are ready to create the release. In this example we are assuming the latest production release is v2.0.0 so will increment the minor version. Hotfix releases are actualy releases thus you sould increment at least the minor version of the release.

Navigate to the project page on Github and draft a new release with the following settings:
* Tag version: `v2.0.1`
* Target: `master`
* Release title: `Release v2.0.1`
* Description: List the features and capabilities this version will include.
* __Click__ `Publish release`.

2. Now it's time to merge the `hotfix/99/zos_query_job-module-read-failure` branch into the `dev` branch. This will ensure the hotfix is propagated and correclty merged into the `dev` branch. 

```
$ git checkout dev
$ git merge hotfix/99/zos_query_job-module-read-failure
$ git push
```

3. Now delete the `hotfix/99/zos_query_job-module-read-failure` branch. 

# Workflow anti-patterns
These are considred anti-pattern practices that for various reasons put the release at risk, conflicting merges, cluttered Git logs.

## Don't add more than one feature to a pull request
   A pull request that has feature code, fixes, bug fixes etc is very long to review and very difficult for the code reviewer. 
   
   * This happens often when:
     * someone either is working on a feature branch and sundenly is tasked with new work
     * think they can squeeze in a an unrelated fix to this branch because they think no one will notice or its no big deal
     * don't really understand Git and think they will lose their current work thus they should have checked out a new branch 
       and started the new task and at anytine checked out the prior banch and picked up where they left off. 
     * they are lazy
  * This causes:
    * a cluttered git log
    * delays in delivering features as pull requests will reject this
    * lost time
    
## Don't merge code without a code review
   Ask a team member to review your code for readability, bugs, performance and meeting the products guidelines. Better
   than one reviewer is 2, see Microsoft's ![research](https://www.microsoft.com/en-us/research/publication/convergent-software-peer-review-practices/)
   
   * This happens when:
     * features are late to deliver
     * developer thinks they have written and tested every part of the feature
   * This causes:
     * bugs, technical debt, rewrites, interface changes and so much more
     * missed dates
     
## Don't write code to any protected branch
   If by chance a branch such as `master`  or `dev` is not protected in that only certain individuals can perform a pull
   request into it, don't use this as a reason to do so. Remember, `master` and `dev` are not your sandboxes to write code
   into, this is why there are feature branches.
   
   * This happens when:
     * developers don't understan Git
     * someone is pressureing development for a fix and standing over their shoulder
     * someone believes they are beyond process and not a team player
   * This casues:
     * bugs, technical debt, rewrites, interface changes and so much more
     * missed dates
     * a cluttered git log
     
# Squashing commits

   Its important that commits are ![squashed](squashing-commits.md) before a pull request is made otherwise it will be 
   rejected. Squashing aids in reducing the entries made into a Git log, making it harder for anyone to manaage, service 
   and backport/forward port changes. 
   
## Don't forget to squash
   When working out of your feature branch, you should be committing changes often, when a logical point is reached meaning
   the feature is ready for a code review, squash the commits into one. 
   
   * This happens when:
     * Developers don't understand Git
     * Develoeprs are rushed to deliver
     * Developers don't care that a Git log is cluttered
   * This causes:
     * Delays... pull requests will reject any code not squashed
     * Cluttered git log

# Branch Naming Conventions
Branch naming is left mostly up to the discretion of the person creating the branch with a few exceptions. `master` 
and `dev` are always named exactly that. 

1. Branch names should use dashes to separate words of the name and should avoid anyuppercase letters.

2. Choose names that are descriptive and concise, you don't need long names because most branches are short lived for 
the duration of a feature, fix, etc. 

3. Convention used:
     __topic__/__issue-tracker-number__/__short-description__
   
4. Topics to choose from
   
   | Topic           | Description   
   |-----------------|------------------|
   | feature         | new feature, something substantial such as a user story |
   | content         | document, readme, wiki page change                      |
   | bugfix          | fix for the development branch                          |
   | hotfix          | fix for the production code that is in master           |
   | update          | generally refelects a change such as enhancment ordelete. Not depcited in the diagrams but follows the life of a bugfix branch |

5. Issue tracker number
   This is the number associated to how the work is being tracked, it could be a the JIRA number, git issue number but in
   some tracking softare its not a simple few digits it can be a long hash, in that case some simple text or the last few
   digits of the hash would suffice. 
   
   For example for JIRA-22 we would just use 22, and for .../gitissue/13, we would use just 13. 
   
6. Short description
   This a very short description of the work being committed to the branch. This is meant to be very high level, keep it
   around 5 words.

7. Some examples of branches:
   * feature/21/ansible-zos-raw-module
   * content/55/change-data-set-ansible-doc
   * enhancement/66/add-copy-module-vsam-support
   * bugfix/47/data-set-write-corruption
   * hotfix/99/connection-failure-aws
 
# Prune branches

After some time, you end up with no longer used branches in the repository. Here we will cover how to remove them.

Remeber, a `git pull` also performs a `git fetch` and that in turn pulls down the tracking branches from origin. After
some time , if you run a `git branch` command that will see many origin/* branches in the local repository that no longer 
exist on remote. 

* To see which branches are in local that aren't on remote:  
   `git remote prune origin --dry-run`

* To delete/prume the stale remote tracking branches:  
  `git remote prune origin` or `git fetch --prune`

* Lets look at pruning the local branches that have been merged:  
  `git branch --merged`

* Delete the local branch that has been merged and no longer needed:  
  `git branch -d unwanted-branch`

* To view any local branches which have not been merged:  
  `git branch --no-merged`

* To delete/prune any local branches which have not been merged and are no longer needed (notice the uppercase -D to force):  
  `git branch -D un-merged-branch-to-delete`
  
# Citations
  * Atlassian, Bitbucket. [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
