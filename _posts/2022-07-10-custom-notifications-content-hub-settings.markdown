---
layout: post
title:  "Custom notifications in Sitecore Content Hub - Custom settings"
date:   2022-07-10 15:00:0 +0100
categories: content-hub custom-notifications custom-settings
author: Szymon Kuzniak
---
This post is a second part of series about creating custom notifications with Sitecore Content Hub.
The problem I am trying to address is triggering an email notification when the entity is updated, depending on the value of one of the relations.
In other words if a product which belongs to "Health" department is modified, I want to send email notification only to members of groups related to healt.

### Custom notifications in Sitecore Content Hub

This post is part of the series:

1. [Create email template](/2022/06/21/custom-notifications-content-hub)
2. Custom settings <= you are here

In this part I am going to use Content Hub settings to map user groups to the relation values.
This will allow me to easily manage the mapping and have different settings depending on the environment.

As with every configuration inside Content Hub I will start with the Manage section, and open the Settings page.
In order to add new settings I need to click on the **+Setting** button in the top right corner.
The setting configuration is straightforward: I need to choose a name and a category and optionally provide a label.
If there is no suitable category, it is possible to create a new one.

<figure>
<img src="/assets/posts/content-hub-notifications-settings/add-new-setting.png" alt="The add new setting dialog." />
<figcaption>The add new setting dialog.</figcaption>
</figure>

The name and the category will be necessary to access the setting from the code.
For my custom notification I have created a category "Demo" and the setting "Demo.UserGroupMapping".

The settings within Content Hub are stored in form of JSON.
In order to create a mapping I will use following configuration:

    {
        "mapping": [
            {
                "department": "health",
                "groups": [
                    {
                        "group": "D.Health"
                    },
                    {
                        "group": "D.HealthManagers"
                    }
                ]
            },
            {
                "department": "wellbeing",
                "groups": [
                    {
                        "group": "D.YogaMasters"
                    }
                ]
            }
        ]
    }


<figure>
<img src="/assets/posts/content-hub-notifications-settings/the-setting-view.png" alt="The setting view." />
<figcaption>The setting view.</figcaption>
</figure>

The JSON setting value can be created using build-in JSON editor as on the above image.
It is also possible to use the text editor to either create the configuration in text format or paste it from any other text editor.
It is worth noting that build-in editor offers validation of the entered JSON.

The snippet above will create a new JSON object called mapping.
The mapping contains of an array of objects, each consiting of a department name and an array of groups.
The department name will be used to match the department set on the product and the groups will be used to find the users to whom the emails should be sent.

The code below will allow me to access the settings:

    var groupsToNotifySetting = await MClient.Settings.GetSettingAsync("Demo", "Demo.UserGroupMapping").ConfigureAwait(false);
    var settingValue = groupsToNotifySetting?.GetProperty<ICultureInsensitiveProperty>("M.Setting.Value");
    if (settingValue == null)
    {
        MClient.Logger.Error("Unable to get 'Demo.UserGroupMapping' from the 'Demo' group setting.");
        return;
    }

    var settingJobjectValue = settingValue.GetValue<JObject>();
    if (settingJobjectValue == null)
    {
        MClient.Logger.Error("Can't get 'Demo.UserGroupMapping' property value");
        return;
    }

    var settingGroupMappings = settingJobjectValue["mapping"];
    if (settingGroupMappings == null)
    {
        MClient.Logger.Error("Can't get JSON object 'mapping'");
        return;
    }

Let's analyze the important lines.

First I am loading the setting, based on the setting category and the setting name.

    await MClient.Settings.GetSettingAsync("Demo", "Demo.UserGroupMapping").ConfigureAwait(false)

The setting value is stored in the property "M.Setting.Value", so in order to access it I need to get the property.
Notice the property is culture insensitive.

    groupsToNotifySetting?.GetProperty<ICultureInsensitiveProperty>("M.Setting.Value")

Next, I have to get the value of the property as JObject.
In order to work with JSON I have to import Newtonsoft.Json.Linq namespace.

    settingValue.GetValue<JObject>()


Finally I am accessing the array of mappings by getting the root level object value.

    settingJobjectValue["mapping"]

The properties can be accessed using the Newtonsoft API.

    foreach(var groupMapping in settingGroupMappings)
    {
        // access the department name using: groupMapping["department"]
        foreach (var group in groupMapping["groups"])
        {
            if (group != null)
            {
                // access the group name using: group["group"]
            }
        }
    }

Once the mapping is set up it is possible to get the user IDs based on the value of the product relation, and send them notification.
I will describe the complete script code and the process in the final part of the series.