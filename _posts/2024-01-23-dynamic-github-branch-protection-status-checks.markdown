---
layout: single
title: "Dynamic GitHub Branch Protection Status Checks"
tags: technology github devops
date: 2024-01-23 00:00:00 +0100
author: David Werner  # as used in `authors.yml`
author_profile: true
header:
  overlay_image: assets/images/tech_gear_banner.jpg  # can also be a different asset
  overlay_filter: 0.2
  show_overlay_excerpt: false
---

# Dynamic GitHub Branch Protection Status Checks

In the branch protection rules of branch `main`, we want to configure the required status checks that need to be passed
before a pull request can be merged to `main`.<br>
However, GitHub only allows checks to be configured in a static way. They are enabled globally for all pull request
changes.
This is problematic for checks that must only be executed for specific changes.

Let's take the E2E test workflow for instance. It should be a required check for all code changes.<br>
But it wouldn't be triggered if the pull request only updates the documentation because it is quite costly.
The `pull_request` trigger is limited to changes related to the E2E tests and the actual source code.

**Example**

```yaml
name: E2E Testing

on:
  pull_request:
    paths:
      - .github/workflows/e2e_*.yml
      - e2e/**
      - index.html
      - manifest.json
      - package-lock.json
      - public/**
      - src/**
```

Now, the pull request could not be merged if the E2E test workflow is set as required status check.

The workaround is to define and check for the relevant file changes in a job of the workflow itself and execute the
actual status check only if the relevant files have changed.
This status check doesn't block the PR from merging if it gets skipped.

**Example**

```yaml
on:
  # Always execute, otherwise GitHub will wait forever for required checks
  pull_request:
    
jobs:
  check-if-relevant:
    runs-on: ubuntu-latest
    outputs:
      is-relevant: ${{ steps.relevant-files-changed.outputs.any_changed }}
    steps:
      # This step is needed by tj-actions/changed-files
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      # https://github.com/marketplace/actions/changed-files
      - uses: tj-actions/changed-files@v41.0.1
        id: relevant-files-changed
        with:
          files: |
            .github/workflows/e2e_*.yml
            e2e/**
            index.html
            manifest.json
            package-lock.json
            public/**
            src/**
 
  e2e-test:
    needs: check-if-relevant
    if: needs.check-if-relevant.outputs.is-relevant == 'true'
    runs-on: ubuntu-latest
    steps:
      [...]
```

## Reusable Workflow as Status Check

The approach works fine for standard workflows, where we can configure any of the jobs as status check.
However, it doesn't work for reusable workflows.

**Example**

```yaml
name: E2E Testing

on:
  pull_request:
    
jobs:
  check-if-relevant:
    runs-on: ubuntu-latest
    outputs:
      is-relevant: ${{ steps.relevant-files-changed.outputs.any_changed }}
    steps:
     [...]
 
  build-deploy:
    needs: check-if-relevant
    if: needs.check-if-relevant.outputs.is-relevant == 'true'
    uses: ./.github/workflows/release.build_deploy.yml
    secrets: inherit    

  e2e-test:
    needs: ephemeral-portal-registration
    uses: ./.github/workflows/e2e_test.execution.yml
    secrets: inherit
    with:
      app-url: ${{ needs.build-deploy.outputs.app-url }}
```

The calling job `e2e-test` can't be set as status check because GitHub ignores its status even when it has run.<br>
As shown in the below screenshot, GitHub doesn't realize that `e2e-test` ran in workflow `E2E Testing` although it shows
that the reusable workflow called by `e2e-test` ran successfully (s. `E2E Testing/e2e-test/e2e-test-execution`).

![ci-cd.status-check-reusable-workflow-caller.png](..%2Fassets%2Fimages%2F2024-01-23-dynamic-github-branch-protection-status-checks%2Fci-cd.status-check-reusable-workflow-caller.png)

(The status checks get configured by job name. Specifying the workflow is not possible. That's why only `e2e-test` is
shown as expected status name in the screenshot.
Once the job had ran, GitHub updates the check overview and prepends the workflow name.)

Configuring job `e2e-test-execution` in the called reusable workflow is not an option either.<br>
If `e2e-test` in the calling workflow would be skipped, the pull request would wait forever for `e2e-test-execution` as
the skipping happens on the workflow parent level, not the child level.

The workaround is to configure an additional, standard job that concludes the workflow as status check and fails if the
`e2e-test` failed before.

**Example**

```yaml
  e2e-test:
    needs: ephemeral-portal-registration
    uses: ./.github/workflows/e2e_test.execution.yml
    secrets: inherit
    with:
      mfe-namespace: ${{ needs.ephemeral-portal-registration.outputs.mfe-namespace }}

  e2e-test-completed:
    needs: e2e-test
    runs-on: 'ubuntu-latest'
    if: always()
    steps:
      - name: Check e2e-test job status
        if: ${{ needs.e2e-test.result == 'failure' }}
        run: exit 1

```

So, in the end our status check setting looks like this:

![ci-cd.status-check-setting.png](..%2Fassets%2Fimages%2F2024-01-23-dynamic-github-branch-protection-status-checks%2Fci-cd.status-check-setting.png)

## Alternative Approach

Another workaround is to add job `required-status-check` to all workflows that possibly need to be passed.

**Example**

```yaml
on:
  pull_request:
    paths:
      - .github/workflows/e2e_*.yml
      - e2e/**
      - index.html
      - manifest.json
      - package-lock.json
      - public/**
      - src/**

jobs:
  e2e-test:
    runs-on: ubuntu-latest
    steps:
      [...]
    
  required-status-check:
    needs: e2e-test
    runs-on: 'ubuntu-latest'
    if: always()
    steps:
      - name: Check workflow status
        if: ${{ needs.e2e-test.result != 'success' }}
        run: exit 1
```

This job would return a failure state if the E2E tests of job `e2e-testing` have failed.

In the repo settings configure `required-status-check` as required status check in the branch protection
rules of the main branch.<br>
The pull requests get blocked if none of the triggered workflows have implemented the `required-status-check` job or at
least one of them is failing.<br>
It gets unblocked if at least one `required-status-check` job is implemented and all of them are successful.

For our repo, the lint workflow would run in any case.
So, there would always be a `required-status-check` job triggered to pass the protection rule if it would be implemented
in this workflow as well.

## Advantages

*   Only the workflows get executed that are actually required, because the list of relevant files are declared at the
    `pull_request` trigger.
    So, the status check overview of the pull request is limited to the relevant checks and no workflow is run unnecessarily.
*   The triggers and specific trigger conditions can be configured declaratively.<br>
    Additional triggers besides the `pull_request` might have to run without relevant file changes and can be configured
    in a clean way. And the files restriction of the `pull_request` trigger are also declared where you would expect
    them in the workflow header and are not the result of a job run.
    <br>
    <br>
    Declarative config:

    ```yaml
    on:
      workflow_dispatch:
      schedule:
        - cron: '0 0 * * 1' # 00:00 on Mondays.
      pull_request:
        paths:
          - package-lock.json
    ```

    Job config:

    ```yaml
    on:
      workflow_dispatch:
      schedule:
        - cron: '0 0 * * 1' # 00:00 on Mondays.
      pull_request:

    jobs:    
      check-if-relevant:
        runs-on: ubuntu-latest
        outputs:
        is-relevant: ${{ (github.event_name == 'pull_request' && steps.relevant-files-changed.outputs.any_changed == 'true') || github.event_name != 'pull_request' }}
        steps:
          # https://github.com/marketplace/actions/changed-files
          - uses: tj-actions/changed-files@v41.0.1
            if: github.event_name == 'pull_request'
            id: relevant-files-changed
            with:
              files: package-lock.json
    ```

## Disadvantages

*   Though this approach would reduce the number of workflow runs, it does run the additional `required-status-check`
    job whenever the dynamically required workflow needs to be executed.
    And everytime for the workflow that acts as a fallback.
*   This approach is more complex with the fallback workflow.
*   It is not transparent for a GitHub repo admin what status checks are actually executed by just taking a look at the
    branch protection rules, as `required-status-check` or a similar name will be quite generic.
