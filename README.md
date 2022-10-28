# circleci-github-actions <!-- omit in TOC -->

A [CircleCI orb](https://circleci.com/orbs/) for interacting with [GitHub Actions](https://github.com/features/actions).

Check out [this orb](https://circleci.com/developer/orbs/orb/movermeyer/github-actions) in the CircleCI Orb Registry.

- [Quick Start](#quick-start)
- [Authentication](#authentication)
- [Commands](#commands)
  - [repository_dispatch](#repository_dispatch)
    - [Parameters](#parameters)
    - [Webhook event `client_payload`](#webhook-event-client_payload)
      - [Example](#example)

## Quick Start

Include the orb in the root of your CircleCI config:

```yaml
orbs:
  github-actions: movermeyer/github-actions@0.0.5 # Make sure to update this to the latest version: https://circleci.com/developer/orbs/orb/movermeyer/github-actions
```

Then call commands in the `steps` section of your CircleCI config:

```yaml
steps:
  - github-actions/repository_dispatch:
      repo_name: "movermeyer/circleci-github-actions" # Your GitHub organization name + repo name
      event_type: "build_complete" # Arbitrary string that your GitHub Actions will filter on
```

## Authentication

Commands need to authenticate with GitHub in order to work.

1. Create a [new GitHub Personal Access Token](https://github.com/settings/tokens/new) with the `repo` scope enabled.
  * Name it something meaningful. Perhaps `circleci-github-actions orb access token` or similar?
  * Make sure to copy down the access token it generates.
2. Create a new CircleCI environment variable, either on the CircleCI project, or within a CircleCI scope used by the CircleCI project.
  * Name it `GITHUB_PERSONAL_ACCESS_TOKEN` and copy the access token that GitHub generated in the last step.
    * If for some reason your project/scope already has a `GITHUB_PERSONAL_ACCESS_TOKEN`, use a different name instead. You will use that new name to override the `github_personal_access_token` parameter when calling commands.

## Commands

There is only a single command implemented so far.

### repository_dispatch

This sends a [`repository_dispatch`](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows#repository_dispatch) event webhook to GitHub Actions.

#### Parameters

| Parameter                    | Description                                                                                                                                                                                            | Required | Default                        | Example               |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ------------------------------ | --------------------- |
| repo_name                    | The organization and repo name to trigger the `repository_dispatch` event for                                                                                                                          | Yes      | N/A                            | `octocat/hello-world` |
| event_type                   | The ['event_type'](https://docs.github.com/en/free-pro-team@latest/rest/reference/repos#create-a-repository-dispatch-event) parameter to send. Arbitry string that GitHub Actions can use to filter on | Yes      | N/A                            | `octocat/hello-world` |
| github_personal_access_token | The name of the environment variable containing the GitHub Personal Access Token. See [Authentication](#authentication)                                                                                | No       | `GITHUB_PERSONAL_ACCESS_TOKEN` |                       |
| metadata            | Add any metadata that you should provide to GitHub Action                                                                                                                                                       | No       | N/A                            | `{ "field_1": "value" }` |

#### Webhook event `client_payload`

By default, the [`client_payload`]((https://docs.github.com/en/free-pro-team@latest/rest/reference/repos#create-a-repository-dispatch-event)) sent to GitHub contains most useful information about the build.

These are simply copied from the various CircleCI environment variables.

| `client_payload` | CircleCI environment variable |
| ---------------- | ----------------------------- |
| `build_num`      | `CIRCLE_BUILD_NUM`            |
| `branch`         | `CIRCLE_BRANCH`               |
| `username`       | `CIRCLE_USERNAME`             |
| `job`            | `CIRCLE_JOB`                  |
| `build_url`      | `CIRCLE_BUILD_URL`            |
| `vcs_revision`   | `CIRCLE_SHA1`                 |
| `reponame`       | `CIRCLE_PROJECT_REPONAME`     |
| `workflow_id`    | `CIRCLE_WORKFLOW_ID`          |
| `pull_request`   | `CI_PULL_REQUEST`             |

##### Example

```json
{
  "event_type":"wheel_build_complete",
  "client_payload": {
    "build_num": "35",
    "branch": "refs/tags/v0.0.14",
    "username": "movermeyer",
    "job": "notify_github",
    "build_url": "https://circleci.com/gh/movermeyer/octokit_test/35",
    "vcs_revision": "46f3ff30f669ec61194f6010d4a8adf98a71b29a",
    "reponame": "octokit_test",
    "workflow_id": "c1b6618e-a6ea-4ce7-a907-74763b5bdd31",
  },
  "metadata": { "field_1": "value" }
}
```
