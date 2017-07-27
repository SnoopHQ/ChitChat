# ChitChat

A chat bot framework.

This a chat bot framework with fairly ambitious long term goals of abstracting away platform specific constructs in favor of universal chat bot primitives. In the near-term it will provide an easy way to create Slack chat bot applications.

## Getting Started

### Installation

### Basic Setup

### Configuring Slack

### Configuring Firebase

----

## API

### `ChitChat.createApp(options)`

Initialize the application with configuration options.

#### General Configuration

General Information about your application.

- `options.appName` (required) -  (String) User friendly application name

#### Slack Configuration

These fields are required for communicating with Slack. Eventually, this will give way to many platform integrations.

- `options.slackClientId` (required) - (String) Your application’s Slack client ID
- `options.slackClientSecret` (required) - (String) Your application’s Slack client secret

#### Firebase Configuration

For now, the only persistence option is Firebase (more will be added as the framework evolves).

You must either provide an instance of Firbase admin object that has been initialized (if you wish to manage it yourself) or supply credentials and database URL for ChitChat to manage it for you.

- `options.firebaseInstance` - (Object) Instance of firebase
- `options.firebaseCredential` - (Object) Firebase credential JSON
- `options.firebaseDatabaseUrl` - (String) Firebase database URL

#### Express Server Configuration

Express is used to create the endpoints for Slack to comminicate with your application.

You can either provide a router from a Express application you are manage yourself or ChitChat will create one for you.

- `options.expressRouter` - (Object) Instance of an Express router
- `options.hostname` - (String) The server’s hostname (default: `localhost`)
- `options.port` - (Number) The server's port (default: `8080`)

----

### Commands

Commands are simple interactions that don't require persistent state. Sometimes commands can be used to start more complex **Conversations**.

#### `ChitChat.createCommand(options)`

Creates a slash command. Names must be unique. An unamed command will handle default cases where no names are matched. Help command is automatically generated based on description text.

- `options.name` - (String) Command name
- `options.description` - (String) User friendly description of the command
- `options.arguments` - (Array<String>) Array of named positional arguments
- `options.generateResponse` - ((context: ChitChatContext) => (ChitChatMessage | Promise<ChitChatMessage>)?) Generate the response message for this command

#### `ChitChat.runCommand(name, args)`

Run a registered command with the current context.

- `name` (required) - (String) the name of the registered command to run
- `args` - (Object) positional arguments as key, value pairs

----

### Triggers

Triggers are reactions to various events within a chat. For instance, there can be a trigger for when certain a certain message is received or when a reaction is added to a message. 

In general, triggers are more ephemeral than commands. They are generally created as the application runs and removed as a matter of course. They can be useful for complex interactions that span multiple channels or interaction types.

#### `ChitChatTriggerType` Constants

- `MESSAGE` - Trigger when message text is matched
- `REACTION` - Trigger when a reaction is added to a message
- `COMMAND` - Trigger when command is run

#### `ChitChat.createTrigger(options)`

- `options.type` (required) - (ChitChatTriggerType) the type of trigger
- `options.user` - (String) if this trigger applies to one user, that user's ID
- `options.channel` - (String) if this trigger applies to a single channel, that channel's ID
- `options.once` - (Boolean) if true, trigger will execute exactly once, if not it will continue to execute indefinitely
- `options.oncePerUser` - (Boolean) if true, trigger will execute at most once per user
- `options.excludeUsers` - (Array<String>) users that are excluded from this trigger
- `options.generateResponse` - ((context: ChitChatContext) => (ChitChatMessage | Promise<ChitChatMessage>)?) generate a response to this command

##### For `MESSAGE` Type Triggers

- `options.text` (required) - (String | Regex) the message text to match

##### For `REACTION` Type Triggers

- `options.message` (required) - (ChitChatMessage) the message to watch for reactions
- `options.reaction` - (String) the reaction to look for (if left out, will fire for all reactions)

##### For `COMMAND` Type Triggers

- `options.commandName` (required) - (String) the command to respond to
- `options.commandArguments` - (Object) the specific argument values to look for

----

### Conversations

A conversation is a more complex contruct that consists of one or more **interactions**. Conversations are stateful and can aggregate and persist state over the course of these interactions. A conversation, in general, happens with a single user in a single channel. However, deviations from this simple pattern can be acheived using trigger based interactions.

#### `ChitChat.createConversation(options)`

- `options.name` (required) - (String) a name for this type of conversation, must be unique within your application
- `options.interactions` (required) - (Array<ChitChatInteraction>) a list of interaction that can occur in this conversation
- `options.initialInteraction` - ((context: ChitChatContext, args: Object) => ChitChatInteraction) a function that returns the first interaction of this conversation

#### `ChitChat.startConversation(name, args)`

- `name` (required) - (String) the name of the conversation to start
- `args` - (Object) arguments that get passed in to conversation as key, value pairs

----

### Interactions

An interaction is composed of a question, or prompt, a user answer, and a response. An interaction can also route to the next interaction in a conversation based on state and response.

#### `ChitChatInteractionType` Constants

- `MESSAGE` - Simple text based interaction
- `BUTTON_CHOICE` - User chooses from a list of buttons
- `TRIGGER` - Interaction completes when a trigger is invoked

#### `ChitChat.createInteraction(options)`

- `options.name` (required) - (String) the name identifier of this interaction
- `options.type` (required) - (ChitChatInteractionType) the type of interaction
- `options.onComplete` - ((context: ChitChatContext, state: Object, response: any) => (void | Promise<void>)) lifecycle function that allows you to perform application specific tasks, like persisting data
- `options.nextInteraction` - ((context: ChitChatContext, state: Object, response: any) => ChitChatInteraction?) produce the next interaction. If not supplied or falsey value returned, conversation will end.

##### For `MESSAGE` type Interactions

- `options.generatePrompt` (required) - ((context: ChitChatContext, state: Object) => (ChitChatMessage | Promise<ChitChatMessage>)) generate the interaction's initial prompt
- `options.parseResponse` - ((response: String) => any) parse the response from raw text
- `options.validateResponse` - ((response: any) => Boolean) validate if the response is valid
- `options.generateInvalidMessage` - ((context: ChitChatContext, state: Object, response: any) => (ChitChatMessage | Promise<ChitChatMessage>)) a message to help the user arrive at a valid response

##### For `BUTTON_CHOICE` type Interactions

- `options.generatePrompt` (required) - ((context: ChitChatContext, state: Object) => Array<ChitChatButton>) generate the interaction's initial prompt
- `options.acknowledgeSelectionText` - (String) Text to acknowledge the user's button choice. If not specified `:thumbsup:` will be used

##### For `TRIGGER` type Interactions

- `options.generatePrompt` (required) - ((context: ChitChatContext, state: Object) => (ChitChatMessage | Promise<ChitChatMessage>)) generate the interaction's initial prompt
- `options.generateTrigger` (required) - ((context: ChitChatContext, state: Object) => ChitChatTrigger) generate the trigger that will conclude the interaction

----

### TODO: ChitChatMessage

----

### TODO: ChitChatContext

----

### TODO: ChitChatTeam

----

### TODO: ChitChatUser

----

## TODO: General Guidelines

----

## TODO: Example Application
