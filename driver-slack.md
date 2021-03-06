# Slack

- [Usage with a Slack App (recommended)](#installation-setup)
- [Usage with the Realtime API](#installation-setup)
- [Usage with an outgoing webhook](#sending-facebook-templates)
- [Sending Slack Menus](#sending-slack-menus)

Slack is a cloud-based set of team collaboration tools and services.

## Usage with a Slack App (recommended)

The easiest way of making use of all BotMan features in your Slack team, is to create your own Slack app. You can do this at the [Slack API website](https://api.slack.com/apps?new_app=1).

Once you have created a new Slack app, you need to configure three sections:

#### Interactive Messages
Interactive messages allow your chatbot to respond to message buttons. If you want to use this feature, just configure it and provide the URL that points to your BotMan logic / controller.

#### Event Subscriptions
Event subscription is needed, so that your BotMan application receives all incoming messages. You can either use event subscriptions or the Realtime API.
When configuring your Slack app for event subscriptions, provide the URL that points to your BotMan logic / controller in the "Request URL" field.

Then subscribe for these bot events:
`message.channels` - A message was posted to a channel
`message.im` - A message was posted in a direct message channel

#### Bots
This is the most obvious part - to communicate with BotMan in your Slack team, you need a bot user. Just give it a nice name.

After you have configured these sections, go to "Install App" and install the app in your own team.
Take note of the *Bot User OAuth Access Token* and use this token as your `slack_token` configuration parameter.

That's it.


## Usage with the Realtime API

> {callout-warning} Please note: The Realtime API requires the additional composer package `mpociot/slack-client` to be installed.
> 
> Simply install it using `composer require mpociot/slack-client`.

Add a new Bot user to your Slack team and take note of the bot token slack gives you.
Use this token as your `slack_token` configuration parameter.

As the Realtime API needs a websocket, you need to create a PHP script that will hold your bot logic, as you can not use the HTTP controller way for it.

```php
<?php
require 'vendor/autoload.php';

use Mpociot\BotMan\BotManFactory;
use React\EventLoop\Factory;

$loop = Factory::create();
$botman = BotManFactory::createForRTM([
    'slack_token' => 'YOUR-SLACK-BOT-TOKEN'
], $loop);

$botman->hears('keyword', function($bot) {
    $bot->reply('I heard you! :)');
});

$botman->hears('convo', function($bot) {
    $bot->startConversation(new ExampleConversation());
});

$loop->run();
```

Then simply run this file by using `php my-bot-file.php` - your bot should connect to your Slack team and respond to the messages.

## Usage with an outgoing webhook

Add a new "Outgoing Webhook" integration to your Slack team - this URL needs to point to the controller where your BotMan bot is living in. You will find this integration when you search for `Outgoing WebHooks` in the Slack Apps Directory.

To let BotMan listen to all incoming messages, do **not** specify a trigger word, but define a channel instead.

With the Webhook implementation, there is no need to add a `slack_token` configuration. 

<a id="sending-slack-menus"></a>
## Sending Slack Menus

BotMan supports sending Slack's [interactive message menus](https://api.slack.com/docs/message-menus) through an expressive and easy to use API.
They can be used in combination with BotMan's [conversation questions](/conversations#asking-questions).

There are multiple ways to define an interactive message menu. You can simply attach a `Menu` object to your question, just as you would with regular Buttons.

You can access the selected option(s) using `$answer->getValue()` which will then return an array containing the selected option values.

### Custom drop down menu 

<center>
	<img width="366" src="/img/slack/interactive_menus.png" />
</center>

```php
// Inside your conversation
$question = Question::create('Would you like to play a game?')
    ->callbackId('game_selection')
    ->addAction(
    	Menu::create('Pick a game...')
    		->name('games_list')
    		->options([
    			[
    				'text' => 'Hearts',
    				'value' => 'hearts',
    			],
    			[
    				'text' => 'Bridge',
    				'value' => 'bridge',
    			]
    			[
    				'text' => 'Poker',
    				'value' => 'poker',
    			]
    		])
    );

$this->ask($question, function (Answer $answer) {
    $selectedOptions = $answer->getValue();
});
```

### Allow users to select from a list of team members

<center>
	<img width="366" src="/img/slack/interactive_menus_users.png" />
</center>

```php
// Inside your conversation
$question = Question::create('Who wins the lifetime supply of chocolate?')
    ->callbackId('select_users')
    ->addAction(
    	Menu::create('Who should win?')
    		->name('winners_list')
    		->chooseFromUsers()
    );

$this->ask($question, function (Answer $answer) {
    $selectedOptions = $answer->getValue();
});
```

### Let users choose one of their team's channels

<center>
	<img width="366" src="/img/slack/interactive_menus_channels.png" />
</center>

```php
// Inside your conversation
$question = Question::create('It\'s time to nominate the channel of the week')
    ->callbackId('select_channel')
    ->addAction(
    	Menu::create('Which channel changed your life this week?')
    		->name('channels_list')
    		->chooseFromChannels()
    );

$this->ask($question, function (Answer $answer) {
    $selectedOptions = $answer->getValue();
});
```

### Let users choose one of their conversations

```php
// Inside your conversation
$question = Question::create('Let\'s get a productive conversation going')
    ->callbackId('select_conversation')
    ->addAction(
    	Menu::create('Who did you talk to last?')
    		->name('conversations_list')
    		->chooseFromConversations()
    );

$this->ask($question, function (Answer $answer) {
    $selectedOptions = $answer->getValue();
});
```