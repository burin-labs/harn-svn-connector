# AGENTS.md

This repository implements `harn-svn-connector`, a pure-Harn connector package for Subversion.

Run targeted checks with a local Harn CLI from the main Harn repository:

```sh
harn check src/lib.harn
harn fmt --check src/lib.harn tests/*.harn
for test in tests/*.harn; do harn run "$test" || exit 1; done
```
