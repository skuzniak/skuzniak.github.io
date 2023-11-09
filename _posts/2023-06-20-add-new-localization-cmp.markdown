---
layout: post
title:  "Add new localization to the CMP entity"
date:   2023-06-20 07:42:31 +0100
categories: content-hub cmp
---
Adding new localization might not sound like a fancy topic for a blog post.
Well, it turns out in the case of CMP there might be some interesting caveats worth knowing about.

Let's ride.

### First things first.

In order to localize a content entity, you need to click Localize button in the top right, three dots, menu:

<figure>
<img src="/assets/posts/cmp-localizations/01-localize.jpg" alt="The Localize button on the CMP content entity." />
<figcaption>The Localize button on the CMP content entity.</figcaption>
</figure>

And select your locale:

<figure>
<img src="/assets/posts/cmp-localizations/02-locale-select.jpg" alt="Selection of the locale when creating a new localized version." />
<figcaption>Selection of the locale when creating a new localized version.</figcaption>
</figure>

From a list:

<figure>
<img src="/assets/posts/cmp-localizations/03-available-localizations.jpg" alt="List of available localizations to select." />
<figcaption>List of available localizations to select.</figcaption>
</figure>

### Problem 1 - New localization values

Imagine you want to add new localization in the first place.
That's not done through the portal languages - they are independent.
In fact, localizations are stored in a taxonomy - `M.Localization`.

You can easily find it by going to the **Manage** section, then **Schema** and finding `M.Content` entity.

<figure>
<img src="/assets/posts/cmp-localizations/04-content-entity-definition.jpg" alt="The M.Content entity definition." />
<figcaption>The M.Content entity definition.</figcaption>
</figure>

Armed with this knowledge, you can now go back to the **Manage** section, then **Taxonomy** and find the `M.Localization` taxonomy.
You can add new localization values by clicking the '+' button in the upper bar.

<figure>
<img src="/assets/posts/cmp-localizations/05-localization-taxonomy.jpg" alt="Adding new localization to M.Localization taxonomy." />
<figcaption>Adding new localization to M.Localization taxonomy.</figcaption>
</figure>

Values from this taxonomy will appear on the list when creating new localization.

### Problem 2 - Create new localization

Now that you have all the locales you need, there shouldn't be a problem with adding new, localized versions of your content, right?
Well, that depends.

You might find a problem when creating a new version, which doesn't seem to make sense at the beginning.

<figure>
<img src="/assets/posts/cmp-localizations/06-create-new-version-problem.jpg" alt="Validation error when creating new localization." />
<figcaption>Validation error when creating new localization.</figcaption>
</figure>

Even though you have provided both mandatory fields, the version does not validate and cannot be saved.
The message reads **"Field must be filled in for the original entity"**.

Much better message, which actually helped me to solve this problem is the notification message (which disappears pretty fast) and it reads **"Make sure that the required fields are filled in for both the original an the variant entity"**.

The actual problem is, the original version also needs the locale to be set up, and the field seems to be manadatory.
However, when you create the content, Locale field is not marked as one and you can skip it.
In this case you will end up with the following issue.

So, either remember about filling in this field every time you create new version, or simply go to content Details secion and provide the necessary value.

<figure>
<img src="/assets/posts/cmp-localizations/07-content-details.jpg" alt="Add missing local on content Details tab." />
<figcaption>Add missing local on content Details tab.</figcaption>
</figure>

When the locale for the original version is provided, other localizations can be created without any problems.

Happy content creation.
