=========
git-revue
=========

git-revue is a lightweight, distributed code review tool for git.

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
commit, or to a specific line in a commit. A comment may stand alone or be in
response to another comment. [diffscuss](https://github.com/hut8labs/diffscuss)
might be a great storage format for comments.


Implementation
--------------

The approval is signed with a single-purpose key created as part of admin
setup. The key should be used solely for git-revue. It must have a password
associated with it, so that even if someone malicious obtains the key, they
cannot forge approvals.

Note the existence of git notes. They might be useful for recording review
messages.


Handling Rebases
----------------

When you rebase a branch, a `git gc` will delete the old commits.

If those commits have comments or approvals, letting that happen removes
important historical data.

How can we prevent that from happening?

Tags are the obvious answer, but that would clutter things badly. If they only
lived in the review repository, that might not be a disaster.

That could be achieved by writing a new git protocol and an associated remote
helper, as described [here](https://rovaughn.github.io/2015-2-9.html).

On pull and fetches, the protocol would not return tags that were created
solely for code review history preservation (unless you ask for them). On
pushes, we detect rebases and create history preservation tags as needed (along
with any necessary metadata). Review preservation tags would follow a naming
scheme so that they're easy to detect.
