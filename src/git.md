# Git

## Clone with non-default SSH key

Maybe you need a specific SSH key to clone from GitHub, and maybe you don’t
want to add it as default. One can specify which SSH command Git will use
to do git operations.

Given my ssh key at `/Users/sirodoht/.ssh/id_ed25519_debug`, here is an
example:

```
GIT_SSH_COMMAND='ssh -i /Users/sirodoht/.ssh/id_ed25519_debug -o IdentitiesOnly=yes' git clone git@github.com:debug-org/api.git
```
