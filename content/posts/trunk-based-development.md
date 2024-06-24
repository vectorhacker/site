---
title: "Trunk Based Development"
date: 2024-03-31T00:27:32-04:00
draft: true
---

Trunk-based development is a way of maintaining source code that is popular among teams that practice continuous integration and delivery. It requires merging frequent small changes to the trunk or “main” of the repository, often directly, for it to work properly. Some developers are afraid to adopt this practice, for fear of breaking the codebase and instead opt for the perceived safety of workflows like GitHub Flow or Gitflow. But I argue that this safety is a myth and that trunk-based development, not only a good alternative to these two strategies, but it is also in fact the only proven way to develop quality software faster and the only way to do proper continuous integration and delivery. 

In the past, source version control was costly and labor-intensive, as it required large amounts of work and effort to track changes to software and merge them together as engineers worked on the source code. These tools improved and out came workflows like Gitflow which relies on strict management of feature and hotfix branches, merged into a develop branch which is then merge into main with cherry picks (Driessen, 2010). This complex workflow works well to maintain code quality, but at the cost of speed and merge conflicts as feature branches tend to live for a long time and is difficult to use with CI/CD (Atlassian, n.d.). Trunk-based development instead focuses on using short-lived feature branches that merge directly to trunk (main) or in some cases have developers commit directly to main and it is designed to work well with, and often requires, CI/CD (Hammant, 2017). It is not only operationally simpler, but it has also been shown to be more effective at producing higher quality code and positively affects continuous delivery and that users of this method are among the highest IT performers (Puppet + DORA, 2017). 

Trunk-based development is often compared to GitHub Flow which has a very similar workflow of branching from and merging directly to trunk, but there are two important distinctions. GitHub Flow makes no assertions about the lifetime of feature branches, so they can live for a long time and cause large merge conflicts when trying to integrate and secondly integration happens on a per-request basis via a chatbot that tests and deploys your pull request branch to production on a queue, which locks the main branch, merges to deploy in several steps that take several minutes each, and then monitors each step for problems and then unlocks main. This second distinction is a big one as with large teams, one could be stuck waiting for a merge request to happen and problems to arise and be the chances of merge conflicts go up over time (Thomson, 2018). The trunk-based workflow does not have this problem, each merge request or commit to main is evaluated individually and does not lock the main branch and each developer is required to pull the latest changes from head frequently, thus reducing the possibility of merge conflicts.

So, trunk-based development is simpler and works well with CI/CD, but it must be dangerous to work directly in main, right? Not so, in fact many different companies in various industries (both highly and less regulated) work this way safely. Microsoft (Thomson, 2018), Google (Potvin, 2015), Atlassian, NASA, and SpaceX all work in trunk. SpaceX is a particularly great example, as they attribute working in trunk as a key factor in the safety of their rockets, often pushing changes to trunk and uploading software to rockets on the launchpad up to 45 minutes before a scheduled launch. If SpaceX uses it in the highly regulated industry of making missiles that launch objects into space, so can you. (Rajadurai, 2020) (Lofgren, 2020). 

To setup trunk-based development, certain things must be put in place. A reliable CI/CD infrastructure must be setup from the beginning, investing in good CI/CD is critical to make trunk-based development work. Engineers must write proper tests that they can trust and run every time they commit to trunk. Engineers commit their changes to trunk frequently, ideally at least daily if not more often, so they may integrate their changes frequently and find out quickly whether their changes will work well with everyone else’s. Continuous code review must be performed and often pair-programming along with comments on commits is sufficient, if not implement merge requests of short-lived feature branches. And finally, since engineers are pushing potentially unfinished changes to trunk daily, care must be taken to hide unfinished features from users in the form of feature toggles and dark launching. (Ehn, 2019) (Thomson, 2018)

In conclusion, trunk-based development is not only operationally simpler than more complicated alternatives like Gitflow, but it strongly correlated with high-performing IT practitioners and is the best way of doing continuous integration and delivery. Working on trunk can seem scary but is in fact the best way to deliver high quality software and figure that your changes are going to in fact work with everyone else’s. Trunk-based development speeds up the creation of software in s safe and effective manner. It requires a bit of an upfront investment, but the rewards are well worth it. 


## Sources

Atlassian. (n.d.). Gitflow workflow. Retrieved from Atlasian Git Tutorial: https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow

Driessen, V. (2010, January 5). A successful Git branching model. Retrieved from nvie.com: https://nvie.com/posts/a-successful-git-branching-model/

Ehn, J. (2019, June 5). Real Programmers Commit To Master . Retrieved from YouTube: https://www.youtube.com/watch?v=hL1OZfgoZGk&t=1271s&pp=ygUXdHJ1bmsgYmFzZWQgZGV2ZWxvcG1lbnQ%3D
Hammant, P. (2017). Trunk Based Development: Introduction. Retrieved from Trunk Based Development: https://trunkbaseddevelopment.com/#history

Lofgren, E. (2020, December 4). Is continuous testing the key to SpaceX software success? Retrieved from Acquisition Talk: https://acquisitiontalk.com/2020/12/is-continuous-testing-the-key-to-spacex-software-success/
Potvin, R. (2015, September 14). Why Google Stores Billions of Lines of Code in a Single Repository. Retrieved from YouTube: https://youtu.be/W71BTkUbdqE

Puppet + DORA. (2017). 2017 State of DevOps Report. Puppet + DORA. Retrieved from https://services.google.com/fh/files/misc/state-of-devops-2017.pdf

Rajadurai, N. (2020, June 25). How SpaceX develops software. Retrieved from {Coders} Kitchen: https://www.coderskitchen.com/spacex-software-development-and-testing/

Thomson, E. (2018, May 9). Git patterns and anti-patterns for successful developers: Build 2018. Retrieved from YouTube: https://youtu.be/ykZbBD-CmP8

