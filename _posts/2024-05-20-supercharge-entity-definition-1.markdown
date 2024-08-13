---
layout: post
title:  "Supercharge your entity definition - pt 1. State flows"
date:   2024-08-13 07:00:0 +0100
categories: content-hub entity-definition
author: Szymon Kuzniak
---

Let's be honest.
Documentation is a great thing, but sometimes you just need to get your hands dirty and do some exercise, to actually understand the concept.
That's how I came up with the idea of this little blog post series.

You see, in Content Hub, in each entity definition, there are few switches ready to be enabled, and I have a weird impression they can be really useful.
I had an occasion to work with one of them - the **State Flows** - and I would love to have an article which would describe every aspect of it.
Hence, I'm going to write one about State Flows, but I want to also check what other switches do, to complete the series.
It will cover what's already in the documentation, but I'll also include some information that I found out by trial and error.

Let's go!

A bit of theory first.
If you go to your entity definition, and expand three dots menu in the top right corner, you will find *Enable/Disable* option as shown on the picture below.

<figure>
<img src="/assets/posts/supercharge-1/01-enable-disable-position.png" alt="The Enable/Disable menu option on the entity definition." />
<figcaption>The Enable/Disable menu option on the entity definition.</figcaption>
</figure>

This is where our magical switches are hiding.
Each option is briefly described, but it's far from thelling the whole story.
Each feature requires some amount of configuration and setting up to work properly.
And while everything is described in documentation, there are some things that really surprised me.

<figure>
<img src="/assets/posts/supercharge-1/02-the-switches.png" alt="The switches available to supercharge the entity definition." />
<figcaption>The switches available to supercharge the entity definition.</figcaption>
</figure>

To best illustrate what will be the effect of enabling the features, I needed a playground.

For the purpose of writing these articles, I have created a simple entity definition, which I'm going to gradually supercharge.
Initially, I created a page with bare list of the entities, on which I can do some basic operations such as delete or edit them.

<figure>
<img src="/assets/posts/supercharge-1/03-the-list-page.png" alt="The page with simple list of entities." />
<figcaption>The page with simple list of entities.</figcaption>
</figure>

I also had to create a simple details page, which is required when creating new entities and viewing the details of the existing ones.

<figure>
<img src="/assets/posts/supercharge-1/04-details-page.png" alt="The page with simple entity details." />
<figcaption>The page with simple entity details.</figcaption>
</figure>

I will use these pages to add components which will become available, as I will be enabling new features.

## State Flow

As I menioned, I decided to start with State flows feature.
It is actually just another name for a good old workflow.
It can be applied to any definition.

### Enabling State Flow

Before you enable State Flow, go to the entity settings and set a Display Template.
The Display Template is used to represent your entities instead of entity ID and usually uses one of the fields in the curly brackets.
Because it is a template, you can combine more fields and also add some text if needed.

<figure>
<img src="/assets/posts/supercharge-1/04.1-entity-display-template.jpg" alt="The Display Template." />
<figcaption>The Display Template.</figcaption>
</figure>

Once the Disaply Template is set up, go to *Enable/Disable* pop-up and switch the State flow switch.
There are three configuration options available.
<div class="alert-box alert-box--info hideit">
    <p>
        All three options here can be changed later.
        If you need to edit any of them, simply go the <em>Enable/Disable</em> pop-up and click on Edit button next to the enabled option.
    </p>
</div> 

**Enable assignees** - enables an option, to configure what to do with the assignees during the state flow transition.
When set to true, on every transition in the state flow configuration, there will be additional option **Update Asignees** available.

**Detail page** - this one is straightforward and well explained - it's a page used in notification email template.

**Fields** - allows you to add additional fields, for which one of three actions - **Keep**, **Clean** and **Overwrite** - can be selected in each transition. It works in similar way to **Enable assignees** but for other editable fields.

Eg.
Field *Name* is selected.

In every transition in the state flow there will be additional **Update Name** option available.
This option allows you to choose what happens with the *Name* field when product reaches the state.
<figure>
<img src="/assets/posts/supercharge-1/04.2-additional-fields.jpg" alt="An example of additional field in the state transition." />
<figcaption>An example of additional field in the state transition. There's also Update Assignees option visible.</figcaption>
</figure>

### What happens

Let's now see what actually happens when the feature is enabled.

Entity definition gets new relations:
* EntityNameToStateMachine
* EntityNameToAvailableRoute
* EntityNameToActiveState
* StateUserToEntityName
* AssignedUserToEntityName
* AssignedGroupToEntityName

In the operation component two operations are now available
* Assign state flows - allows to select one of the state flows compatible with current definition
<img src="/assets/posts/supercharge-1/05-assign-state-flows-button.png" alt="A screenshot of the application showing how does the assign state flows operation works." />
* State flows transitions - allows to use one of the existing transitions
<img src="/assets/posts/supercharge-1/06-state-flows-transition-button.png" alt="A screenshot of the application showing how does the state flows transition operation works." />

You will be able to create a state flow connected with this entity definition.

### Creating state flow

Once the entity is enabled to use state flows, a state flow has to be created.
State flows are bound to the entity definition and each state flow can only work with one entity definition.
It means you have to enable the feature for the entity definition before you can create state flow for it.

<div class="alert-box alert-box--notice hideit">
    <p>State flow will be deleted when the state flow feature is disabled on the entity definition.</p>
</div>


In order to create new state flow:

1. Go to the *State flows* page in the *Manage* section and click on a *+ State flow* button.
2. A target definition has to be selected for which the state flow can be used.
Only enabled definitions are listed, so there is no risk of selecting incompatible one.
3. On top of that, new state flow requires a name.
4. *Detail page* and *Email template* are optional and are quite well explained when hovering over the question mark.

#### States

State flow obviously needs states.
To create a state click on the *+ State* button on the state flow page.
It will open following dialog and will reqruie to provide some data for the state.

*Name* and *Description* - quite self explanatory

*Update assignees* - This option is only available when it was enabled when configuring the State flows featuch in the *Enable/Disable* pop-up.
It controls what happens to the assignees when the state is reached.
When Overwrite is selected it is possible to define either static users/groups by name or dynamic asignees/assignee groups.

Dynamic assignee groups is an interesting concept here, which is not explained clearly in the documentation.
I couldn't figure out how it works yet, but this is definitely on my TODO list.

#### Transitions

Transitions define how the state flow progresses from one state to another.
Transitions are created for each state, and state can have more than one transition. For example, state **In review** can either have **Accept** transition, which will progress to the next state, or **Reject** transition, which will move it back to one of the previous steps.

1. On the state flow page, click on the **Edit transitions** button on the state for which you want to add or edit transition.
<img src="/assets/posts/supercharge-1/15-new-transition.png" alt="Transitions can be added from the list of states." />
2. The state for which the transions are being edited can be found in the title of the dialog.
<img src="/assets/posts/supercharge-1/16-new-transition.png" alt="When editing the transition for a state, the state name is displayed as the title of the dialog." />
3. Click on *+ Transition* button.
<img src="/assets/posts/supercharge-1/17-new-transition.png" alt="Use the + transition button to create new transition." />
4. Provide a *Name*, choose an *Icon* and select *Next state* for the transition.
<img src="/assets/posts/supercharge-1/18-new-transition.png" alt="New transition require some basic data, such as name, icon and the next step." />
5. Save and repeat for other transitions.

### Using state flow

When state flows feature is enabled it will not have any immediate effect on the entities.
Old entities will remain unchanged and new entities will not get the state flow assigned automatically.

To ensure all new entities will get the state flow assigned, we will need an action and an appropriate trigger.

1. Go to the *Manage* page and click on *Actions*
2. Create new aciton
    1. Provide a descriptive name
    2. In the *Type* field select **Start state machine**
    3. Select appropriate *State flow*.
    <img src="/assets/posts/supercharge-1/10-new-action.png" alt="The action is needed to start the state machine for a given state flow." />
3. Go back to the *Manage* page and click on *Triggers*
4. Create new trigger
    1. Provide a descriptive name.
    2. The *Objective* set to **Entity creation**.
    3. *Execution type* set to **In process**.
    <img src="/assets/posts/supercharge-1/11-new-trigger.png" alt="The trigger requires some basic configuration." />
    4. On the *Conditions* tab, select your entity definition for which you want to start the state flow.
    <img src="/assets/posts/supercharge-1/12-new-trigger.png" alt="The trigger should only be triggered for the entity definition that is relevant for the state flow." />
    5. On the *Actions* tab, select the action created in step 2, in the *Post actions* group.
    <img src="/assets/posts/supercharge-1/13-new-trigger.png" alt="The trigger will trigger an action created in one of the previous steps." />
5. Save and close the trigger page and when asked to activate the trigger, select option to activate it.
<img src="/assets/posts/supercharge-1/14-new-trigger.png" alt="When the trigger is saved for the first time there is an option to activate it automatically." />


The buttons to progress the entity in the state flow have to added manually.
This can be done on the details page using *Operations* component or on the *Search* component when configuring the view.

## How about existing entities

Depending on how many items require updating this can be done in at least two ways:

* Update the relations using Excel
* Write a small application that will go over the entities and update them

This article grew quite a lot, so I will write a separate article just to cover this topic.

## State flows and languages

If you store your content in multiple languages, within the same entity, you have to remember that state flow transitions will affect all of them.
By only enabling state flows, it is not possible to have the same entity in different states for different languages.
It means that if the entity is approved, it is approved for all languages.
When it is rejected, it will be rejected for all languages.

In order to have separate state flows for each language, another switch needs to be enabled.
But this is a material for another blog post.

## Disabling state flows

Be careful when disabling state flows.
The message says you will loose the data and this is correct, however not everything will be cleared.
<img src="/assets/posts/supercharge-1/07-disable-state-flows.png" alt="A screenshot of the application showing the alert when disabling state flows feature." />
Few relations related to assignement will stay.
Also on the operations component the state flow related operations will not be removed.
The page with this component will be operational but will produce an error.
<img src="/assets/posts/supercharge-1/08-missing-state-flows-error.png" alt="A screenshot of the application showing the error about definion not enabled for state flows." />
<div class="alert-box alert-box--error">
    <p>You will however loose all state flow definitions that were related to the eneity definion that the state flow is being disabled.</p>
</div>

## Useful links

* [Sitecore Documentation about State flows](https://doc.sitecore.com/ch/en/users/content-hub/state-flows.html)

## Conclusion

Stateflows are really powerful and useful feature.
I hope this little article will help you with the configuration and getting started with it.