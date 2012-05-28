---
layout: post
title: "Packing up: Moving Our Grails Plugins From SVN to Git"
date: 2012-05-28 16:35
comments: true
categories: Grails SVN Git
---

As part of the migration from SVN over to Git in our company, we also need to transport our internal Grails plugins to the new repo structure. These plugins have been developed since Grails 1.1.x, and as such, there's a thing or two that have changed over the years - in terms of how grails plugins are stored in a repo and how the published artifacts are resolved.

<!--more-->

When our internal plugin development process started, we were still using Grails 1.1. Grails plugins were a wonderful way for us to encapsulate functionality, so that we didn't have to reinvent the wheel with every new customer project.

Grails 1.1 relied heavily on SVN, whereas starting from 1.2 versions of Grails, migration towards [Apache Ivy](http://ant.apache.org/ivy/) dependency resolution began to take place. Back then, the _right way_ to publish plugins was to store them in an SVN repository as .zip files. While this feels "unclean" to how I see repos should be used, it was supported out of the box in Grails, and the process of using it was rather straightforward, so we stuck with it.

Fast-forward over to Grails 2.x, and the _correct_ way to handle plugins is to set up an Ivy/Maven compliant repository and publish the .zip artefacts there. In fact, the old way of having the .zip artefacts in SVN is [no longer](http://jira.grails.org/browse/GPRELEASE-36) supported at all.

So, to move over to Git, we would need to do two things:

1. Use _git svn_ to init the new repo and fetch the changes
2. Create an authors file to link committers across repositories
3. We need to get rid of the .zip files in our new git repo and move them to [Artifactory](http://www.jfrog.com/products.php) instead

To do this, we would use three commands:

    $ git svn init --no-minimize-url [SVN-repo]

The init command sets up a new git repo in an empty local dir, and _svn init_ adds the appropriate remote refs to Git. Our SVN repo was laid out so that each plugin was in a subfolder in the repo root, and that subfolder followed the standard trunk/tags/branches layout, so I added _--no-minimize-url_ flag to ensure that git svn considered the provided url the root of the repo, and would not attempt to navigate down towards the actual repo root.

    $ git svn fetch --authors-file=[authors-file]
    
Then I did an svn fetch for the remote refs, providing an authors file so that committers could be mapped correctly to Git. And authors file is a simple text file with each line corresponding to a different author:

    lpiispanen = Lauri Piispanen <lauri.piispanen@my-email-address-is-gone.com>
    jdoe = John Doe <john.doe@my-email-address-is-gone.com>

Finally, I wanted to get rid of all the published artefacts in the SVN dirs, and _filter-branch_ allowed me to do just that. An additional benefit of _filter-branch_ is that it shrinks down the entire repo size - as if the artefacts had never been committed in the first place. You probably would not want to do this on an existing repo, since then the clones that people have already taken will face some nasty history conflicts - but as we're working on a brand new Git repo, it's fine:
    
    $ git filter-branch --index-filter 'git rm --cached --ignore-unmatch grails-*.zip' HEAD


That's it! We now have a clean git repo, ready to be pushed to a different remote. Now, the next step is to push the .zip artefacts to Artefactory and we can archive the SVN repo entirely.
