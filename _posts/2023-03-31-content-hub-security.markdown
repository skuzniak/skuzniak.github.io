---
layout: post
title:  "Content Hub security 101 - managing multiple similar usergroups"
date:   2023-03-31 07:00:0 +0100
categories: content-hub security
author: Szymon Kuzniak
---
Content Hub security model seems pretty simple - there are rules for entity types with conditions and a set of available access rights.
What can potentially go wrong?

## The scenario

Let's assume there are multiple markets configured in your application.
There are also assets, which can be viewed by everyone, but reviewed and approved only by certain group of users.
Such groups, allowed to approve only certain market, can be of course more.
Oh, and there are users who can review more than one market.

Let's see what do I have at the beginning.

<figure>
<img src="/assets/posts/content-hub-security-pt1/01-initial-example.jpg" alt="The initial scenario with two images that belongs to different markets." />
<figcaption>The initial scenario with two images that belongs to different markets.</figcaption>
</figure>

The screenshot above shows the initial scenario, where the user doesn't have access to approve any of the assets.
My goal is to configure the security in a way, the user will be able to approve German market asset, but not the French one.

My first idea would be to setup a user group that will allow him to do so.

### The all-in-one market reviewer group

The combination below shows how I would approach the problem without understanding of Content Hub security model.

| Rule for Entity | Conditions | Access Rights |
|---|---|---:|
| M.Localization | - | Read |
|M.File, M.Asset | M.Final.LifeCycle.Status: Under Review<br />M.Final.LifeCycle.Status: Under Review<br />M.Content.Repository: Standard<br />M.PCM.Market: Germany | Read, Update, Approve<br />CreateAnnotations, ReadAnnotations |

The policy above will work, and the user will have required access.
The problem with this approach is that, whenever I want to add support for another market, I will have to recreate this entire group, just to change the market condition.
This is not an efficient approach - when the group grows, the maintenance will become a huge problem, and the groups will quickly go out of sync.

**Much better solution would be to extract the part of the group, that is common for every reviewer, and the varying part into separate groups.**

### Splitting the all-in-one group

Let's start with extracting the common part.

#### The local reviewers group

| Rule for Entity | Conditions | Access Rights |
|---|---|---:|
| M.Localization | - | Read |
|M.File, M.Asset | M.Final.LifeCycle.Status: Under Review<br />M.Final.LifeCycle.Status: Under Review<br />M.Content.Repository: Standard | Read, Update, Approve<br />CreateAnnotations, ReadAnnotations |

#### The concrete market group

| Rule for Entity | Conditions | Access Rights |
|---|---|---:|
|M.File, M.Asset | M.PCM.Market: Germany | Read, Update, Approve<br />CreateAnnotations, ReadAnnotations |

Thanks to this maneouver, whenever new market role has to be added, all that has to be done is to create new tiny, concrete market group.
Additionally, every time some general adjustments have to be applied to all of the reviewers, there is only one common group to be changed.

### How to use two groups as one?

The problem now, is that once I apply those two groups to the user, in a similar way as I did previously, the result will be quite different.

#### Default combination - ANY

| Policies combination | The result |
|---|---|
| <figure><img src="/assets/posts/content-hub-security-pt1/02-policy-combination-any.jpg" alt="The result of applying new groups to the user with ANY policies combination." /></figure> | <figure><img src="/assets/posts/content-hub-security-pt1/01-initial-example.jpg" alt="The initial scenario with two images that belongs to different markets." /></figure> |

Why did this happen?

This is the result of one of the Content Hub security principles - it is impossible to deny access, once the access had been granted.
In case of **Any** combination of the policies, the local reviewers group grants access regardless of the market, so concrete market group just doesn't have any effect.
The concrete group could be as well be removed.

#### Change combination to ALL

If I change the combination to **All** the effect will be even worse, because the user will lose all the access, since this combination will be far too restrictive.

| Policies combination | The result |
|---|---|
| <figure><img src="/assets/posts/content-hub-security-pt1/04-policy-combination-all.jpg" alt="The result of applying new groups to the user with ALL policies combination." /></figure> | <figure><img src="/assets/posts/content-hub-security-pt1/01-initial-example.jpg" alt="The initial scenario with two images that belongs to different markets." /></figure> |

#### The solution - combine them both

The solution is to use the combination of the two and recreate the initial setup.
On the top level, there should be **Any** combination applied, however both local and concrete groups should be combined toghether using **All** combination.
This concept can be easily visualized by comparing two working solutinos, side by side.

| Initial policy combination | Updated policy combination |
|---|---|
| <figure><img src="/assets/posts/content-hub-security-pt1/06-policy-combination-initial.jpg" alt="The initial policies combination." /></figure> | <figure><img src="/assets/posts/content-hub-security-pt1/07-policy-combination-mixed.jpg" alt="The updated policies combination." /></figure> |

The set with policy combination ALL combines previously separated groups into one.
On the top level the groups are still combined using ANY combination, as in the original setup.

Using this approach, it is possible to keep the groups small and reusable, and easily join them as much as it is needed.

Key takeovers:

* When using **ANY** policy combination, the access is granted if **any** of the policies grants access.
* When using **ALL** policy combination, the access is granted if **all** of the policies **combined** grant the access.
* As a general rule of thumb, you should always use **Any** policy combination on the top level and **All** policy combination to join policies into a single one.
