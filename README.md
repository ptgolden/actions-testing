This repo is meant for developing the normalization and QC workflow for <https://github.com/monarch-initiative/mondo>. The basic issues is that we require a QC step to merge pull requests, but we want `mondo-edit.obo` to be normalized before running that QC, and we want that normalization to happen automatically without a user having to run `make NORM && mv NORM mondo-edit.obo` manually.

The rub of it is that commits from an action [do not trigger actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow?utm_source=chatgpt.com#triggering-a-workflow-from-a-workflow) by default.

All of solutions I've worked on have tried to get around that fact by being clever in certain ways, but it is unavoidable in this case: **if we want to automatically commit changes to mondo-edit.obo and require branch protection contingent on running QC, we have to set up GitHub App tokens**. There is no way around it.

---

This repository implements the workflow we need with a stupidly minimal example, but one that's compatible with mondo itself:

- `make test` runs QC on `mondo-edit.obo` (it checks if the word "ERROR" is present)
- it checks if `mondo-edit.obo` is normalized and auto-commits any fixes in the same way as in mondo (the `NORM` target). (normalization means the file matches the output of `sort`)

What's different here is that authentication happens with a GitHub App token rather than `GITHUB_TOKEN`. Because of that, automatic normalization commits *do* trigger workflows to run again. This simplifies things and lets us have this:

- A PR orchestrator is the top level PR check. It calls two different workflows
    1.`normalize-mondo.edit.yaml`, which checks for normalization and auto-commits a normalized `mondo-edit.obo` if not
    2. `qc.yaml`, which runs `make test`. This step is skipped if the previous step resulted in a push
- After calling those, a `merge_gate` step checks that both of these workflows were successful. (This is necessary because if `qc.yaml` is skipped, there are no failing checks, and it looks in the GitHub UI like "everything is green" for the commit before normalization, even if QC was not run).

Branch protection points to the `merge_gate` step.

---

This requires [registering a GitHub App](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app), which I imagine we should do in the `monarch-initiative` organization. The app I set up here has exactly one permission, which allows it to make commits:
<img width="744" height="90" alt="image" src="https://github.com/user-attachments/assets/48a7e6ee-da2e-4ed5-be8a-9ddf3bc17ef1" />

After installing the app, [the `create-github-app-token` workflow generates app tokens](https://github.com/actions/create-github-app-token#usage) to be used in the `normalize-mondo-edit.yaml` workflow to authenticate pushed commits.
