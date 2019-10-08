# Git Workflow
Last updated 10/08/19 by [Antonio Manueco](mailto:antonio@betablocks.co)


## General Workflow
Our workflow is a feature-branch workflow based around rebasing rather than merging. This has a few advantages:

* Simpler, linear git history
* Reduction of merge conflicts
* Ability to squash down multiple atomic commits into a concise, logical change history

To apply this workflow, the `--rebase` flag must be used when pulling. To have this happen by default, see the next section.

## Git Configuration

### Automatic Rebase When Pulling 

    git config branch.autosetuprebase always # Force all new branches to automatically use rebase
    git config branch.*branch-name*.rebase true # Force existing branches to use rebase.



## Branches

### Permanent Branches
_master_: the latest stable code. This is the branch on the production servers

_develop_: primary working branch 


### Working Locally with Feature Branches
All continuing development should be done in feature branches branched off from _develop_. Create a feature branch that includes the Pivotal story id (if applicable) and a description:

    git checkout develop
    git pull --rebase origin develop
    git checkout –b PIVOTALID_DESC
    git push -u origin PIVOTALID_DESC
    

##### Updating a feature branch
If a feature branch exists for a long enough time, significant changes will occur in _develop_. It is best to update your feature branch periodically rather than deal with many potential merge conflicts at the end. To update the feature branch, rebase the parent branch into it. This ensures the feature branch history will stay clean, legible and squashable.

    git checkout FEATURE_BRANCH
    git fetch origin develop
    git rebase origin/develop
    (fix any conflicts)
    
After rebasing, your local git history will differ from the version on GitHub. To update GitHub, you must force push. **WARNING: Only force push a branch you are not sharing with over developers.**

    git push -f origin FEATURE_BRANCH
    
##### Advanced: Simplying Feature Branch with Interactive Rebase
Once you are done working on a feature branch, and before merging it back into develop, you may choose to cleanup your branch's commit history. To do so, use interactive rebase

    git rebase -i origin/develop

This displays an interactive rebase window that you can use to combine (squash) commits into one, edit commit messages, or delete commits (be careful).

The benefit of this method is that it allows the best of both worlds. During development, you may make many small, atomic commits following the best practice of "commit often". However, when you are ready to merge back into _develop_, it is usually better to have just a few logical commits that allow anybody to read the git history and understand what each commit does.

##### Pull Requests
Once your feature branch is ready to be merged back into _develop_, make sure you that you follow the instruction above in order to rebase the parent branch (in this case _develop_) then submit the Pull Request.
- Pull Request should also delete feature branch automatically upon merge to avoid conflicts.

##### Merging Feature Branch Back into Develop (_Manually_)
Once your branch is ready to be merged back into _develop_, use these commands to first update your feature branch (same as above), and then merge it back into develop.

    git checkout FEATURE_BRANCH
    git fetch origin develop
    git rebase origin/develop
    (fix any conflicts)
    git checkout develop
    git pull --rebase origin develop
    git merge FEATURE_BRANCH
    
##### Cleanup
To delete a local branch:

	git branch -d BRANCH_NAME
	
To delete the remote on GitHub:

	git push origin :BRANCH_NAME
   

### Transient Release Candidate Branches
When it is nearing time to release, a release branch is cut from _master_

    git checkout master
    git pull --rebase origin master
    git checkout –b RC_NAME_DATE


This branch is then polished and continually deployed to the staging server for testing. When the release is deemed ready, the RC branch is merged into _master_ (fast-forwarded), tagged with the version number (see below), and _master_ is deployed to production. The RC branch is also rebased onto _develop_. Then the RC branch can be deleted. 

Using release candidate branches allows us to polish a release while development continues on _develop_ unhindered, as well as leaving _master_ untouched until the last second to allow for any hotfixes that may arise during the polish phase.


#### Tagging

To tag a tested release as deployable:

	git tag -a RELEASE_NAME -m 'RELEASE NAME'
	git push origin RELEASE_NAME


### Hot Fixes
If a hot fix is required, a hot fix branch is branched off of _master_.

    git checkout master
    git pull --rebase origin master
    git checkout –b PIVOTALID_hotfix_DESC

This hotfix branch is deployed and tested on the staging server until it is deemed ready. It is then merged into _master_, and _master_ is deployed to production. The hotfix is also back-merged to _develop_, to ensure the bug is not re-introduced in the next release.

Note: Extreme caution must be taken when hotfixing _master_.

    git checkout master
    git pull --rebase origin master
    git merge HOTFIX_BRANCH
    git push origin master
    git checkout develop
    git pull --rebase origin develop
    git merge HOTFIX_BRANCH
    git push origin develop
    
## No Nos!
- Following this Git Flow, there should be no `Merge commits` as part of the history.  It creates an opaque history and it becomes difficult to read through the history of commits without having to go into individual code commits.
