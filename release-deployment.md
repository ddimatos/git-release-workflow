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
 $ git branch --set-upstream-to origin/feature/21/ansible-zos-raw-module
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
$ git branch --set-upstream-to origin/release-v2.0.0
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
$ git branch --set-upstream-to origin/bugfix/29/zos-raw-module-parse-failure
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
    
In a release based project, old releases might need some maintenance, bug fixes and back ported feature requests. Old 
releases often become incompatible with the most recent versions of projects thus we should encourage users to adopt new releases.
   
To support a paticular release, we will adopt the support branch model identified by an associated Git release tag. Support 
branches are long living branches created to support major and incramental versions of the project. Support branches 
__do not get merged__ back into `master` or `dev`, this __would cause major merge issues__. Instead, commits can be cherry-picked 
from the support branch back into `dev`. 
   
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

2. Now that an incremented release branch `release-v1.0.1` has been created a ![developer](https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer)
can now follow the instructions on how to ![Bugfix old releases](#bugfix-old-releases).
   
3. When you create a pull request to merge `release-v1.0.1` into `support-v1.x`, ensure that the associated fix has been cherry-picked 
into a branch based on `dev` and a pull request is pending. The ![developer](https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#sasha-software-developer)
should have created both a pull request for the `support-v1.x` branch and one for the `dev` branch. 
   
3. Follow the standard release process outlined in ![Create and deploy a release](#create-and-deploy-a-release) 
treating `support-v1.x` as the `master` branch. If you follow the release process, branch `release-v1.0.1` will 
get __deleted__ and `support-v1.x` will remain in the repository indefinitely to bugfix and release as new minor versions.

4. Mark `support-v1.x` as a ![protected branch](https://help.github.com/en/github/administering-a-repository/configuring-protected-branches)
in github so that it does not get accidentally deleted.

To summarize, you are either bug fixing or backporting a feature to an older release that a customer could have in production.
If its a bug, it could be inherint in the `dev` branch, thus propagted to future releases; you will want to make sure you 
cherry-pick the fix accordingly, see [Bugfix old releases](#bugfix-old-releases) also ensure that bugfixes are compatible 
with the current state of `dev`.  
   
__Tip__: Try to maintain as few support branches as possible. These branches are expensive to maintain since need to 
cherry-pick applicable bugfixes into each support branch seperately.

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
$ git branch --set-upstream-to origin/bugfix/79/zos-dat_set-module-validation-failure
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
the latest production releas is based on Note that the `master`. Hotfixes are useful in cases where you want to fix a bug 
in a current released version, but `dev` has new code in it, also why work off of `master`.

1. Create a hotfix branch based off of `master`, fix the issue and include a test case.

```
$ git checkout master
$ git checkout -b hotfix/99/zos_query_job-module-read-failure
$ git branch --set-upstream-to origin/hotfix/99/zos_query_job-module-read-failure
... fix bug ...commit
... add test ...verify ...commit
$ git add .
$ git commit -m "Fix job read failure"
$ git push
```

2. Navigate to the project [ibm_zos_core](https://github.com/ansible-collections/ibm_zos_core) and open a pull request with
the following branch settings:
* Base: `master`
* Compare: `hotfix/99/zos_query_job-module-read-failure`

//TODO: hotfix is on master, need to add instructions to cherry-pick it back into `dev` and `release-branches` similar to ![Bugfix old releases](Bugfix-old-releases)

### Production hotfix release
<!--Release Manager persona icon-->
<a href="https://about.gitlab.com/handbook/marketing/product-marketing/roles-personas/#rachel-release-manager">
<img width="46" alt="image" src="https://user-images.githubusercontent.com/25803172/74109181-12908600-4b36-11ea-9b05-be812d199e48.png">
</a>>
   
1. Now that the hotfix code is in `master` you are ready to create the actual release, assuming or latest production release
is v2.0.0, we will have an incremental increase. Navigate to the project page on Github and draft a new release with the 
following settings:
* Tag version: `v2.0.1`
* Target: `master`
* Release title: `Release v2.0.1 (hotfix)`
* Description: List the features and capabilities this version will include.
* __Click__ `Publish release`.

__Note__: Hotfix releases are actualy releases thus you sould increment at least the minor version of the release.

2. Merge the `hotfix/99/zos_query_job-module-read-failure` into `dev`.

```
$ git checkout dev
$ git merge hotfix/99/zos_query_job-module-read-failure
$ git push
```

5. When the pull request has been reviewed on github:
   * merge the pull request
   * comment and close the pull request
   * delete the `hotfix/99/zos_query_job-module-read-failure` branch

# Citations
  * Atlassian, Bitbucket. [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
