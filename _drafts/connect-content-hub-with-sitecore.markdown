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

1. **Create new Service Bus subscription**  
In order to setup Azure Service Bus to work with Sitecore connector, you will need Topics.
This is important when choosing the pricing tier - it has to be at least Standard, Basic subscription will not have Topics enabled.

2. **Create the connection strings**  
General connection string will be required for setting up the action in the Content Hub.

3. **Create topics**  
Content Hub will use one topic to write messages to every time content should be synchronized with Sitecore.
Another one will be required for Sitecore Connector to report status of the synchronization.  

4. **Create subscription**
Sitecore Connector requires subscription on the topic which will be used to notify about new content.

5. **Create topic connection strings**  
Those connection strings are used to configure Sitecore Connector, so it knows where to get the messages from, and when to send the status.

## Setting up Content Hub

