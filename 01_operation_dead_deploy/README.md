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

### Stage 3: Tracing the deployment

The resource group's Deployments blade listed a single deployment. Its name
suggested who owned it (an intern), what it was for (a test), and the year.
Automated pipelines typically name deployments with build numbers or commit
hashes, so a name that reads like a human typed it indicates the deployment
was run manually. Combined with the TLS 1.0 default from Stage 2, this is
consistent with a template or script launched by hand, outside any pipeline.

The deployment's start time matched the storage account's creationTime and
keyCreationTime in the JSON to within one second (May 24, 2026, 21:42 UTC).
Because three separate records agree, the evidence is corroborated: this
deployment created the storage account.

The deployment details also showed a second resource: a role assignment.
Role assignments do not appear in a resource group's resource list, so it
was only visible in the deployment record. This is arguably more significant
than the storage account itself, because a role assignment persists after the
intern's temporary access expires, and whoever received that role keeps their
permissions. I did not identify the principal that received the role during
this investigation; reviewing it is a recommended follow up.

### Stage 4: Why policy didn't stop it

The resource group's Policies blade showed the naming convention policy as
non compliant. The assignment was scoped to the subscription, so it applied
to the anomalous resource group through inheritance. Before concluding why it
failed to block the deployment, I ruled out the other ways a policy can be
bypassed: there were no excluded scopes, no exemptions, and policy
enforcement was enabled.

That left the effect. On the assignment's Parameters tab, the Effect
parameter was set to Audit. Audit allows a non compliant request to succeed
and only flags the resource; Deny is the effect that would have blocked it.
The parameter reference type was "Default value," meaning the effect came
from the policy definition's default rather than being set during
assignment. This is consistent with nobody deliberately choosing Audit; it
was a default that was never reviewed.

## What broke / what surprised me

* **Nothing would have alerted me.** When I first looked at the subscription,
  nothing stood out. The naming policy was set to Audit, which flags
  violations on the compliance dashboard but notifies no one. Detection
  depended entirely on someone happening to look.
* **Cost reporting wouldn't have caught it either.** My first thought was
  that unexpected spend would surface it, but an empty storage account costs
  pennies, and its cost center tag was a placeholder. Cost based detection
  catches expensive mistakes, not cheap risky ones.
* **The role assignment was hidden in plain sight.** The resource group
  showed one resource, but the deployment record showed two. The second was
  a role assignment, which can act as persistence after temporary access
  expires. I only found it because I opened the deployment details.
* **The network was open by default.** I didn't expect a new storage account
  to be reachable from any network on the internet without anyone
  deliberately choosing that.
* **I misread the timestamps at first.** I compared the JSON's UTC time to
  the portal's local time and thought they were hours apart. Converted
  to UTC, they matched to the second. Lesson: build timelines in UTC.
* **I skipped the JSON View on my first pass** and had to go back. Several of
  the most important configuration findings were only visible there.

## Findings and recommendations

## What I learned

