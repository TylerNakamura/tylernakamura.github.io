---
layout: post
title: "Use Git with Purpose"
date: 2016-06-25 17:16
comments: false
published: true 
categories: Git revision control pull request commit
---

For the uniniated, git is a revision control software that allows developers to collaborate effectively.
Git is especially powerful as it can be used locally, or in collaboration with a remote repository.
Git does this by tracking changes to plain text files.
By tracking changes, you can easily roll back, compare or view any "save-point" of a given project.
It also allows for developers to be working on different branches of a single project with the intention of eventually merging the branches to create the final product.

A large set back to git is the learning curve. 
When I first began to use git, I often became discouraged when trying to learn how to use it in my work-flow.
There are so many steps from the point of writing code, to just "saving" it.
For example, if you want to simply save your code to a remote repository for others to see, it requires that you first add your code to an index, commit the changes with a commit message and then push to a repository.
This is not accounting for the possibility that there is a merge conflict in the cloud.
A **very simple** work-flow might look like this when using git:

```bash
git pull origin master
vim "awesome-code.py"
git add awesome-code.py
git commit -m 'make changes to code'
git push origin master
```

At this point, many of us have been computer users from a very young age and we have been accustomed to simply saving our work by hitting ```ctr+s```.
Going into an arcane "terminal" and then executing all these commands seems highly ludicrous for such a simple task.

After finally gitting the hang of it, I learned to automate these tasks so that my work-flow might even be faster.
I am so accustomed to executing git commands, that sometimes I even precede a standard unix command with "git": ``` git ls```.

It was at this point that I realized that I was just using git as a save button.
**Git is not a save button**.
Using it as such renders your commit log useless and full of uneventful commits:

```
1. made some changes
2. grammar fix
3. changes
4. introduced a bug
5. wrote a for loop
6. updates
```

I was told to commit early and often.
Githubs "contribution graph" encourages people to commit useless commits to the master branch for the purpose of seeing that beautiful little green box.
It wasn't until working in an actual team enterprise environment that I realized that this is not the way it works in the real world.
Below are some main things that I wish I knew when getting started with git:

### Preserve the stability of the master branch
There are too many times when we just use the master branch because it is easier.
Sometimes, in the case of blogs, or minor grammar fixes, working on master branch is acceptable.
But for larger coding projects, as the project continues to grow, the necessity for branching grows.

### Create a commit history that tells a story
Don't fill the commit history with useless commits.
Other team members should be able to look at the commit log and know what the commit is about without having to actually look at the code.

### Each commit is a project within itself
Don't introduce changes that require many commits to implement.
Try to keep each commit or branch as modular as possible.
This means, before committing to master branch, do testing!

Git gives you the ability to do anything. 
For a beginner, this responsibility is often misused. 
Git is not a save button, commit with purpose!

