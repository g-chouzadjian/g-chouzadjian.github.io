---
title:  "Inter-repo Workflow Triggers"
search: true
categories: 
  - GitHub Actions
last_modified_at: 2022-10-18T08:06:00-05:00
---

Sometimes you need a workflow in one repository to trigger a run of a workflow in another repository. A common scenario for this is if your application has a dependency hosted in another repository. If you update your application's upstream, it'd be nice for that change to trigger a rebuild of your application.

This can be done in GitHub Actions, if in a slightly convoluted way...

To achieve this we're going to leverage the `repository_dispatch` event offered by the GitHub API.

This event can be used to [trigger workflows](https://docs.github.com/en/github-ae@latest/actions/using-workflows/events-that-trigger-workflows#repository_dispatch).

### 1. Create the triggering workflow

The triggering workflow needs to create a `repository_dispatch` event, which we can do via the GitHub API.

In your triggering workflow, run:

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"<your-custom-event>","client_payload":{"unit":false,"integration":true}}'
```

**Note:** `event_type` is a custom webhook event name and can be anything you like.
{: .notice--info}


Or you can use an [open-source action](https://github.com/marketplace/actions/repository-dispatch) that wraps the call in a nicer interface like so...

```yaml
...
- name: trigger
    uses: peter-evans/repository-dispatch@v2
    with:
      token: ${{ secrets.REPO_ACCESS_TOKEN }}
      repository: OWNER/REPO
      event-type: <your-custom-event>
      client-payload: '{"ref": "${{ github.ref }}", "sha": "${{ github.sha }}"}'
```

**Note:** the documentation for the above action states the token should be a PAT with `repo` scope, however you can [authenticate as an installation](https://docs.github.com/en/developers/apps/building-github-apps/authenticating-with-github-apps), as long as your app has the correct permissions (`content: read/write, metadata: read`)
{: .notice--info}