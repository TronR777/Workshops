# Cognitive: Basics
Build a Web Chatbot Application utilising the Watson Conversation API, and then connect it to a third party chat service (Slack).

## Requirements
- Bluemix account
- Slack account

## Agenda
##### Part 1: Create a Web UI chatbot using Watson Conversation & Node-RED
- Setup a Conversation instance
- Create intents and entities
- Create a dialog tree utilising your intents and entities
- Setup a web UI and a Node-RED orchestration app

##### Part 2: Create a Slack chatbot
- Create a Slack bot
- Install Slack nodes and setup a Node-RED app
- Create request and response routing to your Conversation service

## Creating Conversation instance
In this section we are going to create a [Conversation](https://www.ibm.com/watson/developercloud/conversation.html) with a basic dialog structure to power our bot.
1. Log into Bluemix and create a Watson Conversation service (if you don't already have one).
  - click on `Create Service`
  - search for and select `Conversation`
  - create the service with a unique name

2. Launch the Watson Conversation tool by clicking on `Launch tool`.

3. Create a new `Workspace`, then we'll go on to create `Intents`, `Entities` and `Dialogs` - if you need help beyond what you find in this lab you can use [this documentation](https://console.bluemix.net/docs/services/conversation/getting-started.html#gettingstarted).

5. Create `Intents`. An intent represents the _purpose_ of a user's input. By recognising the intent expressed by a user, the Conversation service can choose the correct dialog flow to use to respond to it. To plan the intents for your application, you need to consider what your customers might want to do, and what you want your application to be able to handle.
Choosing the correct intent for a user's input is the first step in providing a useful response. The intents you identify for your application will determine the dialog flows you need to create; they also might determine which back-end systems your application needs to integrate with in order to complete customer requests (such as customer databases or payment-processing systems). You should use several examples - at least nine or ten - for each intent you create in Watson Conversation.
For our chatbot, we'll start with the following intents:
  - **#positive** for expressing positive opinion about mobile phones ... e.g.

  ![intent1](./images/conversation-intent1.jpg)
  - **#negative** for expressing negative opinion ... you should build this to look pretty much the inverse of your `#positive` intent
  - **#newphone** for expressing intent to get advice about buying a new phone
    - e.g. I'm looking for a new phone, I need advice on phones, I want to replace my mobile phone, What's the best phone available, ...
  - **#greeting** to capture greetings such as Hi, Hello, ...

6. Create `Entities`. An entity represents a term or object in the user's input that provides _context_ for a particular intent. If intents represent _verbs_ (something a user wants to do), entities represent _nouns_ (such as the object of, or the context for, an action). Entities make it possible for a single intent to represent multiple specific actions. An entity defines a class of objects, with specific values representing the possible objects in that class.
In our case we want to create the following entities:
 - **@brand** with values _Samsung_, _Apple_ and _HTC_ (feel free to include more)
 - **@parameter** with values '_battery_' and '_style_'

7. Create a `Dialog` tree. A dialog uses the intents and entities that are identified in the user's input, plus context from the application, to interact with the user and ultimately provide a useful response. Our dialog tree should help the user choose a new mobile phone based on an existing preference or a required characteristic.
  - First, create/edit `Welcome`, `Help` and `Anything Else` nodes. These nodes are used to initialise the dialog with the user, and catch any user input that we don't have a specific response for. The `Welcome` and `Anything Else` nodes are usually generated when you create a new workspace.

  ![welcome](./images/conversation-dialog-welcome.jpg)

  ![help](./images/conversation-dialog-help.jpg)

  ![else](./images/conversation-dialog-anythingelse.jpg)

  - Now construct your `New phone` advisor dialog tree. Here we will look for positive or negative user responses, based on our previously built `intents`, as well as picking up either a brand name or a characteristic defined by our `entities`.  Our dialog response will then reflect these user inputs.

  ![newphone1](./images/conversation-dialog-newphone1.jpg)

  ![newphone2](./images/conversation-dialog-newphone2.jpg)

  ![brandpos](./images/conversation-dialog-brandpositive.jpg)

  ![brandneg](./images/conversation-dialog-brandnegative.jpg)

  ![parampos](./images/conversation-dialog-parameterpositive.jpg)

  ![newelse](./images/conversation-dialog-newphoneelse.jpg)

7. If you prefer, you can download the whole Workspace [here](./Conversation/basic-workspace.json).
  - you can import a Workspace by clicking on the ![icon](./images/conversation-import.png) icon next to the `Create` button in Workspaces

8. Test the Dialog inside the Conversation tool and make sure that you try all branches of the dialog tree:
![testing dialog](./images/conversation-dialog3.jpg)

9. Finally, go back to the Watson Conversation Workspaces menu, click on the three dots for the Workspace you created, `View Details` and copy and save the Workspace ID. You'll need this for your Node-RED flow.

![dots](./images/conversation-workspaceID.jpg)

## Creating a UI for the Conversation app using Node-RED
In this section we are going to create a UI for our chatbot using Node-RED.

If you are new to Node-RED then take a look at [this tutorial](https://nodered.org/docs/getting-started/first-flow) to get started.

1. Log into Bluemix and if you don't already have one, create a Node-RED Boilerplate Application.
  - click on `Create App`
  - search for and select `Node-RED Starter`
  - create a new application with a unique name

2. After creating the application head over to the `Connections` tab and click on `Connect existing`.

3. Select and connect the Conversation instance we've just created, and restage your Node-RED app. Bluemix will ask you if you want to do this when you add a new connection, or you can do it any time by issuing the `cf restage my-app` command from a terminal window, where _my-app_ is your Bluemix Node-RED application name.

4. After restaging, go to the app's URL and go through the initialisation process.

5. Go to the Node-RED flow editor.

6. We are going to create two flows. One for displaying a basic web conversation UI, and another for communicating with the Conversation.

7. For the first flow, create:
  - `http in` node with method `GET` and URL `/bot`
  - `template` node with the HTML content from [here](./Node-RED/template.html)
  - `http response` node

8. Connect them together as shown here: ![flow template](./images/node-red-bot-template.jpg)

9. For the second flow create:
 - `http in` node with method `POST` and URL `/botchat`
 - `function` node with the following code, which takes the input from the web UI and formats it for passing to Watson Conversation:
```javascript
msg.params = msg.params || {};
msg.params.context = msg.payload.context;
msg.payload = msg.payload.message;
return msg;
```
 - `Conversation` node with Workspace ID of your workspace. You retrieved this previously by selecting `View Details` from your Conversation Workspace.
 - `http response` node

10. Connect them together, `deploy` them in Node-RED and go to your newly constructed UI at: `your-website.mybluemix.net/bot`
![flow web](./images/node-red-bot-conversation.jpg)

11. Note: you can always access the Node RED editor at `your-website.mybluemix.net/red`.

12. You can download and import the whole flow [here](./Node-RED/basic-flow.json) ... don't forget to update the Workspace ID if you do use this!

13. The web application should look something likes this: ![flow web](./images/conversation-web-app.jpg)

- If your chatbot isn't working as you expect, you should wire debug node(s) into your Node-RED flows to determine where the error is.

13. You can also debug by looking at `User Conversations` in the Watson Conversation application. This can show you if you are finding the right `intents` and `entities` as you move down your dialog tree. ![conv debug](./images/conversation-debug.jpg)

You can also train and further improve Watson Conversation using this feature - read more about it [here](https://console.bluemix.net/docs/services/conversation/logs.html#about-the-improve-component).

## Connecting Conversation with Slack
1. Login to Slack [here](https://ibm-fsa2017-eu.slack.com/apps/new/A0F7YS25R-bots) and choose a unique name for your bot.
  - in general, you can create bot in any Slack team which you are a member of [here](https://my.slack.com/services/new/bot)

2. Save the API Token you get at this point - you'll need it shortly.

3. Head over your Node-RED flow editor - we are going to create a new flow for the Slack chatbot that will essentially be another way of interacting with your conversation.

5. In the top-right menu select `Manage palette` and then `Install`.

6. Search for and install `node-red-contrib-chatbot`.

7. Create `Slack In`, `Change`, `Conversation`, `Function`, `Text`, and `Slack Out` nodes and connect them together:
![flow 2](./images/node-red-2.png)

8. Configure the `Slack In` node by clicking on the pencil and entering your Bot name and API Token.

9. Configure the `Slack Out` node to use the same bot.

10. In the `Change` node we've called `Update payload` here, set `msg.payload` to `msg.payload.content`. Make sure you change the type to `msg.` as the default is `string`. `msg.payload.content` is where Slack places the received message, and we need to move this to `msg.payload` as this is where the Conversation node is expecting its input.
![change config](./images/node-red-3.png)

11. Amend the `Conversation` node to reflect your Watson Conversation Workspace ID and ensure `Save context` is checked.

12. Paste this into the `Function` node we've called `Postprocessing`:
```javascript
var output = msg.payload.output.text;
for (var t in output) {
    msg.payload = output[t];
    node.send(msg);
}
return ;
```
This node formats and sends a Slack message for every response received from Watson Conversation.

13. You need to configure the text node that constructs the message by removing its message field and leaving it empty as on the picture:
![change config](./images/node-red-4.png)

15. Deploy your Node-RED flow, and go over to Slack and try to chat with your bot. ![slack app](./images/conversation-Slack-app.jpg)

16. You can download and import the whole flow [here](./Node-RED/slack-flow.json).

Well done! You've built a basic chatbot that uses two UIs!
