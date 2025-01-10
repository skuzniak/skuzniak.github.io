---
layout: post
title:  "Sitecore Buckets DRY tip"
date:   2025-01-09 07:00:0 +0100
categories: sitecore buckets constants dry
author: Szymon Kuzniak
---

Happy new year!

This one is going to be quick.
Recently I wrote this piece of code:

<pre>
var bucketFolderIndex = breadcrumb
    .Select((ancestor, index) => new { ancestor, index })
    .Where(element => element.ancestor.TemplateID == ID.Parse("{ADB6CA4F-03EF-4F47-B9AC-9CE2BA53FF97}"))
    .Select(element => element.index)
    .FirstOrDefault() + 1;
</pre>

It's fairly simple - it goes through the list of item ancestors, trying to find an item which template is a concrete ID and returns its position.
In this case I'm searching for a postion of an item of a `Bucket` template.

Of course this piece would never pass the code review, because of the hardcoded ID.
In the same time, I didn't really want to make constant in our codebase out of Sitecore's internal item.

I was hoping for a better way to do this. I couldn't find anything on the internet, so I dig down into the source.

It was a good decision, because it turns out bucket template is configurable, so it is not stored in any of the constants in Sitecore dlls.
In fact I also shouldn't put it as a constant.
However, there is a dedicated class that will help you access all the required item IDs.

You can access `Bucket` template ID with this tiny piece of code:

<pre>
Sitecore.Data.ID bucketTemplateId = Sitecore.Buckets.Util.BucketConfigurationSettings.BucketTemplateId
</pre>

This code will access a configuration setting `BucketConfiguration.BucketTemplateId` and will default to `{ADB6CA4F-03EF-4F47-B9AC-9CE2BA53FF97}` if the setting cannot be found.

Of course there are other important buckets' settings and constanst available in the `Sitecore.Buckets.Util.BucketConfigurationSettings`.

Happy coding!