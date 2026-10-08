# How To Find GDB Source Code

1. The GDB source code comes from a project call binutils-gdb, which
   is a combination of the binutils project and the GDB project in a
   single repository.

2. Usually the Claude root directory is not the GDB source tree, but
   you should first check for a `.git` entry in the Claude root
   directory to see if it is.

3. The `.git` entry can be a directory (for actual clones) or a file
   (for git worktrees).  Either should be acceptable.  If using a
   shell test to check for the `.git` entry, use `test -e`, never use
   `test -d` as that will not work for git worktrees.

4. If the Claude root directory is not a git repository, then the GDB
   source code will usually be in a sub-directory, try `src/` then
   `binutils-gdb/` first in that order.

5. If neither of the previous options are present, look in every other
   sub-directory looking for a suitable git checkout.  But only search
   one level deep, that is, sub-directories of the Claude root
   directory, but no deeper.

6. If the GDB source code still cannot be found, then ask the user
   where the source code is.
