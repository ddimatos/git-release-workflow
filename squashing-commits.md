# How to Squash Commits
When submitting a pull request, you must squash your commits before we merge. Squasing commits can be intimidating at first, once you figture it out you will see how helpful it is and clean it keeps the git commit history.  

## Squashing Options
* If you are using GitHub for a pull request, it provides you an option to squash before submitting a pull request. 
  * TODO: GitHub instructiosn and images
* From the terminal:
    *  Make sure your branch is up to date with the master branch
    * Run git rebase -i master. 
      * This will ensure that your commit is at the top of the branch pulling in the latest changes.
    * You should see a list of commits, each commit starting with the word "pick"
    * Make sure the first commit says "pick" and change the rest from "pick" to "squash". 
      * This will squash each commit into the previous commit, which will continue until every commit is squashed into the first commit.
    * Save and close the editor
    * It will give you the opportunity to change the commit message
    * Save and close the editor again
    * Then you have to force push the final, squashed commit: git push --force-with-lease origin
