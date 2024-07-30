---
layout: post
title:  "Supercharge your entity definition - pt 1. State flows"
date:   2024-05-09 07:00:0 +0100
categories: content-hub entity-definition
author: Szymon Kuzniak
---

Let's be honest.
Documentation is a great thing, but sometimes you just need to get your hands dirty and do some exercise to actually understand the concept.
That's how I came up with the idea of this little blog post series.

You see, there are few switches ready to be enabled on the entity definition page, and I have a weird impression they can be really useful.
I had an occasion to work with one of them - the State Flow - and I would love to have an article which would describe every aspect of the feature I was enabling.
Hence, I'm going to write one about State Flows, but I want to also check what other switches do to complete the series.

Let's go!

A bit of theory first.
If you go to your entity definition, and expand three dots menu at the top right corner, you will find Enable/Disable option as shown on the picture below.

<figure>
<img src="/assets/posts/supercharge-1/01-enable-disable-position.png" alt="The Enable/Disable menu option on the entity definition." />
<figcaption>The Enable/Disable menu option on the entity definition.</figcaption>
</figure>

This is where our magical switches are hiding.
Each option is briefly described, but it doesn't tell everything.

<figure>
<img src="/assets/posts/supercharge-1/01-the-switches.png" alt="The switches available to supercharge the entity definition." />
<figcaption>The switches available to supercharge the entity definition.</figcaption>
</figure>

For the purpose of writing those articles, I have created a simple entity definition, which I'm going to supercharge with various features.
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

I will use those pages to add components which will become available as I will be enabling new features.

## State Flow

The State Flow is just a fancy name for a good old workflow.
It can be applied to any definition.

### Enabling State Flow

Before you enable State Flow, go to the entity settings and set a Display Template.
The Display Template is used to represent your entities instead of entity ID and usually uses one of the fields in the curly brackets.
But it is a template so you can also add some text if needed.

<figure>
<img src="/assets/posts/supercharge-1/04-details-page.png" alt="The Display Template." />
<figcaption>The Display Template.</figcaption>
</figure>

Once the Disaply Template is set up, go to Enable/Disable page and switch the State flow switch.
There are three configuration options available, **all of them can be changed later**.

**Enable assignees** - this one will enable option to keep or update assignees during the state flow transition.

**Detail page** - this one is straightforward - a page used in notification email template.

**Fields** - this one allows you to choose one of the three options for selected properties and relations - **Keep**, **Clean** and **Overwrite** in each state.
The action will be applied on the transition to a given state.

Eg.
* Field *Name* is selected.
* On the state **Review** the action is set to **Clean**
* On the state **Final** the action is set to **Keep**
* Every time the entity will reach state **Review** field *Name* will be, surprise, surprise, cleaned.
* Every time the entity will reach state **Final** the field *Name* will not change.

This might be useful for helper fields such as comments, that can have different values throughout the entire state flow.

### What happens

* Entity definition gets new relations:
    * EntityNameToStateMachine
    * EntityNameToAvailableRoute
    * EntityNameToActiveState
    * StateUserToEntityName
    * AssignedUserToEntityName
    * AssignedGroupToEntityName
* In the operation component two operations are now available
    * Assign state flows - allows to select one of the state flows compatible with current definition
    <img src="/assets/posts/supercharge-1/05-assign-state-flows-button.png" alt="A screenshot of the application showing how does the assign state flows operation works." />
    * State flows transitions - allows to use one of the existing transitions
    <img src="/assets/posts/supercharge-1/06-state-flows-transition-button.png" alt="A screenshot of the application showing how does the state flows transition operation works." />

### Creating state flow

Once the entity is enabled to use state flows, a state flow has to be created.
State flows are bound to the entity definition and each state flow can only work with one entity definition.
**State flow will be deleted when the state flow feature is disabled on the entity definition.**

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

*Update assignees* - controls what happens to the assignees when the state is reached.
When Overwrite is selected it is possible to define either static users/groups by name or dynamic asignees/assignee groups.
Dynamic assignee groups is an interesting concept here, which is not explained clearly in the documentation.

TODO

First state on the list is treated as an initial state.s

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
Old entities will remain unchanged and new entities will not get the stateflow assigned automatically.

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

#### Operations component on the details page


### Imporant remarks

### Assign default state flow



Additionally the state flow is not assigned to new entities by default.

In order to fully configure state flow for the entity complete the following steps:

1. Enable the feature

#### Disabling state flows

Be careful when disabling state flows.
The message says you will loose the data and this is correct, however not everything will be cleared.
<img src="/assets/posts/supercharge-1/07-disable-state-flows.png" alt="A screenshot of the application showing the alert when disabling state flows feature." />
Few relations related to assignement will stay.
Also on the operations component the state flow related operations will not be removed.
The page with this component will be operational but will produce an error.
<img src="/assets/posts/supercharge-1/08-missing-state-flows-error.png" alt="A screenshot of the application showing the error about definion not enabled for state flows." />
You will however loose all state flow definitions that were related to the eneity definion that the state flow is being disabled.