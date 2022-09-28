---
layout: post
title:  "Custom notifications in Sitecore Content Hub - Saved Selection"
date:   2022-09-28 13:00:0 +0100
categories: content-hub custom-notifications custom-settings
author: Szymon Kuzniak
---
This post is a fourth part of the series about creating custom notifications with Sitecore Content Hub.
This time I wanted to share a script which will send the notification when the user shares a saved selection with another user.

A saved selection can be shared with users and groups, however there is no notification being sent in that case.
Using the technique described in the previous posts, it is possible to create a trigger and an email template which can be used to notify users about the Saved Selection shared with them.

### Custom notifications in Sitecore Content Hub

This post is part of the series:

1. [Create email template](/2022/06/21/custom-notifications-content-hub)
2. [Custom settings](/2022/07/10/custom-notifications-content-hub-settings)
3. [Trigger and complete script](/2022/08/29/custom-notifications-content-hub-wrapup)
4. Saved Selection script <- you are here

This task turned out to be slightly more complicated than I initially though.
The main problem I had, is that the saved selection is some sort of a search and I cannot generate a link that I could use in the email template.
Right now I am just returning link to the assets page with intention to update the script, and this blog post, once I find out how to generate the link.
Additionally, the M.SavedSelection is not listed neither on Schema or Taxonomy or Entities list, so I had to look up it's properties and relations in the developer tools by analyzing the browser requests.

### The script

The script is pretty straightforward.
First I do some validation, whether the entity type is correct.
Then I need to get the list of users that should be notified - I can do this by getting the value of two relations of the M.SavedSelection entity: **SavedSelectionToUser** and **SavedSelectionToUserGroup**.
Finally I am going to evaluate the name of the user who created the saved selection so I can put it in the email.

Let's see how the script looks like

    using Stylelabs.M.Framework.Essentials.LoadConfigurations;
    using Stylelabs.M.Framework.Essentials.LoadOptions;
    using Stylelabs.M.Sdk.Contracts.Base;
    using Stylelabs.M.Sdk.Models.Notifications;

    var productEntityDefinitionName = "M.SavedSelection";
    var usersRelationName = "SavedSelectionToUser";
    var groupsRelationName = "SavedSelectionToUserGroup";

    var entity = (IEntity)Context.Target;

    if (entity.DefinitionName != productEntityDefinitionName)
    {
        return;
    }

    // Get the users mentioned directly
    await entity.LoadRelationsAsync(new RelationLoadOption(usersRelationName));
    var usersRelation = entity.GetRelationAsync<IParentToManyChildrenRelation>(usersRelationName);
    List<long> users = usersRelation.Result.Children.ToList();

    // Get the users from the groups
    await entity.LoadRelationsAsync(new RelationLoadOption(groupsRelationName));
    var groupsRelation = entity.GetRelationAsync<IParentToManyChildrenRelation>(groupsRelationName);
    var groups = groupsRelation.Result.Children;
    foreach (var userGroup in groups)
    {
        var userQuery = Query.CreateQuery(entities => from e in entities
                                                        where e.DefinitionName == "User"
                                                        && e.Parent("UserGroupToUser") == userGroup // Couldn't come up with the better idea - .In is not supported
                                                        select e);
        var usersFromGroups = MClient.Querying.QueryAsync(userQuery, new EntityLoadConfiguration { RelationLoadOption = RelationLoadOption.All }).Result;
        users.AddRange(usersFromGroups.Items.Select(u => u.Id ?? 0));
    }

    // make sure the users are on the list only once
    var distinctUsers = users.Distinct().ToList();

    // prepare data to be sent to the template
    var savedSelectionLink = "/en-us/assets-module/assets"; // this is tricky stuff, couldn't find out how to get a link to saved selection
    var sharingUser = entity.CreatedBy.HasValue ? MClient.Users.GetUsernameAsync(entity.CreatedBy.Value).Result : string.Empty;

    // create and send the notification
    var emailRequest = new MailRequestById
    {
        Recipients = distinctUsers,
        MailTemplateName = "SavedSelectionSharedNotification",
    };
    emailRequest.Variables.Add("SelectionLink", savedSelectionLink);
    emailRequest.Variables.Add("SelectionAuthor", author);

    await MClient.Notifications.SendEmailNotificationAsync(emailRequest);

This is it for now.
I really hope I will be able to update this page soon with the way to get the link to the saved selection.

Happy coding!
