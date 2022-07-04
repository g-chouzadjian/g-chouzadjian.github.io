---
layout: post
title: Key Management Service (KMS)
---

> Cloud KMS let's you manage assymetric/symmetric cryptographic keys.

These key's can be used to encrypt your data-at-rest and/or in data-in-transit.

[infra-as-code reference](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/kms_crypto_key)

## Object Hierachy

### Key rings

These are used to logically group a set of keys which can be useful for IAM purposes.

```hcl
resource "google_kms_key_ring" "example-keyring" {
  name     = "keyring-example"
  location = "global"
}
```

A key-ring's name does not need to be unique to a project, but must be unique across a location.

```bash
gcloud kms keyrings create "test" \
    --location "global"
```

### Keys

```hcl
resource "google_kms_crypto_key" "example-key" {
  name            = "crypto-key-example"
  key_ring        = google_kms_key_ring.keyring.id
  rotation_period = "100000s"

  lifecycle {
    prevent_destroy = true
  }
}
```

```bash
echo -n '<plain-text>' | gcloud kms encrypt \
--plaintext-file=- \
--ciphertext-file=- \
--location=global \
--keyring=<keyring-name> \
--key=<key-name> \
--project=<project-name> | base64
```

When `--plaintext-file=-`, `kms encrypt` will read from stdin. Similarly when `--ciphertext-file=-`, `kms decrypt` will write to stdout.

*Note: To help with data transmission/storage of the ciphertext, we can choose to encode stdout using the `base64` program*