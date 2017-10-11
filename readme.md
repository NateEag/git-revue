git-revue
=========

git-revue is a distributed code review tool for git.

It provides command-line and web interfaces. The web interface serves as
both code browser and code review tool.

The name is a reference to a [revue](http://en.wikipedia.org/wiki/Revue), which
is in some sense a review of the thing set before the viewer. It was chosen
largely because 'git-review' was already taken.

It also does not yet exist. These are just my design notes on how it might be
built.


Design
------

Commits can be approved for merging into a branch. Approvals are signed,
annotated tags. Their tag message contains the branch the commit is approved to
merge into.

A given repository may be fitted with a pre-receive hook that rejects pushes
that merge an unapproved commit ID into a protected branch. Branches are
protected by configuring the pre-receive hook's list of protected branches. The
hook can be set to allow merging commits that yield an identical patch to
to merging an approved commit ID.

Anyone can use the web or command-line interfaces to leave a comment on a
commit. A comment may apply to the whole commit, to a specific file in the
commit, to specific lines in a commit (usually in one file), or even specific
characters within a line in a commit. It might be useful to allow comments
on diffs as such, too, and to treat the 'comment on commit' as a diff from
commitref^..commitref?

A comment may stand alone or be in response to another comment.

[diffscuss](https://github.com/hut8labs/diffscuss) might be a good format for
reading and writing reviews. They could be stored, pushed, and pulled as git
notes or similar.


Implementation
--------------

The approval is signed with a single-purpose key created as part of admin
setup. The key should be used solely for git-revue. It must have a password
associated with it, so that even if someone malicious obtains the key, they
cannot forge approvals.

Note the existence of git notes. They might be useful for recording review
messages.

Consider using custom git objects? With a custom protocol, pushing them around
might be pretty simple, and we could perhaps avoid polluting the tag and branch
lists. However, that leaves us at risk of breaking the rebase-following behavior.


Handling Rebases
----------------

When you rebase a branch, a `git gc` can delete the old commits.

If those commits have comments or approvals, letting that happen removes
important historical data.

How can we prevent that from happening?

Tags are the obvious answer, but that would clutter things badly. If they only
lived in the review repository, that might not be a disaster.

On pull and fetches, by default we should not return tags that were created
solely for code review history preservation (unless you asked for them).

That could be achieved by storing review-related tags in
[namespaced refspecs](https://git-scm.com/book/en/v2/Git-Internals-The-Refspec#Pushing-Refspecs),
and having git-revue configure remotes with custom fetch configurations, so you
don't retrieve a mountain of rebase-preservation tags unless you want them. We
could also use custom commands as wrappers around push/fetch with custom
refspecs, to keep the UI cleaner.

On pushes, we detect rebases and create history preservation tags as needed
(along with any necessary metadata), in a namespace such as
ref/tags/review-history.

That could be done by writing a new git protocol and an associated remote helper,
as described [here](https://rovaughn.github.io/2015-2-9.html). However, a
simpler alternative would be a pre-receive or update hook in the review repo -
if a new ref is not a fast-forward, create the necessary history preservation
tags. If anything fails, exit non-zero, so that history cannot be lost by
accident.

This is not truly distributed - for that, the hooks would probably need to be
pre-rebase (and other history-changing comands), but you would then have to
push the tags manually.

What about cases where you need to nuke an old branch post-rebase (due to a
copyright violation, for instance)? Well, should be simple enough - manually
delete the references and demand git take out the garbage, per usual.


Existing Projects
-----------------

[git-appraise](https://github.com/google/git-appraise) appears to cover a lot of
this ground. I did most of the above design work before I realized it existed.

[git-series](https://github.com/git-series/git-series) looks like it does the heavy
lifting around tracking rebases I want. It would be instructive to understand what
approach it takes.
