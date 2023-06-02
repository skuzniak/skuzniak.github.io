---
layout: post
title:  "Extracting data from Content Hub"
date:   2023-06-02 07:00:0 +0100
categories: content-hub external-systems good-neighbour
author: Szymon Kuzniak
---

Content Hub is a great tool to centralize all your content operations.
However, it can be tricky to get the content out of it to all the different channels you might want to support.
The main problem is obviously, how do you know if there is anything new to be synchronized, and I'm pretty sure you don't want to poll Content Hub every now and then to find it out.
Sure, it's possible to write a separate action for every channel and handle them using a trigger, but there's even a better way, which allows you to set up Content Hub just once, and then only connect new clients.

**Say hello to Azure ServiceBus.**

Let's check how easy it is.

First, let me answer a couple of obvious questions.

### Why Azure ServiceBus and not any other system?

The short answer is that it is natively supported by Content Hub.
There is a dedicated action, which sends the message to the ServiceBus and all that is needed, is the connection string and the name of the Topic.

### Azure ServiceBus also supports Queues, and Topics require higher service tier. Do I have to use Topics?

The difference between Queue and Topic is simple - Queue only allows one receiver of the message, while Topic allows multiple subscriptions and thus multiple receivers.
Topic is also the default choice if you want to connect Sitecore XP and is more versatile than Queue.
In this tutorial, I will focus on the Topics, however Queues are also supported out of the box.

### Can I use other message bus solutions?

I haven't tried that but I would say yes, you can.
However the experience won't be that smooth, since you will have to handle all the message composition and communication with the service on your own.

Now, let's see how easy it is to setup notifications about content updates in Content Hub.

## Azure ServiceBus

Let's start with ServiceBus configuration.
It's similar to what is required by a Sitecore Connect for Content Hub, so I could as well use this one.
Since I already covered that topic, you can refer to the detailed description from [this post](/2022/10/24/connect-content-hub-with-sitecore#setting-up-azure).

### High level list of steps is as follows:

1. Create new Azure ServiceBus instance
2. [Get the general connection string to the ServiceBus](/2022/10/24/connect-content-hub-with-sitecore#general-connection-string)
3. Create new Topic
4. Create new Subscription

## Content Hub Action & Trigger

Once the Azure ServiceBus is set up, you can proceed to configuring Action and Trigger in the Content Hub

First, an Action is needed, which can be executed by all the Triggers that are expected to send data to ServiceBus.
Start with creating new Action and set it's type to **Azure Service Bus**.
This allows you to provide the connection string to your ServiceBus instance, select whether you are using Topics or Queues as well as provide a name of the selected destination

<figure>
<img src="/assets/posts/sitecore-integration/10-content-hub-create-action.png" alt="Create Azure Service Bus Action." />
<figcaption>Create Azure Service Bus Action.</figcaption>
</figure>

This action can be reused anytime you want to send anything to this topic in Azure ServiceBus.

Once the action is ready, you need a Trigger which will call the action when certain criteria are met.
This one is up to the requirements of your project. 
You just have to remember that at the end, the Service Bus action has to be selected, so the message is actually send to the Service Bus

## Connect new client

The steps above should cover sending the message, now we need to consume it.
The simplest client code can be found below.

<script src="https://gist.github.com/skuzniak/8afa1042a7bf7ada7bb962b150759d28.js"></script>

It ain't much, just receiving the message and showing it in the console but that's only a starting point.
This is precisely what's needed for effective communication - a notification that a piece of content has been updated.
With this information you can now use Content Hub's API to get the details of updated content.

The best part?

Once you have this set up, all you need is another subscription in Azure ServiceBus and you can create another client.
Content Hub doesn't have to even be aware of where the content is going!

Happy coding!
