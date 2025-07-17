---
layout: post
title:  "A juniors guide to contributing to ruby events"
date:   2025-07-18 01:36:47 +0300
categories: ruby
---
Assumptions:
You have a github account

Fork repo
clone repo to your local machine
Set up database with ``` rails db:create```
Migrate by running ```rails db:migrate```
Address current issue he/she may run into which is about some migrations and enum declarations.

Seed database with ```rails db:seed```

start application by running ```bin/dev```

Play around with the application. 

Work on your issue of choice:
Currently you're on main
create new branch name describing your issue
while on this new branch, implement your issue
Confirm that it works in browser
run CI with ```bin/lint```. This will autocorrect lint issues if any for example those to do with indentation
Push your branch to github ```git push origin your-branch-name```
Open a PR, in the description, describe what your PR is about and what it fixes linking the issue number. While not mandatory, its a good idea to attach a screenshot/video showing your fix. This makes it easier for the reviewers. 
Upon finally opening your PR, the CI will be triggered by default. This is just a bunch of checks which let you know incase some of those lint issues we talked about earlier are present. 

If none of them are present, which is usually the case if you ran ```bin/lint```, the CI will let you know that all tests ran on your PR pass. If some of them are failing, you'll proceed to check which specific one is failing, make the change to solve it locally, commit, then push your latest changes to Github. The CI will again run automatically and hopefully now, your CI tests will all pass. 

If everything passes, now onto the good part which is the review process. The maintainers will let you know incase of anything that needs to be changed and maybe how to go about it. Still, just like for the failing lint, locally apply the requested changes by the maintainer, commit and push to Github. 

If the CI for your PR passes and that the maintainer has agreed that there are not further changes that are necessary for your PR, your PR will be merged into main. The merging may not happen as quickly as you might want to but eventually it will be merged. 

There are so many issues to work on for the project for all skill levels from juniors to experienced developers. For those that may be going through this article and you stumble upon something upon the way, don't hesitate to reach out. I'll be more than delighted to help out!

