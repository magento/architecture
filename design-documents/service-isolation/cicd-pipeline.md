# CI/CD tools

## About

This proposal describes desired state of CI/CD tools and focuses on developer experience of CI tools.

### Terminology
Preview environment - environment used by the team for internal testing and experiments, contains code for a feature that being developed.
Stage environment - environment used by the team to demo changes to product owner.

### Desired state of tools
1. Interaction with CI/CD tools should be aligned GitOps approach (see [this](https://www.youtube.com/watch?v=BF3MhFjvBTU) video)
1. CI tools should allow update preview and stage environments
1. CI tools should allow PR delivery to mainline (auto merging of PRs)
1. Adding new types of tests, deployment configuration, etc should not require devops team and be developer activity to not have team blockage
1. CI tools should support Magento application that consists from multiple services (for GraphQl and service isolation)
1. CD tools should be available to deploy Magento application that consists from multiple services (for GraphQl and service isolation)
1. CI/CD tools that used internally to test Magento available for extension developers and system integrators
1. CD tools should support rollback (and other deployment best practices like blue/greed deployment)
1. Publication also triggered from Git

### Use cases (current and future)
1. Run builds on code change
1. Update preview and stage environment used by the team for testing
1. Automatically merge pull requests after approval if builds are passed
1. Run builds for multiple repo combinations (open source + bundled extensions, commerce + bundled extensions, commerce only, etc)
1. Specify what type of tests to run, all or only unit tests for instance
1. Specify branches in each repository to use for a build 
1. Send notifications about tests failures
1. Collect and provide access to logs
1. Allow to run only specific tests
1. Have a smoke suite of tests that will run first and cache big issues with code
1. Deploy other services needed to run tests on the application when change is made to one service

## Design

### Solution
Pipelines should be configured by code and/or conventions. Triggered by interaction with Git. There are should be minimal/or no interaction with UI (with the exception of PR approvals) if the pipeline is successful to update environments, create builds, merge PR, etc.

The proposed solution works best when developing separate services, when each service lives in it's own repo. But can also work for testing monolith application, application that consists from separate UIs and arbitrary combination of extensions.

Branches named by convention. Branches that start with feature contain clean feature related changes. Pushing changes to feature branches triggers CI pipeline below. User receive notification via slack/email with links to builds.

Pipeline that creates PRs.

On commit to a feature branch, PR automatically being submitted to mainline in a draft state. Every time new PR being created, this pipeline also looks by convention for related PRs and link them automatically. There are should be also option to link PRs manually.

![New CI pipeline - creare PR](img/new-ci-pipeline-create-pr.png)

Pipeline that runs tests on PRs.

When `run-tests` label is added to one of the linked PRs it would trigger the following pipeline. If PRs are in active state, pipeline would also refresh stage environment.

![New CI pipeline](img/new-ci-pipeline.png)

Pipeline that merges PRs.

This pipeline will be executed after PRs approved. If branches contain latest mainline and tests run on latest commit, PRs will be merged immediately. Otherwise it will rerun tests and merge PRs if tests turn out green.

![New CI pipeline - merging PR](img/new-ci-pipeline-merge-pr.png)

Each team may have it's own Kubernetes cluster that can be used to build images, run tests, etc.

Alternative approaches for triggering pipeline that runs tests
1. Add flag in the commit message
2. Trigger builds on each commit with a delay and restart when changes pushed to a different repo
3. Allow to specify flag to not ran the pipeline and do option #2

#### Storing pipeline configuration
Pipeline configuration can be stored in the separate repo. Pipeline build specific configuration can be stored in a separate repo or in the comment section on of the the PRs. Seems like storing pipeline configuration in the separate repo would be more clear.

Here is example of configuration.

```
{
    "branches": {}
    "run_tests": {
        "unit": {
            "skip": {}
        },
        "integration": {
            "skip": {}
        }
        "functional": {
            "skip": {}
            "retries": 2
        }
    }
}
```

#### Tools
Existing tooling doesn't have support support of multiple repositories. We may need to create our own tooling. This will be investigated in separate proposals.

## Open questions
* How to deliver security fixes?
