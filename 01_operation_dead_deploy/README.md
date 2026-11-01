# Investigating Unauthorized Provisioning in an Azure Subscription

## Scenario

A coworker mentioned that an intern had deployed resources, so I was assigned to perform a policy compliance review. I noticed a resource that was created in a way that didn't follow naming policy. I investigated with read-only access, so I could prevent destruction of evidence. My goal was to determine what type of resource was created, was it created within policy, if not who created it out of policy, when they created it, and where it was deployed, along with whether or not it was tagged correctly. Most importantly, I wanted to know why our naming policy did not stop the violation.

## Environment

This investigation took place in a live, multi user Azure training tenant with Reader access so I could examine resources without altering evidence.
Tools used:
* Resource groups list
* Storage accounts Overview, tags blade, JSON View
* Resource Group Deployments blade and deployment detail.
* Resource group Policies blade (Compliance and Assignments)

## Investigation

### Stage 1: Locating the anomaly

While reviewing the subscription's resource groups, I found one entry that
didn't match the `rg-<workload>-<environment>-<region>-<instance>` naming
convention. Instead, it was named as a single word followed by a number
(`<word><number>`). It had no rg prefix, no environment indicator to show
whether it was production, development, or test, and no region. Its location
was also East US, while every other resource group was in Central US.

This suggested the resource group was created outside the normal
provisioning process, possibly by an inexperienced employee, a compromised
account, or a bad actor, so it was worth investigating further.

### Stage 2: Inspecting the resource

When I opened the anomalous resource group, I found a single storage account.
It was tagged with an owner (naming an intern), a cost center tag with the
value "unspecified," and an environment tag with the value "unknown."

This is important because the cost center and environment tags existed but
didn't specify where costs should be allocated, leaving orphaned cost. It's
worse than the tags being missing entirely, because a policy that only checks
whether those tags exist would pass this resource. The tags have values; the
values just don't carry any information. Not specifying the environment also
makes it difficult to know whether taking any action against this resource
will affect production (a big problem) or development and testing.

Next, I reviewed the storage account's configuration in the JSON View. The
minimum TLS version was set to 1.0, which still needs to be raised to at least
1.2, the modern minimum. The network rules had no virtual network or IP
restrictions and a default action of Allow, so the account could be reached
from any network on the internet. However, anonymous blob access was disabled,
so even though anyone could reach the account, they could not access the data
inside without valid credentials. Finally, both account keys still showed
their original creation date from May, meaning they had not been rotated in
roughly six months, well past a 90 day rotation baseline.

This combination is consistent with a deployment that relied on default settings with no hardening. The pattern is not someone setting a bad config, but someone not setting a config at all.



## What Broke / what surprised me

## Findings and recommendations

## What I learned

