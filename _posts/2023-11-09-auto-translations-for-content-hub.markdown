---
layout: post
title:  "Automatic product translations in Content Hub - case study"
date:   2023-11-09 07:00:0 +0100
categories: content-hub external-systems azure natural-language-processing
author: Szymon Kuzniak
---

Long time there was no word from me but now, I'm coming back with new, exciting project, I was working on recently.
Our client requested an automated translation service for products from Content Hub PCM module.
As much as it was interesting, the project had also few challenges which I will describe here shortly.

The idea was simple - the client wanted to utilize Azure Natural Language processing service for translating their products from English to 15 other languages.
Sounds easy?

## Communication with the translation service

Our first problem was communication with Azure translation service.
Initially, we though it could be done directly from Content Hub script, however this quickly turned out to be impossible.
The version of the Content Hub we are using, does not permit access to `System.Net` namespace from the script level.
Fortunately there was an easy solution - utilizing API Action in Content Hub.

<figure>
<img src="/assets/posts/automatic-translations/01-api-call-action.jpg" alt="The API call action configuration dialog." />
<figcaption>The API call action configuration dialog.</figcaption>
</figure>

When executed, such an action will simply make a request to the API endpoint.
There is an option to select which HTTP method will be used and whether to pass some headers or parameters.
All the logic can be then handled within the API.

We have decided to use Azure Function for the API backend, as it allows for a  1 million free executions monthly.
This should be more than enought in our case.
On top of that rest of the client's infrastructure is also hosted on Azure so this was rather natural choice.

It's worth mentioning that API Action does not have any requirements about where and how the API is implemented.
The only requirement is that the API is reachable by Content Hub.

## Passing parameters to the function

Content Hub does allow to dynamically pass property values as the API call parameters.
However, this only works for properties which are not multilanguage (or we simply weren't able to figure out correct notation).
Instead of passing all the data to translate as API call parameters, we used the entity ID sent with the API call, to get the original values using Content Hub API.

<figure>
<img src="/assets/posts/automatic-translations/02-data-flow.jpg" alt="The flow of the data in the solution." />
<figcaption>The flow of the data in the solution.</figcaption>
</figure>

This solution provided additional benefit.
In the middle of the project, we have found out that not all data is available in English, but there is a lot of German copy.
From the Azure function level, we were able to easily implement fallback mechanism.
If the English property is empty, we can check for the German version and use this one.
That would not have been so easy if we passed all the English data as parameters.

## Enabling translation for the users

Once the translation process was in place, we needed to make it available for the users.

Initially we agreed to implemented the translation as a part of the product state flow.
This seemed like a natural choice, because we assumed the translation is part of the product creation process.

We couldn't be more wrong.

The client wanted to translate products, which were already created and published.
If we would implement the translation process as part of the state flow, it would require to unpublish products which were translated.
This approach was not acceptable and we needed to think of something else.

Our second idea was to attach the traslation Action to the custom button on the product list and the product detail page.
This was much better approach for our client as it didn't require unpublishing products and allowed them to translate products in different states of the state flow.

## Problems with products versions

Finally, almost at the end of the project we have discovered, what was our biggest problem.

The entities of the `M.PCM.Product` do not have versions.
This means that if the product is published, it is published in all the languages.
And in consequence when the automatic translation will be triggered for the product, the translated copy will immediately be available on the client's webpage.
Given the uncertainty of the automated translation quality, this was not acceptable.
Translated copy still needs to be reviewed and approved by a human.

There were couple of ideas how to approach this issue.

The first propsition, was to temporarily unpublish the product, translate, review and refine the content, and publish the product back.
Well, long story short, we've been asked to come up with something more robust.
Another suggestion was to create separate entity for each language, but the client did not like the idea either.

Finally, we have added a multilanguage, boolean property which would be checked when the translation is reviewed and accepted.
On top of that, the connector, which synchronizes products to the website was updated to check the status of the translation and use only approved copy. This approach allows us to control each language separately, so the client can process each language in their own pace. 

## Summary

Thanks to the extensibility of the Content Hub we could implement this requirement in less than a month.
Of course the solution is more of a MVP, but it allowed the client to evaluate the translaiton quality and decide if they want to continue work on the solution to make it better and more user friendly.

We are currently thinking on introducing versioning to the product data, but this is definitely topic for another blog post.

See you next time.