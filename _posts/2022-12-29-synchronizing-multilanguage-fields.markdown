---
layout: post
title:  "Synchronizing multilanguage fields with Sitecore CMP connector"
date:   2022-12-29 07:00:0 +0100
categories: content-hub sitecore-xp
author: Szymon Kuzniak
---
I was about to demo the connection between Sitecore Content Hub and Sitecore XP, and I though that apart from regular content elements, such as articles or blog posts, I could maybe synchronize also products.
I was quite surprised, when I found out that I can't acutally synchronize majority of the fields of my products because... they are multi-lingual.

When I first tried to synchronize the product, I started getting following exceptions in the log files

```
19756 23:56:42 ERROR [Sitecore Connect for Content Hub]: An error occured during converting 'VolumeLabel' field to '{F7DE3FC9-70CC-482C-95A3-558CB3127C26}' field. Field mapping ID: '{31B3FCE4-1E66-4F2F-96B3-0A4DB3BDF8E1}'.
Exception: System.InvalidOperationException
Message: Culture is required for culture sensitive properties.
Source: Stylelabs.M.Sdk
   at Stylelabs.M.Sdk.Models.Base.EntityBase.GetPropertyValue(String name)
   at Sitecore.Connector.CMP.Pipelines.ImportEntity.SaveFieldValues.TryMapConfiguredFields(ImportEntityPipelineArgs args)
```

My first thought - I have probably selected incorrect mapping template or did not provide the culture.
However, upon further investigation, I noticed there is no other template for mapping Content Hub's multilanguage fields.

Fortunately, thanks to Sitecore's extensibility, I was able to write a small processor, which will fill this little gap.

### A bit of theory first.

The connector configuration can be found in `App_Config\Modules\Sitecore.Connector.CMP` folder.
There are three files - the one I was interested in is called `Sitecore.Connector.CMP.config`.
The connector uses pipeline `cmp.importEntity` to process imported entities, so all I had to do was to identify correct processor.
The one with type `Sitecore.Connector.CMP.Pipelines.ImportEntity.SaveFieldValues` seemed like a good fit.

With a little help of dotpeek, I have confirmed this is a problem with connector - there is either a relation mapping (not relevant in my case), or field mapping, which uses `entity.GetPropertyValue(name)` method.
This method is throwing the `InvalidOperationException` when called with a multilanguage property name.

### The plan to fix.

1. Create a separate field mapping template with versioned `Culture` field (so it can be filled in different cultures)
2. Create a processor which will inherit from `Sitecore.Connector.CMP.Pipelines.ImportEntity.SaveFieldValues`
3. Handle the new template in the new processor
4. Fallback to base implementation for the existing templates

#### The templates

<figure>
<img src="/assets/posts/cmp-connector/new-templates.jpg" alt="New templates." />
<figcaption>New templates required for the extension to work.</figcaption>
</figure>

I had to create one base template - `CMP Culture Settings` with a single, versioned field to store the culture.
Then I created a mapping template and make it inherit from templates: `CMP Culture Settings`, `CMP Field Name Settings` and `Sitecore Field Name Settings`.
I also had to update the insert options for `Entity Mapping` template, to allow creating items based on new template.

#### The code of the processor

<script src="https://gist.github.com/skuzniak/a6286da0b32141ef0e4fe989f718ce07.js"></script>

And finally the configuration that has to be installed in the `App_Config/Include` folder.

<script src="https://gist.github.com/skuzniak/f40aa89a1b426a9fa72df0b3fcc6ea44.js"></script>

Now the import of multilanguage fields should be working.

Happy coding!
