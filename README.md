# pr-preview-deploy-ux-action

GitHub Actions allows users to configure an environment for a job, but it's not yet possible to configure that environment as `transient`. This GitHub Action creates and updates transient Deployment objects using the GitHub API to provide a friendly user-experience for interacting with PR preview deployments that are conventionally destroyed when a PR is merged or closed.

## Features

- Each open PR contains a link to its active deployments in the timeline.
- No additional notifications or emails are generated.
- PRs and the [Deployments tab](https://github.com/urcomputeringpal/pr-preview-deploy-ux-action/deployments) show the history of all deployments. Following PR close, deployment links are no longer available. Deployments are reflected as 'Destroyed' in the UI.
- Environments are cleaned up on PR merge or close, hiding stale preview deployments from the Environment section in the Repository sidebar and the Settings tab.

## Requirements

- GitHub App with repo Administration permissions

## Example

### Active PR

<img width="858" alt="image" src="https://user-images.githubusercontent.com/47/232167347-5a30a8a7-5e15-47e7-9862-8f1ab5bdb01f.png">

### Merged PR

[Example](https://github.com/urcomputeringpal/pr-preview-deploy-ux-action/pull/20)

## Usage

### Create Preview Deploy when PRs are created or updated

```yaml
name: PR Preview
on:
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: urcomputeringpal/pr-preview-deploy-ux-action@v0
        name: Start preview deploy
        id: start
        with:
          step: start
          token: ${{ secrets.GITHUB_TOKEN }}
          head_ref: ${{ github.head_ref }}
          env: preview-${{ github.event.pull_request.number }}
          env_url: https://pr-preview-${{ github.event.pull_request.number }}.example.com
          
      - run: echo perform your preview deploy logic here

      - uses: urcomputeringpal/pr-preview-deploy-ux-action@v0
        name: Finish preview deploy
        if: always()
        with:
          step: finish
          status: ${{ job.status }}
          token: ${{ secrets.GITHUB_TOKEN }}
          head_ref: ${{ github.head_ref }}
          env: preview-${{ github.event.pull_request.number }}

          deployment_id: ${{ steps.start.outputs.deployment_id }}
          env_url: https://pr-preview-${{ github.event.pull_request.number }}.example.com
 ```
 
 ### Delete Preview environments when PRs are merged or closed

- [Create a new app](https://docs.github.com/en/developers/apps/creating-a-github-app) with the following permissions:
  - Repository Deployments: Read and write
  - Repository Administration: Read and write
- Create secrets for the App ID and the App Private key. 

 ```yaml
name: PR Preview Cleanup
on:
  pull_request:
    types: [ closed ]
  
jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - run: echo perform your cleanup logic here

      - name: Cleanup Preview deploy
        uses: urcomputeringpal/pr-preview-deploy-ux-action@v0
        with:
          step: cleanup
          env: preview-${{ github.event.pull_request.number }}
          app_id: ${{ secrets.APP_ID }}
          private_key: ${{ secrets.APP_PEM }}            
```

See [.github/workflows](./.github/workflows) for more examples.
