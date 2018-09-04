---
id: message-board-tutorial
title: Message Board Tutorial
sidebar_label: Message Board Tutorial
---

Learn the basics with simple steps through this simple Message Board app development in Shift.
<br><br>
## The Plan
To start our project we need a plan.
<br><br>
__What we want to see?__
 - __Messages Dashboard__ - The default path for our application. This is where all the submitted messages will be shown.
 - __Message__ - A Message view where we will have an individual message shown.
 - __Message Form__ - A simple form for Message submission.
<br><br>

__What is a Message in this App context?__

A __Message__ will have the following properties:
 - `id` (unique)
 - `title`
 - `description`
 - `date`
<br><br>

__What we want to happen?__
- Messages to be shown at the Message Dashboard.
- Messages to be submitted through Message Form.
- Store the Messages in a db (Graphcool used in this tutorial as the endpoint).
- If a Message Title is clicked, redirect to the individual Message.
<br><br>

__When we want it?__
- Yesterday.
<br><br>
## Create a new Project
In Dashboard you can simply create a new project by giving it a name.

![Create Project](/docs/assets/firsttutorial.gif)
<br><br>
## Structure
Let's start by structuring our App.
First we will need to add the [__Router__](router.md) component to handle all the different paths. For default the component [__Router__](router.md) has two [__Route__](route.md) components as __defaultChildren__. > You can delete the default children and for each path add new __Route__ components.

Now we configure each [__Route__](route.md) component. On the __component editor__ tab, under the Settings header we set the `Label`, `Path` and `Default path` settings for each [__Route__]: 

| Label       | Path                | Default Path     | Exact Path    |
|-------------|---------------------|------------------|---------------|
| Home        | /                   |                  | tick          |
| Message     | /message/:messageid | /message/default |               |
| PostMessage | /newMessage         |                  |               |

> We will be using the messageid as a parameter to fetch and show each individual message.

Starting with the __Home Route__ we will add a [__Query__](query.md) component to get the data from our Graphql DB endpoint. Then to it we add a [__ReplicateList__](replicatelist.md) which will ease our job by replicating, with the help of it's parent, each instance of data (__Message__) received. 
<br><br>
For each __Message__ we will want to show its __Title__ (which will be a link to the individual message) and __Date__, so we add the following components:

![Home](/docs/assets/firsttutorial_home.png)

For the __Message Route__ we will want also a [__Query__](query.md) component to fetch our __Message__ and each component to display its previous data and the __Description__ aswell:

![Message](/docs/assets/firsttutorial_message.png)

At last the __PostMessage Route__ as our form to submit new Messages, will have a [__LocalState__](state.md) component with a child [__Form__](form.md) and 
this will have the following components for input:

![PostMessage](/docs/assets/firsttutorial_postmessage.png)

So to wrap our structure, this is all we will need.
<br><br>
## Data
Someone at the back-end will have to take care of the DB. Well, we can be that someone.

For this tutorial we will use the __GraphCool__ interface.  Go to [__Graph.cool__](https://www.graph.cool) and create an account.  Login to your account, create a new project called 'Messages' and go to the Graphcool console. It should look like this (insert image).

Ignoring the file and user `Types`, add a new type 'Message'. You will notice that Graphcool automatically gives 'Message' an unique id field. Add the fields: date, description and title.  Your type should look like this:

```
type Message @model {
  date: DateTime!
  description: String!
  id: ID! @isUnique
  title: String!
}
```

Now we need to connect this to your app in Shift.  To do this, copy the simple API endpoint url under endpoints in Graphcool and paste this into the __GraphQL Settings__ under the __Project Configuration__ tab in __Shift__. Remember to save. 
<br>

Our connection is ready. Now we need two [__Queries__](queries.md) and one [__Mutation__](mutations.md).

- query __allMessages__ - all the messages to be shown in the Messages feed.
- query __message__ - the individual message fetched using it's `id`.
- mutation __postMessage__ - post messages.

>In __Code__ you can not only add this queries and mutation but also use the [__GraphiQL__](https://github.com/graphql/graphiql) interface to do it. All this inside __Shift__.

Under the __Code__ tab, right click on files and add three separate graphql query files. Your file structure should look like: (insert image here).  Press enter to save the file and remember to use the .gql extension on each file.

Insert the below queries and mutation into their respective files.

```
query allMessages {
  allMessages {
    id
    title
    description
    date
  }
}
```

```
query message ($id: ID!) {
  Message (id: $id){
    id
    title
    description
    date
  }
}
```

```
mutation addMessage (
	$description: String!
	$date: DateTime!
	$title: String!){
    createMessage(
      description: $description
      date: $date
      title: $title) {
			id
            title
            description
            date
    }
  }
```

> Each __Query__ and __Mutation__ needs to be added separately.

You can check that these are all working correctly using the GraphiQL platform. For the message query and addMessage mutation you will need to provide the query variables.    

Another step complete, it's time to create data.
<br><br>
## Content

Our app runs on data, receives and submits __Messages__, so let's link it all together. <br> 

In the GraphiQL interface we can "manually" submit messages to the database. To do this opening the previously created `addMessage` mutation from the AddMessage.gql file. At the bottom in `Query Variables` in a json format
we add:

```
{
  "title": "A Pretty Message Title",
  "description": "a random description",
  "date": "2018-05-03"
}
```
> In a first instance we insert the date for the messages submitted manually, but later on this will be done automatically by our app.

From just hitting the play button at the top of the GraphiQL interface we can see that our Mutation query went through and we have the returning information as well as the id generated for our message.

Now we have messages in our DB, it's time to see it.

### Home

Go to the Route we created as __Home__ and select the previously created __Query__ component. On the __Query__ settings we set the `Query` option to __allMessages__ (our `Query` defined in Code).

In the settings for the __ReplicateList__ component (under the __allMessages__ query), set the `Item Name` to __message__ and `Each Item Key` to __message.id__ so it uses the message id as key for each data item. Then set the `Data` by clicking on `Data` --> `Bind values` --> `Value` --> click on the drop down menu and scroll down to __allMessages__. When you select __allMessages__ you should see some text appear on your page.

(Insert image 14 here)

You will also see that these messages have been added as props by going to the __Props and Binding__ tab. Remember that these are the props passing to the __ReplicateList__ component and therefore these will only show if you view __Props and Binding__ with that component selected. 

(Insert image 15 here)

Now to set the title and date for each message we select the __Text__ child of __ReplicateList__, go to the component settings then set the `Content` by clicking on `Content` --> `Bind values` --> `Value` --> click on the drop down menu and scroll down to __message__. Open __message__ and select __title__. When you select __title__ each message title should now appear on your page.

(Insert image 16 here)

Go ahead and do the same for the __Date__ child of __ReplicateList__ except, instead of `Content` in the settings, select `Date` and set the `Value` to  __message__, __date__.

(Insert image 17 here)

In our plan there was a link that would take us to each individual message, and since we forgot to add it on purpose, let's see how to quickly shortcut and deal with it in Shift.

Add a __Link__ component as a child of __ReplicateList__ and then drag the __Text__ component (which is our message title) into  it. You will need to delete the text component that apears by default when you add a __Link__ component.

(Insert image 18 here)

In the __Link__ configuration settings we set the link path by clicking `Link` --> `Bind values` --> `Expression` and inserting the string `/message/${this.props.item.id}`. This will set the link path to our __Message__ Route with the message `id` from the __message__ previously received from the __Query__ and fed from the __ReplicateList__.

(Insert image 19 here)

>As you have noticed the Component-Tree works with Components and Html elements respecting the order of the well known DOM Tree. That's why the __Link__ is added as a parent of __Text__ and not the other way around.


### Message

We are routing every message by its `id` as a parameter defined in this __Message__ Route (`/message/:messageid`). Now we need to tell our __Message__ Route __Query__ to deal with it.

In the __Query__ configuration settings set the `Query` option as __message__ and for the `Arguments` create a module, open in a new tab and set the 'return' to:

```
{
    id: this.props.route.params.messageid
}
```

"__Query__ deal with it !" we said.

Then simply bind the 'Content' and 'Date' in the settings for __title__, __description__ and __date__ like we did before. For a reminder go to each component's settings and set the `Content` or `Date` by clicking `Bind values` --> `Value` --> and scrolling the drop down menu until you find the field you would like to bind.

(Insert image 20 here)

>__Tip:__ go to the __Home__ Route and start the preview, see how each link takes you to each individual message path and presentation.

We are done here. How quick was it?

### PostMessage

For the __PostMessage__ Route we will need to configure the __Local State__ component, and the __Actions__ for it.

In our __Local State__ component settings we set the `Default State` by creating a module, opening in a new tab and setting the return to:

```
{
  title: "",
  description: "" 
}
```
Remember to save your changes to this file (ctrl s)

(Insert image 21 here)

To add __Actions__ to your __Local State__ simply click 'Add action' under the Actions in __Local State__ settings. This will prompt you for the 'New action name'. Add the following Action for every change in the target values.

New action name: changeField
Template: Synchronous state change

Remember to hit 'enter' to save your changes.

(Insert image 22 here)

Click on the action to open it in a new tab.  Insert the following code and save.

```
export default function (event) {
  this.setLocalState(Object.assign({}, this.localState, {
    [event.target.name]: event.target.value
  }));
}
```

Add this Action for submitting the data gathered from the __Form__, setting the state back to it's default and sending the user to the newly created message. You will see that the __date__ property is added to the variables fetched from the inputs.

Name: onSubmit
Type: Do Mutation

```
import MUTATION from 'shift-files/addMessage.gql';
import { mutate } from 'shift-apollo-client';
import shiftHistory from 'shift-history';
import query from 'shift-files/allMessages.gql';

export default async function () {
  this.setLocalState(Object.assign({}, this.localState, {
    loading: true,
  }));
  
  try {

    this.localState.date = new Date().toISOString()
    
    const response = await mutate({
      mutation: MUTATION,
      variables: this.localState,
      // update list of messages on home page
      update: function (store, { data: { createMessage } }) {
        
        try {
          
          const data = store.readQuery({ query });
          data.allMessages.push(createMessage);
          store.writeQuery({ query, data })

        } catch (e) {
          this.setLocalState(Object.assign({}, this.localState, {
            error: e,
            loading: false,
          }));
        }
      }
    });

    this.setLocalState(Object.assign({}, this.localState, {
      title: "",
      description: "",
      error: null,
      loading: false,
    }));

    // change path to the newly created message
    shiftHistory.push(`/message/${response.data.createMessage.id}`)

  } catch (e) {
    this.setLocalState(Object.assign({}, this.localState, {
      error: e,
      loading: false,
    }));
  }
};

```

Now we need to link up the __Form__ child components with the above actions.  To do this we go to each __InputText__ configuration `Events` and add an 'onChange' event.  Insert the following code into the `Event Handler` which will open into a new tab.

```
export default function eventHandler(event) {
  event.preventDefault();
  //-- Event Handler
  this.props.changeField(event);
}
```
(Insert image 23 here)

Now do the same to the __InputButton__ by going to `Events`, adding an 'onClick' event and inserting the following code into the `Event Handler` which will open into a new tab.  

```
export default function eventHandler(event) {
  event.preventDefault();
  //-- Event Handler
  this.props.onSubmit(event);
}
```
For the button input we also need to set the `Value` and `Type` as __Submit__. // Ask Julian if this is actually necessary

For the final step, we need to bind the __title__ and __description__ input fields. To do this we go to the configuration settings for each __InputText__ field and set the `Name` and `Value` options as `title` and `description` respectively.

(Insert image 24 here)

__It still doesn't look pretty, but it works! Congratulations!__
<br><br>