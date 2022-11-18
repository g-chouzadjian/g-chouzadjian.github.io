---
title:  "Complex job orchestration"
search: true
categories: 
  - GitHub Actions
last_modified_at: 2022-11-18T08:06:00-05:00
---

Job orchestration in GHA can suck and requires some creative hacks for complex scenarios.

## Scenario 1: Job and File Change Dependecy 

For this scenario, we want one job to always run on a push to main. We then want a second job to run only when that previous job has finished successfully **and** also when only certain files have changed. There is no mechanism to perform both of these simulatenously.

On their own these are simple scenarios, but together, they cause headaches.

To cover the first dependency we can use the `jobs.<job_id>.needs` syntax:
> Use `jobs.<job_id>.needs` to identify any jobs that must complete successfully before this job will run.

To cover the files changed dependency, we're going to use a combination of git diff and conditional job execution, see the solution below:


```yaml
name: blog
on:
  push:
    branches:
      - main
jobs:
  depended:
    runs-on: ubuntu-latest
    ...

  get-changed-files:
    runs-on: ubuntu-latest
    # use the github context to get the HEAD sha and assign to an 
    # env var at the job level so it can be used in subsequent steps.
    env:
      after: ${{ github.sha }}

    # use job outputs to use variables between jobs.
    outputs:
      changed_files: ${{ steps.get_files.outputs.changed_files }}
    steps:
      # fetch the repository and it's history using the fetch-depth attribute.
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      # to get the SHA of the previous commit, we can use the file associated with
      # the triggering push event (using github context). The full webhook payload
      # is present in this file.
      - name: Get before SHA
        # to export an env var from within the command-line, use the GITHUB_ENV  
        run: |
          echo before=$(jq -r '.before' ${{ github.event_path }}) >> $GITHUB_ENV

      # use `git diff` to get the name of the changed files using the exported SHAs
      # from the previous steps.
      - name: Get the changed files
        id: get_files
        run: |
          echo changed_files=$(git diff --name-only $before...$after) >> $GITHUB_OUTPUT  
    
  dependee:
    needs: [depended, get-changed-files]
    # use the in-built contains() function to check the change_files array for the 
    # desired file.
    if: contains(needs.get-changes-files.outputs.changed_files, <desired-file/s>)
    ...
```

While it's a bit convoluted, the above workflow displays how you can cover multiple depdencies for job execution using a combination of inbuilt workflow features and bash/git hacks. The following GHA features were used in the above solution:
- [github context](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context)
- [environment variables](https://docs.github.com/en/actions/learn-github-actions/environment-variables#about-environment-variables)
- [job outputs](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idoutputs)
- [checkout action](https://github.com/actions/checkout)
- [push webhook payload](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#push)
- [GITHUB_ENV](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-environment-variable)
- [git diff](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---name-only)
- [if conditional](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif)