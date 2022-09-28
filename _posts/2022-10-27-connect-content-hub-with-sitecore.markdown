---
layout: post
title:  "How to connect Sitecore XP with Content Hub"
date:   2022-10-27 15:00:0 +0100
categories: content-hub sitecore-xp
author: Szymon Kuzniak
---
Sitecore claims to be "a good neighbor" in your infrastructure, meaning they put a lot of effort to be as easy to integrate as possible.
I think this is a very good direction, and it will only result in good design, but what about other products from Sitecore family?
I understand being nice to your neigbours but you would expect a little more towards your relatives, right?

Today I want to go through the configuration of the Content Hub to automatically work with Sitecore XP.
The process itself is fairly simple, although there are some caveats worth noting.

The connection is done through Azure Service Bus.
Once the entity is created or modified, Content Hub will post messages to Service Bus and forgets about them.
Sitecore Connector listens on the Service Bus to get the identifier of the updated entity.
Using the identifier, Sitecore Connector gets the entity details through the API, and creates an item within Sitecore.
Sitecore Connector can fill required fields, or link assets related to the entity.

The process can be divided into three parts

## Setting up Azure

Azure Service Bus abstracts communication between Content Hub and other systems.
Content Hub can write messages to the Bus without knowing about them in a generic way.
Thanks to this approach it is possible to connect many other systems with Content Hub. Not only Sitecore XP.

### Create new Service Bus subscription

In order to setup Azure Service Bus to work with Sitecore connector, you will need Topics.
This is important when choosing the pricing tier - it has to be at least Standard - Basic subscription will not have Topics enabled.

<figure>
<img src="/assets/posts/sitecore-integration/01-azure-servicebus-create.png" alt="Create new Azure Service Bus." />
<figcaption>Create new Azure Service Bus.</figcaption>
</figure>

### Get the general connection string

General connection string will be required for setting up the action in the Content Hub.
Connection strings can be obtained from Shared Access Policy.
You will have to create new one with Send and Listen claims.

<figure>
<img src="/assets/posts/sitecore-integration/02-azure-general-connection-strings.png" alt="Add new shared access policy." />
<figcaption>Add new shared access policy.</figcaption>
</figure>

Once the policy is created, you can get the connection string by clicking on the newly created policy.

<figure>
<img src="/assets/posts/sitecore-integration/03-azure-general-connection-string-get.png" alt="Get the general connection string." />
<figcaption>Get the general connection string.</figcaption>
</figure>

### Create topics

Content Hub will use one topic to write messages to, every time content should be synchronized with Sitecore.
Another one will be required for Sitecore Connector to report status of the synchronization.

<figure>
<img src="/assets/posts/sitecore-integration/04-azure-topic-create.png" alt="Create new topic." />
<figcaption>Create new topic.</figcaption>
</figure>

<figure>
<img src="/assets/posts/sitecore-integration/05-azure-topic-list.png" alt="Complete list of topics needed for the integration between Sitecore XP and Content Hub." />
<figcaption>Complete list of topics needed for the integration between Sitecore XP and Content Hub.</figcaption>
</figure>

### Create subscription

Sitecore Connector requires subscription on the topic which will be used to notify about new content (content-hub in my example).
The subscription is created from the topic screen, by clicking on the **+ Subscription** button.
For the purpose of the PoC I will use default values, but production use might require deeper dive into Azure documentation and fine tuning the settings of the subscription.

<figure>
<img src="/assets/posts/sitecore-integration/06-azure-topic-details.png" alt="Details of the topic 'content-hub' used to send messages from the Content Hub." />
<figcaption>Details of the topic 'content-hub' used to send messages from the Content Hub.</figcaption>
</figure>

<figure>
<img src="/assets/posts/sitecore-integration/07-azure-topic-subscription-create.png" alt="Create new subscription screen." />
<figcaption>Create new subscription screen.</figcaption>
</figure>

### Get connection strings for topics

Each topic can also has it's own connection strings.
They are used in configuration of the Sitecore Connector, so it knows where to get the messages from, and where to send the status.
For each topic, a new Shared Access Policy has to be created with appropriate claim.
Topic used to read the messages (content-hub), requires Listen claim, and topic used to send messages (sitecore-notification) requires Send claim.

<figure>
<img src="/assets/posts/sitecore-integration/08-azure-topic-shared-access-policy-listen.png" alt="Shared Access Policy for the 'content-hub' topic." />
<figcaption>Shared Access Policy for the 'content-hub' topic.</figcaption>
</figure>

<figure>
<img src="/assets/posts/sitecore-integration/09-azure-topic-shared-access-policy-send.png" alt="Shared Access Policy for the 'sitecore-notifications' topic." />
<figcaption>Shared Access Policy for the 'sitecore-notifications' topic.</figcaption>
</figure>

## Setting up Content Hub

Once the Service Bus is created, I can proceed to set up communication on the Content Hub side.
The main idea is to use the trigger to fire a service bus action when certain conditions are met.

### Create the action

So far, I was only writing about actions to execute a script.
To send the messages to Azure Service Bus, there is a dedicated type of action - Azure Service Bus.

<figure>
<img src="/assets/posts/sitecore-integration/10-content-hub-create-action.png" alt="Create Azure Service Bus Action." />
<figcaption>Create Azure Service Bus Action.</figcaption>
</figure>

The connection string required at that step is the general connection string for the entire Service Bus created in the second step.
The Destination Type should be left as Topic and the Destination name should be set to the topic which has the subscription set up.
In my case that would be content-hub.
Finally the **Test connection** button allows to see if the configuration is correct.

<figure>
<img src="/assets/posts/sitecore-integration/11-content-hub-test-action.png" alt="Test Azure Service Bus Action." />
<figcaption>Test Azure Service Bus Action.</figcaption>
</figure>

### Create the trigger

Once the action is created, I have to create the trigger which will be responsible for connecting system events (modification of a particular entity type in Content Hub) with the action, created in a previous step.
In order to do so, I will create a new trigger with the objective set to **Entity modification** and execution type set to **In background**.
In the Conditions section, I will select **M.Content** entity type, and narrow it only to Articles in a final state flow.

<figure>
<img src="/assets/posts/sitecore-integration/12-content-hub-trigger-condition.png" alt="Trigger condition setup." />
<figcaption>Trigger condition setup.</figcaption>
</figure>

As an action I will select the action I have created in a previous step.

## Setting up Sitecore XP