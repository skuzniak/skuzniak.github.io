---
layout: post
title:  "Sitecore Content Hub for Developers"
date:   2022-04-11 15:00:0 +0100
categories: sugcon content-hub speaking
author: Szymon Kuzniak
---

It's trivia to say that time flies.
<figure>
<img src="/assets/posts/content-hub-for-developers/sugcon-before-presentation.jpeg" alt="Me with a nervous smile before my presentation." />
<figcaption>Me with a nervous smile before my presentation.</figcaption>
</figure>

I remember preparing my submission for Sugcon CFP in December, then the shock when my talk was accepted and freaking out for three months about presenting in front of entire Sitecore community and now here we are, two weeks after this splendid event. 
The conference was great as previous two events I have attended and it was good to see that the community is still strong.

If you haven't attended, or perhaps if you would like to recall some of the details of my talk, you can find below a short summary.

The event was opened by Sitecore CEO Steve Tzikakis himself and it was really encouraging to hear directly from him how much Sitecore cares about the community.
Steve talked about the direction Sitecore is heading which might seem revolutionary from my, as a .NET developer, point of view.

<figure>
<img src="/assets/posts/content-hub-for-developers/sugcon-opening-keynote.jpeg" alt="Sitecore CEO Steve Tzikakis during keynote." />
<figcaption>Sitecore CEO Steve Tzikakis during keynote.</figcaption>
</figure>

The word **composable** was nearly overused during the keynote and the Sitecore strategy overview given shortly after by Dave O'Flanagan.
It describes a new suite of Sitecore products.
Independent, API and cloud based applications, which can be used to compose any architecture you or your clients should require.

Sitecore content hub is one of those products.
And you might ask why invest into another content related application when there is already one we all know and love - good, old Sitecore XP.
It all becomes clear, when you look at the role of each of those systems.
While Sitecore XP's main role is to deliver the content to the customers, Content Hub's responsibility is to deliver the content to the internal systems and employees.
Long time ago Sitecore XP should have started being considered as just one of the channels for content delivery.
And since content is distributed acorss different channels in various forms, you can think of all those web editors who are copy-pasting it from Word documents into Sitecore.
With Content Hub they can focus on arranging the content on the page using Sitecore XP because the content is already there.
It is fed automatically from Content Hub where it was produced.
In this way Content Hub complements Sitecore XP.

## What is Content Hub

Content Hub consists of 5 modules, all off them accessible through the navigation.
There are three core content repositories for different types of content: assets, products and text.
Content Hub's main goal is to provide easy access to thousands of elements so it puts heavy focus on search and share functionality.
On top of that there are two additional modules: for organizing work into tasks and projects and for exporting content elements into digital assets using flexible templates.

<figure>
<img src="/assets/posts/content-hub-for-developers/content-hub-pim-overview.jpg" alt="Content Hub is a repository for content" />
<figcaption>Content Hub is a repository for content</figcaption>
</figure>

This is how the users see Content Hub.

Internally, a very important concept is schema.
Schema defines all the available entities - products, assets and content elements as well as taxonomies used to describe them.
Each entity definition determines what properties are available and what relations to other entities are possible to set up.
Just like in Sitecore XP everything is an item, in Content Hub nearly everything is an entity.

## Role of a developer

For most of the time working with Content Hub is about configuration of what users can or cannot do.
This includes mostly changes to the schema or the user interface.
But sometimes it's also about making users life a little bit easier.
Or ours.
And this can be done by automating all off the boring and repetitive stuff that has to be done, using scripts.
The scripts are written in a web based editor inside Content Hub.
Thankfully the scripts are also based on the Web SDK, a C# based API to interact with the Content Hub.
And this has two implications - they can be developed using Visual Studio.
And they are quite powerful.

My idea for working with the scripts was pretty simple.
I have set up a base class which consits of a Execute and SetUp methods.
In a SetUp method I am creating a script context which is just an object based on [this documentation page](https://docs.stylelabs.com/contenthub/4.0.x/content/api-reference/scripting/stylelabs.m.scripting.types.v1_0.action.iactionscriptcontext.html).
The Execute method contains actual script.
In this way I can develop my script inside Visual Studio, taking advantage of all of its cool features and once the script is ready I can simply copy it to Content Hub.
This is maybe a little naive solution but it's a good start for something more comprehensive, like [this example Content Hub solution](https://github.com/sitecore/contenthub-vs-solution-example).

## Let's talk about automation

Software development is not only programming, is also deployment and automation.
In case of Content Hub, Sitecore is responsible for the deployment and maintenance of the application but the developers are responsible for deployment of the configuration.

Depending on the license agreement Content Hub is offered with up to three environments.
One environment is meant for development, second for testing new configuration changes and finally the last one for the end users.
The changes in configuration can be moved between the environments using export/import feature.
This is quite simple process of clicking a button on one environment and then clicking a button on another one.
However, I really like to take a backup of the environment where I'm deploying changes.
I also tend to rename the packages to include the name of the environment and copy them over to a shared location.

And in this is only the beginning.

Sometimes it might be required to exclude taxonomies from the deployment (because of their size, which might be impactin deployment time and small frequency of changes) so it means the packages must be split to be able to install them in correct order.
This doubles the clicking and renaming.
Another package split might be needed in order to update connection strings in actions which are connecting to Azure or other services.
That's even more clicking and renaming.

If only it was possible to script the entire process.

Well it turns out it is.
Content hub provides command line interface which allows for some handy operations: synchronizing scripts, checking statuses of jobs and of course exporting and importing packages.
If that's not enough is possible to develop custom plugins.
It is possible to select which system components to include in the package, where to save it and how it should be named.
Just perfect.

## Sitecore XP and Content Hub

It is of course possible to connect Content Hub with Sitecore XP.
Sitecore offers a package which has to be installed and of course a little configuration is necessary to make the two systems talk to each other.
Two modules which are supported for such a connection are DAM and CMP.

DAM support is really simple since the assets are linked on demand by Sitecore XP users whenever they need to include an image.
Inside Content Hub there is a dedicated asset search page which should be configured in Sitecore, and a small adjustment in allowed content header does the trick.

CMP support is more sophisticated since the communication is reversed.
It's Content Hub which pushes content to Sitecore every time new piece is accepted.
This communication is done using Azure Service Bus and Content Hub Action to post messages to a Service Bus topic which is then monitored by Sitecore to get the content from Content Hub automatically.

I think that configuration details really deserves a dedicated article, which I would like to publish soon.

## Does a cloud based application require a developer

This was the question I started my talk with.
After couple of months with Content Hub I am sure there is a space for someone with developer background to use their skills working with this system.
And I am pretty sure there is even more to discover!

At the end I would like to say big köszönöm (thank you in Hungarian) to everyone who helped me prepare and deliver this talk.
Everyone involed in organization of this great event - without you my talk would have never existed.
Łukasz Skowronski who was a great support as a room master.
Sebastian Winter who helped me shape the presentation.
All my friends and collegues from Poland and Apollo Division who showed their support.

<figure>
<img src="/assets/posts/content-hub-for-developers/sugcon-budapest.jpeg" alt="Mandatory run to catch-up with the sightseeing." />
<figcaption>Mandatory run to catch-up with the sightseeing, after spending first day on rehersing.</figcaption>
</figure>