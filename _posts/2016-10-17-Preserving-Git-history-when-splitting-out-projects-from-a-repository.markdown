---
layout: post
title:  "Preserving Git History When Splitting Out Projects From A Repository"
date:   2016-10-17 10:14:21
categories: git history
---
When starting a new project from scratch, I often just create a repository and start any projects needed in there. I use this monolithic-style approach because I mostly have worked in startup environments where it is often difficult to determine the various needs of a new application at the outset.  I have found this approach to be the easiest way to get started and get new services up and running for a demo or early beta release in an evironment of uncertainty.  The sooner you get your applications up and running, the sooner you will gain valuable domain experience and will be able to make business and architectural decisions based on this knowledge.  

After more is learned about contextual boundaries and business needs, you can then refactor some projects into their own repositories.  When I get to this point in time, I usually like to keep the history of the original subproject since this history can often have valuable learning opportunities, as well as provide a good reference if previous configurations or functionality is needed.

Here is a step-by-step way of porting a subfolder from a monolithic project into a new repository while preserving the history specific to the subproject:

1. Create a temporary branch on the source repo 
    - `git checkout -b temp-move-branch`
2. Collapse the subproject source folder down to the repository root. This effectively makes the source subproject the root of the repo. This seems like a drastic step, but keep in mind this change is taking place only on a temporary brach.
    - `git filter-branch --subdirectory-filter path/to/target/subfolder`
3. Create the new repo (presumably in Github)
4. Clone the new repo locally
    - `git clone …`
5. In the new repo, set the source repo’s collapsed branch as an upstream target
    - `git remote add -t temp-move-branch upstream file:///path/to/original/repo`
6. Create a branch on the new repo and checkout to it
    - `git checkout -b temp-move-branch`
7. Merge in the code from the upstream remote
    - `git fetch upstream`
    - `git branch -u upstream/temp-move-branch`
    - `git rebase upstream/temp-move-branch`
8. Merge to master
    - `git checkout master`
    - `git rebase temp-move-branch`
    - `git push -f`
9. Remove upstream config from new repo
    - `git remote remove upstream`
10. Remove temporary branch in both repos
    - `git branch -D temp-move-branch`
11. Remove the original subproject source files from master of the original repo and you are finished!


