---
layout: post
title:  "Custom notifications in Sitecore Content Hub - Email template"
date:   2022-06-21 19:00:0 +0100
categories: content-hub notifications
author: Szymon Kuzniak
---

Sitecore Content Hub has a rich set of notifications complemented by a set of editable email templates.
They can be easily turned on and off depending on the channel - either email or internal push message.
However administrators are restricted only to this initial list and its base functionality.
Sometimes it might be necessary to notify users about events which are not available out of the box or use non-standard email template.

As with many Content Hub customizations, notifications can be extended using scripts and dedicated API.
Adding new notification type, for example to send email to only a group of users interested in particular group of products was pretty simple - trigger a script action when the product is updated and match the relation with user group to find out to whom send the message.

### Custom notifications in Sitecore Content Hub

This post is part of the series:

1. Create email template <= you are here
2. [Custom settings](/2022/07/10/custom-notifications-content-hub-settings)

Once I had the idea of how I can approach this task, I wanted to create an email template.
And this turned out to be not so easy as I initially assumed because... there is no such option.
Directly in CH it is possible to edit existing templates but not to create them.
Quick google search revealed an article on how this can be done and to my surprise it turned out that others are using API to create new email template.
Unfortunately the way shown in the blog post didn't work for me but using it as a cornerstone I was able to create the template I wanted.

Below is the snippet that does exactly that.


    // These are the names of the email template properties which can only be set from the code
    var fieldTemplateName = "M.Mailing.TemplateName";
    var fieldTemplateLabel = "M.Mailing.TemplateLabel";
    var fieldTemplateVariables = "M.Mailing.TemplateVariables";

    // This is the name of the template and a user friendly label
    var templateName = "CustomNotificationEmail";
    var templateLabel = "Custom Notification Email";

    // Label template value is dependant on the culture, so all available cultures will be required
    var cultures = await MClient.Cultures.GetAllCulturesCachedAsync();

    // Creation of the template
    var mailTemplate = await MClient.EntityFactory.CreateAsync(Stylelabs.M.Sdk.Constants.MailTemplate.DefinitionName);

    // Loading properties which will be used
    await mailTemplate.LoadPropertiesAsync(new PropertyLoadOption(fieldTemplateName));
    await mailTemplate.LoadPropertiesAsync(new PropertyLoadOption(fieldTemplateLabel));
    await mailTemplate.LoadPropertiesAsync(new PropertyLoadOption(fieldTemplateVariables));

    // Setting name of the template - it will be used to send emails
    mailTemplate.SetPropertyValue(fieldTemplateName, templateName);
    
    // Setting user friendly label in every culture
    foreach (var culture in cultures)
    {
        mailTemplate.SetPropertyValue(fieldTemplateLabel, culture, templateLabel);
    }

    // Template variables can only be passed as JArray object
    var templateVariables = @"[
        { 
            'Name': 'Variable_1',
            'VariableType': 'String',
            'Templated': 'true',
            'Template': '{variable1}'
        },
        { 
            'Name': 'Variable_1',
            'VariableType': 'String',
            'Templated': 'true',
            'Template': '{variable2}'
        },
    ]";
    var templateVariablesObject = JArray.Parse(templateVariables);
    mailTemplate.SetPropertyValue(fieldTemplateVariables, templateVariablesObject);

    // Save the template
    await MClient.Entities.SaveAsync(mailTemplate);


The code above will create a simple template and fill in all the properties which are not accessible from the UI.

Existing template can be retrieved using the notification API

    IMailTemplateEntity template = await MClient.Notifications.GetMailTemplateEntityAsync("TemplateName", 
                new EntityLoadConfiguration(CultureLoadOption.All, PropertyLoadOption.None, RelationLoadOption.None));

Once the template is awailable, its other properties, such as description, email subject or the body of the message can be edited directly from the Content Hub UI.

In the second post I will show how to make use of the template and match the relation to user groups using settings.

Stay tuned.