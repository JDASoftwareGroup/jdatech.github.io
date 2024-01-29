---
layout: single
title: "Ensuring Code Quality: A Guide to Dynamic GitHub Pull Request Gates"
tags: technology github devops
date: 2024-01-23 00:00:00 +0100
author: David Werner
author_profile: true
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
---

# Ensuring Code Quality: A Guide to Dynamic GitHub Pull Request Gates

In the dynamic world of software development, maintaining the integrity of your codebase is paramount. 
GitHub, a collaborative coding platform, provides a robust toolkit to uphold the quality and security of your projects.

GitHub's [branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) 
with mandatory [status checks](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging) 
stand as a bulwark, ensuring that contributions can only merge into the main branch after passing through quality and 
security checks.

But hey, you're still with us! 
Your interest in CI/CD, DevOps, and the intricate dance of collaboration in the software development world is palpable. 
Excellent!<br>
Now, let's dive into the realm of _dynamic_ status checks — a feature not directly supported by GitHub out of the box, 
but fear not, there are workarounds to achieve this goal.

## The Problem

In the branch protection rules of the main branch, configuring the required status checks for a
[pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) 
is crucial. 
Standard checks, such as linting, unit tests, and code coverage, should be typically required for all pull requests.

The following is a workflow example to check the source code changes of an application.
The tests are executed in job `test` after the application got built and deployed.
Therefore, job `test` should be configured as required status check in the branch protection rules.

```yaml
on:
  pull_request:
    paths:
      - src/**
      - tests/**

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      [...]
    
  test:
    needs: build-deploy    
    runs-on: ubuntu-latest
    steps:
      [...]
```

However, running this workflow is expensive and time-consuming because the application first needs to be built and 
deployed before the tests can be executed. 
And the tests themselves can also take quite some time.

Therefore, the test workflow should only be triggered if the source code or the tests themselves have changed.
In contrast, unrelated changes such as documentation changes should not trigger code related checks.<br>
This is why the `pull_request` workflow trigger is limited to changes in directories `src` and `tests`, as specified by
`paths`.

But what if the test workflow is not triggered because no files in those directories have changed?
Then, the pull request could not be merged because the required status check is not passing, as it was not executed at 
all.<br>
This is where the problem lies and why we need dynamic status checks that are only required if the relevant files have 
changed.

Fortunately, there are two workarounds!

## Check for relevant Changes

The workaround is to check for the relevant file changes in a job of the workflow itself and execute the actual status 
check only if the relevant files have changed.<br>
The `paths` of trigger `pull_request` are specified in the `files` input of the 
[tj-actions/changed-files](https://github.com/marketplace/actions/changed-files) action (or any similar GitHub Action). 

This status check doesn't block the PR from merging if it gets skipped.
It only blocks it if it gets executed and fails.

```yaml
on:
  # Always execute, otherwise GitHub will wait forever for required checks
  pull_request:
    
jobs:
  check-if-relevant:
    runs-on: ubuntu-latest
    outputs:
      is-relevant: ${{ '{{' }} steps.relevant-files-changed.outputs.any_changed }}
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
            src/**
            tests/**
 
  build-deploy:
    needs: check-if-relevant
    if: needs.check-if-relevant.outputs.is-relevant == 'true'
    runs-on: ubuntu-latest
    steps:
      [...]
    
  test:
    needs: build-deploy
    runs-on: ubuntu-latest
    steps:
      [...]
```

### Reusable Workflow as Status Check

The approach works fine for standard workflows, where any job can be configured as status check.
However, it doesn't work for [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows).

In this example the test workflow utilizes reusable workflows for the build and deploy and the test steps.

```yaml
on:
  pull_request:
    
jobs:
  check-if-relevant:
    runs-on: ubuntu-latest
    outputs:
      is-relevant: ${{ '{{' }} steps.relevant-files-changed.outputs.any_changed }}
    steps:
     [...]
 
  build-deploy:
    needs: check-if-relevant
    if: needs.check-if-relevant.outputs.is-relevant == 'true'
    uses: ./.github/workflows/release.build_deploy.yml
    secrets: inherit    

  test:
    needs: build-deploy
    uses: ./.github/workflows/test.execution.yml
    secrets: inherit
    with:
      app-url: ${{ '{{' }} needs.build-deploy.outputs.app-url }}
```

The calling job `test` can't be set as status check because GitHub ignores its status.

Configuring a job in the called reusable workflow `test.execution.yml` is not an option either.<br>
If `test` in the calling workflow would be skipped, the pull request would wait forever for the status check in the 
reusable workflow as the skipping happens on workflow parent level, not child level.

The workaround is to configure an additional, standard job `test-completed` that concludes the workflow as status 
check and fails if the `test` failed before.<br>
Setting `if: always()` ensures that the job is always executed, even if the `test` job failed.

```yaml
on:
  pull_request:
    
jobs:
  check-if-relevant:
    runs-on: ubuntu-latest
    outputs:
      is-relevant: ${{ '{{' }} steps.relevant-files-changed.outputs.any_changed }}
    steps:
     [...]
 
  build-deploy:
    needs: check-if-relevant
    if: needs.check-if-relevant.outputs.is-relevant == 'true'
    uses: ./.github/workflows/release.build_deploy.yml
    secrets: inherit  

  test:
    needs: build-deploy
    uses: ./.github/workflows/test.execution.yml
    secrets: inherit
    with:
      app-url: ${{ '{{' }} needs.build-deploy.outputs.app-url }}

  test-completed:
    needs: test
    runs-on: 'ubuntu-latest'
    if: always()
    steps:
      - name: Check test job status
        if: needs.test.result == 'failure'
        run: exit 1
```

## Conclude with final Status Check

Another workaround is to add a concluding job like the `required-status-check` in the following example to all 
dynamically required workflows.

```yaml
on:
  pull_request:
    paths:
      - src/**
      - tests/**

jobs:  
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      [...]

  test:
    needs: build-deploy
    runs-on: ubuntu-latest
    steps:
      [...]
    
  required-status-check:
    needs: test
    runs-on: 'ubuntu-latest'
    if: always()
    steps:
      - name: Check workflow status
        if: needs.test.result != 'success'
        run: exit 1
```

This job would return a failure state if the tests of job `test` failed.

In the repo settings configure `required-status-check` as required status check in the branch protection
rules of the main branch.<br>
The pull requests get blocked if none of the triggered workflows have implemented the `required-status-check` job or at
least one of them is failing.<br>
It gets unblocked if at least one `required-status-check` job is implemented and all of them are successful. 
It's worth noting that at least one workflow needs to run in any case (e.g. a lint or source formatting check) and can be piggybacked to implement
this.

## Comparison

### Pro relevant Changes Check

*   This approach is less complex as the final status check workflow because its fallback workflow has to be kept in mind.
*   It's transparent for a GitHub repo admin what status checks are actually executed by just taking a look at the
    branch protection rules. Whereas it is not obvious with a general status check such as `required-status-check`.

### Pro final Status Check

*   Only those workflows get executed that actually apply to the changed files, because the list of relevant files are 
    declared at the `pull_request` trigger.<br> 
    As a result, the status check overview of the pull request is limited to the relevant checks and no workflow is run
    unnecessarily.
*   The triggers and specific trigger conditions can be configured declaratively.<br>
    Additional triggers besides the `pull_request` that have to run in any case (even without relevant file changes) can
    be configured in a clean way. And the files restriction of the `pull_request` trigger are also declared where you 
    would expect them: in the `pull_request` trigger.
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
        is-relevant: ${{ '{{' }} (github.event_name == 'pull_request' && steps.relevant-files-changed.outputs.any_changed == 'true') || github.event_name != 'pull_request' }}
        steps:
          # https://github.com/marketplace/actions/changed-files
          - uses: tj-actions/changed-files@v41.0.1
            if: github.event_name == 'pull_request'
            id: relevant-files-changed
            with:
              files: package-lock.json
    ```

## Conclusion

GitHub's branch protection rules with required status checks are a powerful tool to ensure quality and security.
They are only missing the ability to configure dynamic status checks that are only required if relevant changed had been
committed.

However, we have two workarounds to solve this problem:
1. Check for the relevant file changes in a job of the workflow itself and execute the actual status check only for 
   relevant changes.
2. Add a concluding job to all dynamically required workflows and configure it as required status check.

With GitHub continuously evolving and gathering user feedback, our hope is that the need for such workarounds becomes 
obsolete.<br> 
We look forward to a future where seamless and dynamic status checks align effortlessly with the needs of developers,
making these inventive workarounds a thing of the past.
