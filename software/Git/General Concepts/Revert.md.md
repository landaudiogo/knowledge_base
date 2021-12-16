# Overview

Git revert is a command to undo changes. It differs from a reset or a checkout because it does not change where the pointer is in the commit history, the procedure in Layman terms is:

After indicating which commit we want to revert, git figures out a way to undo the changes of that commit specifically, as long as it is in the branch's history. 

Having undone the changes of that commit, a new commit is added to the branch.

# Use

This command is helpfeul when trying to undo the changes of a specific commit, or if a commit has been added to a remote repository and the history cannot be tampered with.

# Operations

If by any chance the user wants to undo a revert, it works in the same way, so the user would just have to indicate the commit whose changes we would like to undo (in this case, the revert commit), and the inverse operation will be performed over that commit onto a new commit in the branch.
