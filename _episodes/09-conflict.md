---
title: Conflicts
teaching: 15
exercises: 0
questions:
- "What do I do when my changes conflict with someone else's?"
objectives:
- "Explain what conflicts are and when they can occur."
- "Resolve conflicts resulting from a merge."
keypoints:
- "Conflicts occur when two or more people change the same lines of the same file."
- "The version control system does not allow people to overwrite each other's changes blindly, but highlights conflicts so that they can be resolved."
---

As soon as people can work in parallel, they'll likely step on each other's
toes.  This will even happen with a single person: if we are working on
a piece of software on both our laptop and a server in the lab, we could make
different changes to each copy.  Version control helps us manage these
[conflicts]({{ page.root}}{% link reference.md %}#conflict) by giving us tools to
[resolve]({{ page.root }}{% link reference.md %}#resolve) overlapping changes.

To see how we can resolve conflicts, we must first create one.  The file
`b117.txt` currently looks like this in both partners' copies of our `variants`
repository:

~~~
$ cat b117.txt
~~~
{: .language-bash}

~~~
Almost all sequences carry the N501Y mutation.
But P681H is also very common.
And of course it contains D614G.
Note that P.1 also contains N501Y.
~~~
{: .output}

Let's add a line to the collaborator's copy only:

~~~
$ nano mars.tx
$ cat b117.txt
~~~
{: .language-bash}

~~~
Almost all sequences carry the N501Y mutation.
But P681H is also very common.
And of course it contains D614G.
Note that P.1 also contains N501Y.
This line added to Wolfman's copy.
~~~
{: .output}

and then push the change to GitHub:

~~~
$ git add b117.txt
$ git commit -m "Add a line in our home copy"
~~~
{: .language-bash}

~~~
[main 5ae9631] Add a line in our home copy
 1 file changed, 1 insertion(+)
~~~
{: .output}

~~~
$ git push origin main
~~~
{: .language-bash}

~~~
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 331 bytes | 331.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/vlad/variants.git
   29aba7c..dabb4c8  main -> main
~~~
{: .output}

Now let's have the owner
make a different change to their copy
*without* updating from GitHub:

~~~
$ nano b117.txt
$ cat b117.txt
~~~
{: .language-bash}

~~~
Almost all sequences carry the N501Y mutation.
But P681H is also very common.
And of course it contains D614G.
Note that P.1 also contains N501Y.
We added a different line in the other copy
~~~
{: .output}

We can commit the change locally:

~~~
$ git add b117.txt
$ git commit -m "Add a line in my copy"
~~~
{: .language-bash}

~~~
[main 07ebc69] Add a line in my copy
 1 file changed, 1 insertion(+)
~~~
{: .output}

but Git won't let us push it to GitHub:

~~~
$ git push origin main
~~~
{: .language-bash}

~~~
To https://github.com/vlad/variants.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/vlad/variants.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
~~~
{: .output}

![The Conflicting Changes](../fig/conflict.svg)

Git rejects the push because it detects that the remote repository has new updates that have not been
incorporated into the local branch.
What we have to do is pull the changes from GitHub,
[merge]({{ page.root }}{% link reference.md %}#merge) them into the copy we're currently working in, and then push that.
Let's start by pulling:

~~~
$ git pull origin main
~~~
{: .language-bash}

~~~
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0
Unpacking objects: 100% (3/3), 326 bytes | 163.00 KiB/s, done.
From https://github.com/vlad/variants
 * branch            main       -> FETCH_HEAD
   a372fc3..a296941  main       -> origin/main
Auto-merging b117.txt
CONFLICT (content): Merge conflict in b117.txt
Automatic merge failed; fix conflicts and then commit the result.
~~~
{: .output}

The `git pull` command updates the local repository to include those
changes already included in the remote repository.
After the changes from remote branch have been fetched, Git detects that changes made to the local copy
overlap with those made to the remote repository, and therefore refuses to merge the two versions to
stop us from trampling on our previous work. The conflict is marked in
in the affected file:

~~~
$ cat b117.txt
~~~
{: .language-bash}

~~~
Almost all sequences carry the N501Y mutation.
But P681H is also very common.
And of course it contains D614G.
Note that P.1 also contains N501Y.
<<<<<<< HEAD
We added a different line in the other copy
=======
This line added to Wolfman's copy.
>>>>>>> a2969411348ddbf09da601bff5b24ed1ee26ed7e
~~~
{: .output}

Our change is preceded by `<<<<<<< HEAD`.
Git has then inserted `=======` as a separator between the conflicting changes
and marked the end of the content downloaded from GitHub with `>>>>>>>`.
(The string of letters and digits after that marker
identifies the commit we've just downloaded.)

It is now up to us to edit this file to remove these markers
and reconcile the changes.
We can do anything we want: keep the change made in the local repository, keep
the change made in the remote repository, write something new to replace both,
or get rid of the change entirely.
Let's replace both so that the file looks like this:

~~~
$ cat b117.txt
~~~
{: .language-bash}

~~~
Almost all sequences carry the N501Y mutation.
But P681H is also very common.
And of course it contains D614G.
Note that P.1 also contains N501Y.
We removed the conflict on this line.
~~~
{: .output}

To finish merging,
we add `b117.txt` to the changes being made by the merge
and then commit:

~~~
$ git add b117.txt
$ git status
~~~
{: .language-bash}

~~~
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 1 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
	modified:   b117.txt
~~~
{: .output}

~~~
$ git commit -m "Merge changes from GitHub"
~~~
{: .language-bash}

~~~
[main c50ef97] Merge changes from GitHub
~~~
{: .output}

Now we can push our changes to GitHub:

~~~
$ git push origin main
~~~
{: .language-bash}

~~~
Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 8 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (9/9), 1.02 KiB | 522.00 KiB/s, done.
Total 9 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
To https://github.com/vlad/variants.git
   a296941..c50ef97  main -> main
~~~
{: .output}

Git keeps track of what we've merged with what,
so we don't have to fix things by hand again
when the collaborator who made the first change pulls again:

~~~
$ git pull origin main
~~~
{: .language-bash}

~~~
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 4), reused 6 (delta 4), pack-reused 0
Unpacking objects: 100% (6/6), done.
From https://github.com/vlad/variants
 * branch            main     -> FETCH_HEAD
   a296941..c50ef97  main -> main
Updating a296941..c50ef97
Fast-forward
 b117.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
~~~
{: .output}

We get the merged file:

~~~
$ cat b117.txt
~~~
{: .language-bash}

~~~
Almost all sequences carry the N501Y mutation.
But P681H is also very common.
And of course it contains D614G.
Note that P.1 also contains N501Y.
We removed the conflict on this line.
~~~
{: .output}

We don't need to merge again because Git knows someone has already done that.

Git's ability to resolve conflicts is very useful, but conflict resolution
costs time and effort, and can introduce errors if conflicts are not resolved
correctly. If you find yourself resolving a lot of conflicts in a project,
consider these technical approaches to reducing them:

- Pull from upstream more frequently, especially before starting new work
- Use topic branches to segregate work, merging to main when complete
- Make smaller more atomic commits
- Where logically appropriate, break large files into smaller ones so that it is
  less likely that two authors will alter the same file simultaneously

Conflicts can also be minimized with project management strategies:

- Clarify who is responsible for what areas with your collaborators
- Discuss what order tasks should be carried out in with your collaborators so
  that tasks expected to change the same lines won't be worked on simultaneously
- If the conflicts are stylistic churn (e.g. tabs vs. spaces), establish a
  project convention that is governing and use code style tools (e.g.
  `htmltidy`, `perltidy`, `rubocop`, etc.) to enforce, if necessary

> ## Solving Conflicts that You Create
>
> Clone the repository created by your instructor.
> Add a new file to it,
> and modify an existing file (your instructor will tell you which one).
> When asked by your instructor,
> pull her changes from the repository to create a conflict,
> then resolve it.
{: .challenge}

> ## A Typical Work Session
>
> You sit down at your computer to work on a shared project that is tracked in a
> remote Git repository. During your work session, you take the following
> actions, but not in this order:
>
> - *Make changes* by appending the number `100` to a text file `numbers.txt`
> - *Update remote* repository to match the local repository
> - *Celebrate* your success with some fancy beverage(s)
> - *Update local* repository to match the remote repository
> - *Stage changes* to be committed
> - *Commit changes* to the local repository
>
> In what order should you perform these actions to minimize the chances of
> conflicts? Put the commands above in order in the *action* column of the table
> below. When you have the order right, see if you can write the corresponding
> commands in the *command* column. A few steps are populated to get you
> started.
>
> |order|action . . . . . . . . . . |command . . . . . . . . . . |
> |-----|---------------------------|----------------------------|
> |1    |                           |                            |
> |2    |                           | `echo 100 >> numbers.txt`  |
> |3    |                           |                            |
> |4    |                           |                            |
> |5    |                           |                            |
> |6    | Celebrate!                | `AFK`                      |
>
> > ## Solution
> >
> > |order|action . . . . . . |command . . . . . . . . . . . . . . . . . . . |
> > |-----|-------------------|----------------------------------------------|
> > |1    | Update local      | `git pull origin main`                     |
> > |2    | Make changes      | `echo 100 >> numbers.txt`                    |
> > |3    | Stage changes     | `git add numbers.txt`                        |
> > |4    | Commit changes    | `git commit -m "Add 100 to numbers.txt"`     |
> > |5    | Update remote     | `git push origin main`                     |
> > |6    | Celebrate!        | `AFK`                                        |
> >
> {: .solution}
{: .challenge}
