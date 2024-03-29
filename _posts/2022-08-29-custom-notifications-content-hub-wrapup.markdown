---
layout: post
title:  "Custom notifications in Sitecore Content Hub - Trigger and complete script"
date:   2022-08-29 15:00:0 +0100
categories: content-hub custom-notifications custom-settings
author: Szymon Kuzniak
---
This post is a third part of the series about creating custom notifications with Sitecore Content Hub.
The problem I am trying to address is triggering an email notification when the entity is updated, depending on the value of one of the relations.
In other words, if a product which belongs to "Health" department is modified, I want to send email notification only to members of groups related to healt.

### Custom notifications in Sitecore Content Hub

This post is part of the series:

1. [Create email template](/2022/06/21/custom-notifications-content-hub)
2. [Custom settings](/2022/07/10/custom-notifications-content-hub-settings)
3. Trigger and complete script <- you are here

Since I already have my custom email template, and the mapping between department names and the user groups stored in the settings, I can proceed to creating the trigger which will execute the entire script.
The trigger will run under certain conditions - in my case when the product entity is modified.
When it is run it will call an action which will execute the script that handles the notification logic.
This approach can be used to create other notifications which are not supported out of the box.

### The script

I am going to start with creating a script - I will need it to create an Action and the Trigger (remember this order if you have not passed the certification yet).
The script will first read the value of the relation which describes the department.
Based on this value and using script from [previous post about custom settings](/2022/07/10/custom-notifications-content-hub-settings) I will get the names of the user group that should be notified.
Once I have the groups, I will be able to get the ids of the users who belong to the groups.
Those ids, and the name of the template ([I have created one in the first blog post](/2022/06/21/custom-notifications-content-hub)) are needed for the notification client to send the email.
As the final touch, I will get some basic data from the modified entity to create the URL to it, and give notified users some basic information about it.

Let's see how the script looks like

    using Newtonsoft.Json.Linq;
    using Stylelabs.M.Sdk.Models.Notifications;
    using System.Linq;

    // let's start with some definitions

    var productEntityDefinitionName = "Demo.Product";
    var departmentRelationName = "Demo.DepartmentToProduct";

    var settingsGroupName = "Demo.Notifications";
    var settingName = "Demo.DepartmentToGroupMapping";
    var settingValuePropertyName = "M.Setting.Value";

    // Step 1. Get the name of the department from the relation on the modified product.
    MClient.Logger.Info("Step 1.");

    var entity = (IEntity)Context.Target;

    if (entity.DefinitionName != productEntityDefinitionName)
    {
        return;
    }

    await entity.LoadRelationsAsync(new RelationLoadOption(departmentRelationName));
    var relation = entity.GetRelationAsync<IChildToOneParentRelation>(departmentRelationName);
    var departmentId = relation.Result.Parent;

    if (!departmentId.HasValue)
    {
        return;
    }

    // Step 2. Get the name of the group, based on the mapping from the settings.
    MClient.Logger.Info("Step 2.");

    var groupsToNotifySetting = await MClient.Settings.GetSettingAsync(settingsGroupName, settingName).ConfigureAwait(false);
    var settingValue = groupsToNotifySetting?.GetProperty<ICultureInsensitiveProperty>(settingValuePropertyName);
    if (settingValue == null)
    {
        MClient.Logger.Error($"Unable to get {settingName} from the {settingsGroupName}.");
        return;
    }

    var settingJobjectValue = settingValue.GetValue<JObject>();
    if (settingJobjectValue == null)
    {
        MClient.Logger.Error($"Can't get {settingsGroupName} property value");
        return;
    }

    var settingGroupMappings = settingJobjectValue["groupMapping"];
    if (settingGroupMappings == null)
    {
        MClient.Logger.Error("Can't get JSON object 'groupMapping'");
        return;
    }

    var groupNameObject = settingGroupMappings.FirstOrDefault(m => m["department"]?.Value<string>() == $"{departmentId}");
    var groupName = groupNameObject == null ? string.Empty : groupNameObject["group"]?.Value<string>();

    if (string.IsNullOrEmpty(groupName))
    {
        return;
    }

    // Step 3. Send the email to the users who belong to the found group, using notifications client.
    MClient.Logger.Info("Step 3.");

    var groupId = MClient.Users.GetUserGroupIdAsync(groupName).Result;
    var userQuery = Query.CreateQuery(entities => from e in entities
                                                    where e.DefinitionName == "User"
                                                    && e.Parent("UserGroupToUser") == groupId.Value
                                                    select e);
    var users = MClient.Querying.QueryAsync(userQuery, new EntityLoadConfiguration { RelationLoadOption = RelationLoadOption.All }).Result;

    var emailRequest = new MailRequestById
    {
        Recipients = users.Items.Select(u => u.Id ?? 0).ToList(),
        MailTemplateName = "CustomTemplateName",
    };

    var productPath = $"PathTo/ProductDetail/{entity.Id}"; // use this to constuct link to the changed product

    // use variables to pass data from the script to the email template (variable has to be created when the template is created).
    emailRequest.Variables.Add("ProductName", entity.Identifier);
    emailRequest.Variables.Add("DateUpdated", DateTime.Now);
    emailRequest.Variables.Add("ProductPath", productPath);

    await MClient.Notifications.SendEmailNotificationAsync(emailRequest);

    MClient.Logger.Info("Sent.");

Once the script is ready and published, I can create an action and a trigger.
I will start with adding new trigger (yes, I remember I was writing about suggested order, but, once you pass the certification exam, you should know that actions can also be created from the trigger screen).
To create a trigger, I have to choose the name, select the events that will fire the trigger and choose its execution type.
For my trigger I am going to need it to react on modification and work in background.

<figure>
<img src="/assets/posts/content-hub-notifications-wrapup/create-trigger-1.png" alt="The General section of new trigger dialog." />
<figcaption>The General section of new trigger dialog.</figcaption>
</figure>

In the next section I have to select the entity type which modification should fire the trigger.
I am selecting the entity type representing my Product, using **Add Definition** button.
Here, I can also further narrow the amount of possible entities which will cause the trigger to fire, by selecting additional conditions.
For example I could configure my trigger to fire only for products in Final state of the State Flow.

<figure>
<img src="/assets/posts/content-hub-notifications-wrapup/create-trigger-2.png" alt="The Conditions section of new trigger dialog." />
<figcaption>The Conditions section of new trigger dialog.</figcaption>
</figure>

Finally, I am going to configure what should happen once the trigger fires - I need to select an action.
Since the action does not exist yet, I have to create it.
Action can be created by clicking on a **Manage Actions** button and then **New Action**.
Obviously it is possible to create an action from the Actions section on the Manage screen.
Again I need to name the action, choose the action type - in this case Action Script and select previously created script which will be executed and responsible for sending the notification.

<figure>
<img src="/assets/posts/content-hub-notifications-wrapup/create-trigger-action.png" alt="The Manage Actions section of new trigger dialog." />
<figcaption>The Manage Actions section of new trigger dialog.</figcaption>
</figure>

When using this way of creating action and trigger it is vital to remember that previous step only created the action, but did not assign it with the trigger.
This has to be done manually using **Add Action** button.

<figure>
<img src="/assets/posts/content-hub-notifications-wrapup/create-trigger-3.png" alt="The Actions section of new trigger dialog." />
<figcaption>The Actions section of new trigger dialog.</figcaption>
</figure>

Once everything is tied up, it's time to test the trigger by modifying the product, and verifying whehter the email was send to the correct group of users.
This can be easyily done using gmail account, which supports multiple email addresses using '+' character.
I will create two users with email address related to the group I want to test:

* username+health@gmail.com
* username+fitness@gmail.com

Now I will create two user groups: NotificationHealth and NotificationFitness and assign the users accordingly.
Once the user groups are created, I'm going to create the mapping in the settings.
And finally I can make some changes to the product from Health department and wait patiently for the email with notification.

Happy coding!
