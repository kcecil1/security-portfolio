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

### Stage 1

While reviewing the subscription's resource groups, I found one entry that
didn't match the `rg-<workload>-<environment>-<region>-<instance>` naming
convention. Instead, it was named as a single word followed by a number
(`<word><number>`). It had no rg prefix, no environment indicator to show
whether it was production, development, or test, and no region. Its location
was also East US, while every other resource group was in Central US.

This suggested the resource group was created outside the normal
provisioning process, possibly by an inexperienced employee, a compromised
account, or a bad actor, so it was worth investigating further.

### Stage 2



## What Broke / what surprised me

## Findings and recommendations

## What I learned

