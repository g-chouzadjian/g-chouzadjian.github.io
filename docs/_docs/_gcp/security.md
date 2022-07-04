---
layout: post
title: Security
---

## What is Security?

>Information security, sometimes shortened to InfoSec, is the practice of protecting information by mitigating information risks. It is part of information risk management. It typically involves preventing or reducing the probability of unauthorized/inappropriate access to data, or the unlawful use, disclosure, disruption, deletion, corruption, modification, inspection, recording, or devaluation of information. It also involves actions intended to reduce the adverse impacts of such incidents. Protected information may take any form, e.g. electronic or physical, tangible (e.g. paperwork) or intangible (e.g. knowledge). Information security's primary focus is the balanced protection of the confidentiality, integrity, and availability of data (also known as the CIA triad) while maintaining a focus on efficient policy implementation, all without hampering organization productivity.

A key take-away is that we do not build systems with the overall purpose of security. Security is a cross-cutting concern and affects every part of the data-flow but it is not the **goal** of the system.

## What is "proper" data flow?

To ensure secure/proper data flow, InfoSec uses the CIA triad (Confidentiality, Integrity and Availability) to implement the below principles:

- You cannot view data you shouldn't.
- You cannot change data you shouldn't.
- You *can* access data you *should*.

Common terms: data-in-flight, data-at-rest.

## How do we control data flow?

We use the Triple A model:

- Authentication - who are you?
- Authorization - what are you allowed to do?
- Accounting - what did you do?

...put image or graphic here


## Key security mindset (Principles)

- Least privilege.
- Defense-in-depth.
- Fail securely.

IAM (Identity Access Management)
---

IAM serves AuthZ

### Resource Hierachy

A resource is something you create in GCP, a project is a container for a set of related resources. A folder contains any number of projcets and subfolders.

An organisation is tied to G-Suite or Cloud identity domain.

Trust boundary - things  within a project, trust each other.

### Permissions and Roles

#### Permissions

- A permission allows you to perform a certain action.
- Each one follows the form: **Service.Resource.Verb**
- Usually corresponds to REST API methods.

#### Roles

- A role is a collection of permissions to use or manage GCP resources.
- There are primitive roles which are at the project-level and are often too broad:
    - Viewer: Read-only.
    - Editor: can view and change things.
    - Owner: Can view/edit and also control access and billing.
- There are also predefined roles, which give granular access to specific GCP resources.
    - e.g. `roles/bigquery.dataEditor`, `roles/pubsub.subscriber`.
- There are also custom roles which are at the project or organisation level. You can define a granular set of permissions yourself.

