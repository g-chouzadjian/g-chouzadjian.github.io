---
title:  "Inter-repo Workflow Triggers"
search: true
categories: 
  - GitHub Actions
last_modified_at: 2022-10-18T08:06:00-05:00
---

For inter-repo workflow orchestration you can leverage the `repository_dispatch` event from the GitHub API.

The triggering workflow needs to run the following API call to trigger the event:

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"on-demand-test","client_payload":{"unit":false,"integration":true}}'
```

**Note:** `event_type` is a custom webhook event name can can be anything you want
{: .notice--info}


Or you can use an [open-source action](https://github.com/marketplace/actions/repository-dispatch) that wraps the above call:

```yaml
...
- name: trigger
    uses: peter-evans/repository-dispatch@v2
    with:
      token: ${{ secrets.REPO_ACCESS_TOKEN }}
      repository: anzx-platform-security-poc/reusable-workflows
      event-type: trigger-event
      client-payload: '{"ref": "${{ github.ref }}", "sha": "${{ github.sha }}"}'
```
