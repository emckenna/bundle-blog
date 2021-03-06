## Git Bundles
I very much enjoy working with Git.  After working with CVS and Subversion so many years, Git takes the pains out of using a version control system.  I'm always finding out new things about Git and how flexible it is.  My most recent discover has been with utilizing bundles.  If you are not familiar with Git bundles, here is an intro, http://git-scm.com/2010/03/10/bundles.html, but simply put, bundles are a way to package up a set of commits which can then be transferred by any means available, and committed to another repository.

## Why did I need to use bundles
Our clients utilize servers with restricted access.  Part of our team (me) had access, the other part was in the process of getting access.  We wanted to get started ASAP so I was reminded that git was built with distributed repositories and I was tasked with sorting out a solution.  After a trip to google, I stumbled upon Git bundles.  

## I started off doing it wrong.
To Begin, I read over the tutorial offered by git-scm.com to get some background.  Feeling better about what I was doing, I wanting to get something working, so I jumped right into creating bundles for two repos, going back and forth about If I was including too much or too little in the way of branches, mostly referring to the master and current development branch.   After deciding on branches, I created the bundles and moved those bundles over to the accessible development server.  My next step was to clone two repos from those bundles, one for each developer.  At this point, my setup was the following:  
* repo1.bundle -> cloned by dev1
* repo2.bundle -> cloned by dev2

Not too bad.  The first sign of trouble appeared when discovering 'git push' commands failed.  We still had our commits, but just added overhead of then having to create bundles of each developer's workspace and move those commits back up.  But only managing two groups of commits this way, while a bit wonky, was not too painful.  Not yet anyway.  The first real issue cropped up when I needed to contribute to a branch already being worked on.  While the commits were separate, I was looking at having to manually merge two groups of commits which just seemed like a very bad idea.   Now I was staring at: 
* repo1.bundle -> cloned by dev1
* repo2.bundle -> cloned by dev1
* repo2.bundle -> cloned by dev2

Then later that morning, two more developers were being added to do work on another branch,  which duplicated the very issue that really made me question how I was managing the code.  With two more developers, I was now standing with: 
* repo1.bundle -> cloned by dev1
* repo2.bundle -> branch-1, cloned by dev1
* repo2.bundle -> branch-1, cloned by dev2
* repo2.bundle -> branch-2, cloned by dev3
* repo2.bundle -> branch-2, cloned by dev4
 
This made my head hurt thinking about the time it was going to take just to correctly shuttle all the commits back and forth.
There had to be a better way.  

## Then I got it right!
So after seeing the headache coming with my setup, I started asking others for some insight and we realized that what we really needed was just a single clone of each bundle pulled from the restricted code base.  Then, by creating those clones as bare repositories, we were then able to interact with our 'distributed' repository, just like normal.  Now my setup was looking like this:
* repo1.bundle -> cloned-bare -> cloned by dev1
* repo2.bundle -> cloned-bare -> cloned by dev1, dev2, dev3, dev4

I'll still need to bundle up the changes from these repos and move them back to the restricted space, but that is expected behavior.  I guess the lesson to really learn here is that if you are using an established piece of software and your use of it is making your job harder, you are using the tool incorrectly (trust your instincts), or there is something better. ;)

## Example
    // find a place to work, i'm going to use /tmp
    cd /tmp
    // Clone the repo
    git clone https://github.com/emckenna/bundle-blog.git -b example true-origin
    // create the initial bundle
    git --git-dir=true-origin/.git bundle create tru-orig.bundle example
    // create our bare repository from the initial bundle @@
    git clone --bare /tmp/eric/tru-orig.bundle -b example /tmp/remote/example-bare-clone
    // change to our simulated remote environment
    cd remote
    // clone from bare
    git clone example-bare-clone/ example-clone
    cd example-clone
    // edit README.md
    git add README.md
    git commit -m 'a commit to bundle'
    // push our changes normally
    git push origin example
    // look for our changes
    git log --oneline -5  // to see your commits
    // package up and move the commits to the true-origin
    cd /tmp/remote
    git --git-dir=example-bare-clone bundle create commits.bundle example

    // back to our origin repository
    cd /tmp/true-origin
    git pull /tmp/remote/commits.bundle example:example
    // check for commits
    git log --oneline -5 // to see last commits
    git status //shows that we have stuff to push
    // push to true original repository
    git push origin example

@@ if you already cloned, you can set as bare repository with 'git config --bol core.bare true' and then remove all files leaving the .git directory. by you will also need to specifiy the .git folder if using --git-dir option e.g. /tmp/remote/example-bare-clone/.git
