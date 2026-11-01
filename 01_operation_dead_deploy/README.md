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

## What Broke / what surprised me

## Findings and recommendations

## What I learned

