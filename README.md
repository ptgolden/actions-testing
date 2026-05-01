This repo is meant for developing the normalization and QC workflow for <https://github.com/monarch-initiative/mondo>. The basic issues is that we require a QC step to merge pull requests, but we want `mondo-edit.obo` to be normalized before running that QC, and we want that normalization to happen automatically without a user having to run `make NORM && mv NORM mondo-edit.obo` manually.

The rub of it is that commits from an action [do not trigger actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow?utm_source=chatgpt.com#triggering-a-workflow-from-a-workflow) by default.

All of solutions I've worked on have tried to get around that fact by being clever in certain ways, but it is unavoidable in this case: **if we want to automatically commit changes to mondo-edit.obo and require branch protection contingent on running QC, we have to set up GitHub App tokens**. There is no way around it.

---

This repository implements the workflow we need with a stupidly minimal example, but one that's compatible with mondo itself:

- `make test` runs QC on `mondo-edit.obo` (it checks if the word "ERROR" is present)
- it checks if `mondo-edit.obo` is normalized and auto-commits any fixes in the same way as in mondo (the `NORM` target). (normalization means the file matches the output of `sort`)
