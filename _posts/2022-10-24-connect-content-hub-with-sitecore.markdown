---
layout: post
title:  "How to connect Sitecore XP with Content Hub"
date:   2022-10-24 07:00:0 +0100
categories: content-hub sitecore-xp
author: Szymon Kuzniak
---
Today, I would like to go through the configuration of the Content Hub, to automatically synchronize content with Sitecore XP.
The process itself is fairly simple, although there are some caveats worth noting.

The connection is done through Azure Service Bus.
Once the entity is created or modified, Content Hub will post message with the updated entity identifier to Service Bus and forgets about them.
Sitecore Connector, a dedicated module installed on Sitecore XP, listens on the Service Bus to get the identifier of the updated entity.
Using the identifier, Sitecore Connector gets the entity details through the API, and creates an item within Sitecore, based on the mapping between the entity and the item template.
Sitecore Connector can fill required fields, or link assets related to the entity.

Azure Service Bus abstracts communication between Content Hub and other systems.
Content Hub writes messages to the Service Bus without the knowlede about the receiver at the other end.
Thanks to this approach, it is possible to connect many other systems with Content Hub.
First two sections of the below instructions are generic, and can be used to set up the connection regardless of what system will receive the messages.
The last section is dedicated to Sitecore XP consumer.

The configuration can be divided into three parts.

## Setting up Azure

### Create new Service Bus subscription

In order to setup Azure Service Bus to work with Sitecore connector, you will need Topics.
This is important when choosing the pricing tier - it has to be at least **Standard** - Basic subscription will not have Topics enabled.

<figure>
<img src="/assets/posts/sitecore-integration/01-azure-servicebus-create.png" alt="Create new Azure Service Bus." />
<figcaption>Create new Azure Service Bus.</figcaption>
</figure>

### <span id="general-connection-string">Get the general connection string</span>

General connection string will be required for setting up the action in the Content Hub.
Connection strings can be obtained from `Shared Access Policy`.
You will have to create new one with the Manage claim (Send and List claims will be included automatically).

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
Another one will be required for Sitecore Connector to report status of the synchronization (this is outside of the scope of this article).

<figure>
<img src="/assets/posts/sitecore-integration/04-azure-topic-create.png" alt="Create new topic." />
<figcaption>Create new topic.</figcaption>
</figure>

<figure>
<img src="/assets/posts/sitecore-integration/05-azure-topic-list.png" alt="Complete list of topics needed for the integration between Sitecore XP and Content Hub." />
<figcaption>Complete list of topics needed for the integration between Sitecore XP and Content Hub.</figcaption>
</figure>

### <span id="subscription">Create subscription</span>

Sitecore Connector requires subscription on the topic which will be used to notify about new content (content-hub in my example).
The subscription is created from the topic screen, by clicking on the `+ Subscription` button.
For the purpose of this PoC, I am using default values, but production use might require deeper dive into Azure documentation and fine tuning the settings of the subscription.

<figure>
<img src="/assets/posts/sitecore-integration/06-azure-topic-details.png" alt="Details of the topic 'content-hub', used to send messages from the Content Hub." />
<figcaption>Details of the topic 'content-hub', used to send messages from the Content Hub.</figcaption>
</figure>

<figure>
<img src="/assets/posts/sitecore-integration/07-azure-topic-subscription-create.png" alt="Create new subscription screen." />
<figcaption>Create new subscription screen.</figcaption>
</figure>

### <span id="topic-connection-strings">Get connection strings for topics</span>

Each topic also has its own connection strings.
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

Once the Service Bus is configured, you can proceed to set up communication on the Content Hub side.
The main idea is to use the trigger, to fire a service bus action when certain conditions are met.

### Create the action

So far, I was only writing about actions to execute a script.
To send the messages to Azure Service Bus, there is a dedicated type of action - Azure Service Bus.

<figure>
<img src="/assets/posts/sitecore-integration/10-content-hub-create-action.png" alt="Create Azure Service Bus Action." />
<figcaption>Create Azure Service Bus Action.</figcaption>
</figure>

The connection string required at that step is the [general connection string](#general-connection-string) for the entire Service Bus, created in the second step.
The Destination Type should be left as `Topic`, and the Destination name should be set to the topic, for which you have created the subscription.
In my case that would be `content-hub`.
Finally the `Test connection` button allows you to see if the configuration is correct.

<figure>
<img src="/assets/posts/sitecore-integration/11-content-hub-test-action.png" alt="Test Azure Service Bus Action." />
<figcaption>Test Azure Service Bus Action.</figcaption>
</figure>

### Create the trigger

Once the action is created, I have to create the trigger which will be responsible for connecting system events (modification of a particular entity type in Content Hub) with the action, created in a previous step.
In order to do so, I will create a new trigger with the objective set to `Entity modification` and execution type set to `In background`.
In the Conditions section, I will select `M.Content` entity type, and narrow it only to Articles in a final state flow.

<figure>
<img src="/assets/posts/sitecore-integration/12-content-hub-trigger-condition.png" alt="Trigger condition setup." />
<figcaption>Trigger condition setup.</figcaption>
</figure>

As an action I will select the action I have created in a previous step.

## Setting up Sitecore XP

So far, I have only configured one side of the connection. Content Hub, every time there is an article to synchronize, will send a message to Azure Service Bus, which will contain the identifier of the article.
Now I need to set up a consumer for those messages.

### Sitecore Connect for Content Hub

First of all, a content package with [Sitecore Connect](https://dev.sitecore.net/Downloads/Sitecore_Connect_for_Content_Hub.aspx) have to be installed.
This is a standard Sitecore package, which has to be installed using installation wizard.

### Connection Strings to Service Bus

Once Sitecore Connect is installed it is time to set the connection strings.
Sitecore needs [connection strings to both topics](#topic-connection-strings) and the name of the [subscription](#subscription) which has to be copied from the Azure Service Bus and added to the connection strings.

* CMP.ServiceBusEntityPathIn - connection string to the topic with incoming messages (content-hub in my case)   
<figure>
<img src="/assets/posts/sitecore-integration/08-azure-topic-shared-access-policy-listen.png" alt="Shared Access Policy for the 'content-hub' topic." />
<figcaption>Shared Access Policy for the 'content-hub' topic.</figcaption>
</figure>

* CMP.ServiceBusSubscription - name of the subscription (sitecore in my case)
* CMP.ServiceBusEntityPathOut - connection string to the topic with outgoing messages (sitecore-notifications in my case)   
<figure>
<img src="/assets/posts/sitecore-integration/09-azure-topic-shared-access-policy-send.png" alt="Shared Access Policy for the 'sitecore-notifications' topic." />
<figcaption>Shared Access Policy for the 'sitecore-notifications' topic.</figcaption>
</figure>

The connection strings to the Azure Service Bus will have the following form:

    <add name="CMP.ServiceBusEntityPathIn" connectionString="Endpoint=sb://szymonkuzniak-contenthub.servicebus.windows.net/;..." />
    <add name="CMP.ServiceBusSubscription" connectionString="sitecore" />
    <add name="CMP.ServiceBusEntityPathOut" connectionString="Endpoint=sb://szymonkuzniak-contenthub.servicebus.windows.net/;..." />

### Connection String to Content Hub

Serivce Bus message only notifies Sitecore about the change and provides the identifier of updated entity.
There is no other information carried over the message, so Sitecore needs to connect to Content Hub and get all the information it needs.
To connect to Content Hub, I need to provide the following information:

* ClientId and ClientSecret - those can be taken from the OAuth section from Content Hub
* Username and password - create a username dedicated to communication between Sitecore and Content Hub
* URI - the URL to the Content Hub instance

```
    <add name="CMP.ContentHub" connectionString="ClientId={client_id};ClientSecret={client_secret};UserName={username};Password={password};URI={uri};" />
```

### Configuration of the Sitecore Connect

Sitecore Connect can synchronize content into any template, thanks to a flexible mapping system.
For this POC purpose I have set up an SXA based page, and created a simple Article template which is based on the SXA's page template.
It has a Description field and Title and Content fields inherited from a `Page` template. 
I am going to map to those fields to corresponding properties in Content Hub.
The names of the fields do not have to match those from Content Hub because they will be mapped in the configuration.
The template can of course have as many other fields as needed which are independed from the Content Hub content.

<figure>
<img src="/assets/posts/sitecore-integration/13-sitecore-template.png" alt="Sitecore template structure to store the imported articles." />
<figcaption>Sitecore template structure to store the imported articles.</figcaption>
</figure>

To save articles from Content Hub, I will need a folder.
As part of the Sitecore Connect installation, Sitecore created the CMP item under the `sitecore/Content`, which is the suggested place to store all the synchronized content.
Sitecore advises using cloning feature to get the content to the target website, since the CMP folder is created outside of the website content tree.
The folder to store the imported content items is a bucket, so the template which is going to be used should be bucketable.

<figure>
<img src="/assets/posts/sitecore-integration/14-sitecore-folder.png" alt="Sitecore folder to store the imported articles." />
<figcaption>Sitecore folder to store the imported articles.</figcaption>
</figure>

The configuration of the import process is done in the `System/Modules` section, under the path `/sitecore/system/Modules/CMP/Config`.
Each entity type requires separate configuration item, to map entity properties to Sitecore fields.

First I have to create new `Entity Mapping` configuration item and fill its properties:
* Entity Type Schema - In case of the Content Types this value is the idenfier of the `M.ContentType` taxonomy used to describe the Content Type, otherwise it is the identifier of the entity which is going to be synchronized   

<figure>
<img src="/assets/posts/sitecore-integration/17-content-hub-content-type-taxonomy.png" alt="Content Type taxonomy values." />
<figcaption>Content Type taxonomy values.</figcaption>
</figure>

* Bucket - points to the location where the imported articles will be saved, usually under `sitecore/content/CMP` item.
* Template - is the Sitecore template used to create new item
* Item Name Property - tells the connector which entity property use as a Sitecore item name

<figure>
<img src="/assets/posts/sitecore-integration/18-sitecore-article-mapping.png" alt="Example Content Hub entity to Sitecore item mapping." />
<figcaption>Example Content Hub entity to Sitecore item mapping.</figcaption>
</figure>

Finally, the connector requires the properties of the entity to be mapped to Sitecore fields.
Each property will need separate mapping item created under the entity mapping.
The best way to get all the property names for particular content type, required to correctly configure the connector, is to create the example content type entity and view its json representation using the url `https://yourcontnethubdomain/api/entities/[EntityID]`.
Make sure you will also configure the default mapping for the `__Display name` field, otherwise the synchronization will not work.

<figure>
<img src="/assets/posts/sitecore-integration/15-content-hub-example-article.png" alt="Article entity standard view." />
<figcaption>Article entity standard view.</figcaption>
</figure>

<figure>
<img src="/assets/posts/sitecore-integration/16-content-hub-example-article-api.png" alt="Article entity API view." />
<figcaption>Article entity API view.</figcaption>
</figure>

This is not applicable to general entites, as the property names can be read directly from the entity definition.

<figure>
<img src="/assets/posts/sitecore-integration/19-sitecore-property-mapping.png" alt="Example Content Hub property to Sitecore field mapping." />
<figcaption>Example Content Hub property to Sitecore field mapping.</figcaption>
</figure>

## That's all folks!

Once all the fields are mapped the connection should be ready to go.
All that is left is to create and publish the appropriate content type entity in Content Hub and wait for it to be synchronized in your Sitecore instance.

For the end I have left couple of possible issues you might encounter and troubleshooting ideas in case something is not right.
* Actions in Content Hub have their own logs so you can easily check if the action you are interested in was even executed.
* Make sure you have tested if the connection between Content Hub and Azure Service Bus is working correctly.
* If the message reaches Sitecore, you are nearly home - even if anything is not configured properly, you will find it in the Sitecore logs.
* Make sure to use correct identifier, especially if you are about to demo this to someone :)
