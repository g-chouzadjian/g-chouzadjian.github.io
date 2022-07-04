---
layout: post
title: Identity and Access Management (IAM)
---

### Tips

- To get role bindings for a particular service-account:

```
gcloud projects get-iam-policy <YOUR GCLOUD PROJECT>  \
--flatten="bindings[].members" \
--format="table(bindings.role)" \
--filter="bindings.members:<YOUR SERVICE ACCOUNT>"
```