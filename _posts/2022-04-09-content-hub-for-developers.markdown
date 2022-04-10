---
layout: post
title:  "Sitecore Content Hub for Developers"
date:   2022-04-09 15:00:0 +0100
categories: sugcon content-hub speaking
author: Szymon Kuzniak
---

It's trivia to say that time flies.
I remember preparing my submission for CFP in December, then the shock when my talk was accepted and freaking out for three months about presenting in front of entire Sitecore community and now here we are, two weeks after this splendid event. 
The conference was great as previous two events I have attended and it was good to see that the community is still strong.

If you haven't attended, or perhaps if you would like to recall some of the details of my talk, you can find below a short summary.

The event was opened by Sitecore CEO Steve Tzakikis himself and it was really encouraging to hear directly from him how much Sitecore cares about the community.
Steve talked about the direction Sitecore is heading which might seem revolutionary from my, as a .NET developer, point of view.
The word 'composable' was nearly overused during the keynote and the Sitecore strategy overview given shortly after by Dave O'Flanagan.
It describes a new suite of Sitecore products.
Independent, API and cloud based applications, which can be used to compose any architecture you or your clients should require.

Sitecore content hub is one of those products.
And you might ask why invest into another content related application whether there is already one we all know and love - good, old Sitecore XP.
It all becomes clear, when you look at the role of each of those system.
While Sitecore XP's main role is to deliver the content to the customers, Content Hub's responsibility is to deliver the content to the internal systems and employees.
Remember that Sitecore XP is just one of the channels for content delivery.
Think of all those web editors who are copy-pasting content from Word documents into Sitecore - with Content Hub they can focus on arranging content on the page using Sitecore XP because the content is already there.
It is fed automatically from Content Hub where it was produced.
Magic?
No, it's composable architecture.
In this way Content Hub complements Sitecore XP.



Content Hub consists of 5 modules, all off then accessible through the navigation.
There are three core content repositories for different types of content: assets, products and text.
Content Hub's main goal is to provide easy access to thousands of elements so it puts heavy focus on search and share functionality.
On top of that there are two additional modules: for organizing work into tasks and projects and for exporting content elements into digital assets using flexible templates.
This is how the users see Content Hub.
Internally, a very important concept is schema.
Schema defines all the available entities - products, assets and content elements as well as taxonomies used to describe them.
Each entity definition determines what  properties are available and what relations to other entities are possible to set up.
Properties can


What is the role of a developer you might ask.
For most of the time working with content hub is about configuration of what users can or cannot do.
This includes mostly changes to the schema or the user interface.
But sometimes it's also about making  users life a little bit easier.
Or ours.
And this can be done by automating all off the boring and repetitive stuff that has to be done, using scripts.
Scripts  can executed in many  ways.
They can be triggered by a CRUD operation on an entity.
They can be triggered by a user login.
They can be attached to a button.
There's one catch though.
The scripts are written in a web based editor inside content hub.
Thankfully the scripts are also based on the Web SDK, a C# based API to interact with the content hub
And this has two implications - they can be developed using visual studio.
And they are quite powerful.
Let's have a look.



Software development is not only programming, is also deployment and automation.
In case of content hub Sitecore is responsible for the deployment and maintenance of the application but the developers are responsible for deployment of the configuration.
Depending on the license agreement content hub is offered with up to three environments.
One environment is meant for development, second for testing new configuration changes and finally the last one for end users.
The changes in configuration can be moved between environments using export/import feature.
This is quite simple process of clicking a button on one environment and then clicking a button on another one.
However I really like to take a backup of the environment where I'm deploying changes.
I also tend to rename the packages to include the name of the environment and copy them over to a shared location.
And in this is only the beginning.
In the case of our project we wanted to exclude taxonomies from the deployment so it means we need to split the packages to be able to install them in correct order.
This doubles the clicking and renaming.
Another package split might be needed in order to update connection strings in actions which are connecting to Azure or other services.
Even more clicking and renaming.
If only it was possible to script the entire process.
Well it turns out it is.
Content hub provides command line interface which allows for some handy operations: synchronizing scripts (there's a lightning talk about that), checking statuses of jobs and of course exporting and importing packages.
If that's not enough is possible to develop your own plugin.
But that's a topic on separate talk.
It is possible to select which system components to include in the package, where to save it and how it should be named.
Just perfect.
Let me show you.

There are two possibilities to connect to content hub either by using a token from the logged in user or by providing username and password.
In the first option CLI will open content hub insurance and ask for the access permission from currently logged in user.
The latter is quite obvious, user name and password have to be provided when creating connection.
