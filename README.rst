Yet another git repository about git
====================================

.. code:: bash. that assumes you already know the pure basics and need help for a special case or want to recall some "advanced basics".

In truth, it's a log of what I keep forgetting, had trouble to look up or that I deemed worthy of writing down. This is most likely going to evolve like patches on a coat. I recommend using it as a reminder or a first input to looking up what you want to do. Maybe it will help someone else out there.

Here's the yet to be historically grown table of contents:

- `Bisect`_
- `Tags`_
- `History Navigation`_
- `Rebase`_
- `Diff Remote Changes`_
- `Change a Merge`_
- `Merging Unrelated History (or Combining Two Projects)`_


Bisect
------

The most essential bug hunting tool too few people know.

Scenario: Some commit between aa and dd introduced a bug and you don't know which one. Try this:

.. code:: bash

	git bisect start

And the game begins.. code:: bash. This commit has the bug, so mark it as:

.. code:: bash

	git bisect bad

dd is fine, so mark it as:

.. code:: bash

	git bisect good HASHOFDD

Keep marking commits as good or bad and it will take you where you want to go.

At the end return with:

.. code:: bash

	git bisect reset

Check out https://git-scm.com/docs/git-bisect for more hunting tools.



Tags
----

Those commands you always recover from history until once they are not there.. code:: bash.

Create a tag, the "thorough" way:

.. code:: bash

	git tag -a TAGNAME -m "TAG DESCRIPTION"

Create a tag for a certain commit:

.. code:: bash

	git tag -a TAGNAME -m "TAG DESCRIPTION" COMMITHASH

I introduced a tag for this. - What? Where? Push your tag:

.. code:: bash

	git push origin TAGNAME

or all of them (given they are annotated and on the same branch):

.. code:: bash

	git push --follow-tags

Show a tag commit:

	git show TAGNAME



History Navigation
------------------

History is never clean and if it is, it's fake. Prove me wrong with tree view:

.. code:: bash

	git log --graph --oneline --all



Diff Remote Changes
-------------------

Do I really want to merge this?

.. code:: bash

	git fetch origin main
	git diff origin/main

Diff only a file:

.. code:: bash

	git fetch origin main
	git diff origin/main -- /path/to/file/of/interest

Where ``--`` separates parameters (before) from filenames (afterwards). Sidenote: Git now also has the ``--end-of-options`` parameter, which does the same but is a bit safer.



Rebase
------

Now I'm finished, let's change it all again before pushing upstream. Rebase all commits not yet pushed interactively:

.. code:: bash

	git rebase -i

Rebase ``n`` commits:

.. code:: bash

	git rebase -i HEAD~n

Make ``push -f`` really worth it! Rebase all commits in your repository interactively:

.. code:: bash

	git rebase -i --root



Change a Merge
--------------

Yeah I take that.. code:: bash. but not all of it. If you want to influence a merge, best not hope for a conflict and tell git not to commit automatically:

.. code:: bash

	git fetch A
	git merge --no-commit A/main

Where ``A`` is the repository name and ``main`` is the current repository's main branch that you want to merge into.

Now you can ignore changes:

.. code:: bash

	git restore FILENAME

Or fix them and eventually merge:

.. code:: bash

	git commit



Merging Unrelated History (or Combining Two Projects)
-----------------------------------------------------

Possible Scenario: Repository A was copy-pasted as a new repository B at some point in time, without preserving the git metadata. Now you want to merge it back again. Standard merge procedure (in repository A - or a copy of it, to guard against mistakes):

.. code:: bash

	git remote add B ~/path/to/B
	git fetch B

The merge will fail because there is no common commit within the two projects unless you do the following:

.. code:: bash

	git merge B/main --allow-unrelated-histories

Where ``B`` is the repository name and ``main`` is A's main branch that you want to merge into.


Alternative Approach
~~~~~~~~~~~~~~~~~~~~

This is the other way round. You take repository B and assign all commits in repository A older than the oldest one in B as B's parents. This time, run from repository B - or a copy of it, to guard against mistakes:

.. code:: bash

	git remote add A ~/path/to/A
	git fetch A
	git replace --graft $(git log B/main --format=%H | tail -1) A/main

Where ``A`` and ``B`` are the repository names, ``main`` is the respective repository's main branch and ``$(git log B/main --format=%H | tail -1)`` returns the initial commit from repository B (can be replaced by the hash instead).

Credit for this solution goes to dahlbyk on `stackoverflow <https://stackoverflow.com/questions/42292219/merge-two-git-repos-and-keep-the-history/42457384#42457384>`_

So far, this solution preserves the commit hashes and replace references can be pushed to share them with others. To rewrite history and thereby change commit hashes, you can do the following.

Make the replacement permanent as if A and B were never split:

.. code:: bash

	git filter-branch --prune-empty --tag-name-filter cat -- --all

Where ``--prune-empty`` removes commits with no changes included due to the history changes and ``--tag-name-filter cat`` preserves the tags. If you do not want to preserve the tags, delete them first.

Then clean up if you absolutely want to (delete filter-branch backups, delete unreferenced commits, run garbage collection) - DANGER ZONE:

.. code:: bash

	git reset --hard
	git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
	git reflog expire --expire=now --all
	git gc --aggressive --prune=now

See also: https://git-scm.com/docs/git-filter-branch#_checklist_for_shrinking_a_repository

By the way, you can also use grafts to remove parents, i.e. "squash" all your changes up to a certain commit into one commit - without using squash.
