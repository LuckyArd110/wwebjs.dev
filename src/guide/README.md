---
title: Getting Started
description: How to connect and send/receive messages
---

# Getting Started

:::warning
The whatsapp-web.js guide is still a work in progress. To learn about all the features available to you in the library, please check out the [documentation](https://docs.wwebjs.dev/).
:::

## Installation

You can get the module from npm:

```
$ npm i whatsapp-web.js
```

:::tip Info
 NodeJS v12 or higher is required
:::

## First steps

Once installed, you're ready to connect:

```javascript
const { Client } = require('whatsapp-web.js');
const client = new Client();

client.on('qr', (qr) => {
    console.log('QR RECEIVED', qr);
});

client.on('ready', () => {
    console.log('Client is ready!');
});

client.initialize();
```

### QR code generation

Since whatsapp-web.js works by running WhatsApp Web in the background and automating its interaction, you'll need to authorize the client by scanning a QR code from WhatsApp on your phone.

Right now, we're just logging the text representation of that QR code to the console, but we can do better. Let's install and use [qrcode-terminal](https://www.npmjs.com/package/qrcode-terminal) so we can render the QR code:

```bash
$ npm i qrcode-terminal
```

And now we'll modify our code to use this new module:

```javascript
const qrcode = require('qrcode-terminal');

const { Client } = require('whatsapp-web.js');
const client = new Client();

client.on('qr', qr => {
    qrcode.generate(qr, {small: true});
});

client.on('ready', () => {
    console.log('Client is ready!');
});

client.initialize();
```

There we go! You should now see something like this after running the file:

![](./images/qr-gen.png)

After scanning this QR code, the client should be authorized and you should see a `Client is ready!` message being printed out.


### Listening for messages

Now that we can connect to WhatsApp, it's time to listen for incoming messages. Doing so with whatsapp-web.js is pretty straightforward. The client emits a `message` event whenever a message is received. This means we can capture it like so:

```javascript
client.on('message', message => {
	console.log(message.body);
});
```

The code above, by default is listening in personal chat and group chat. If you want to specify the chat come from, you can identify using `isGroup` key extended from `getChat()` function. The keys will return `Boolean`. So, the code looks like:

```javascript
client.on('message', async message => {
  	const chat = await message.getChat();
  	if (chat.isGroup){
  		console.log("Group message");
  	} else {
  		console.log("Private Message");
  	}
  }
```

Running this example should log all incoming messages to the console.

### Replying to messages

The messages received have a convenience function on them that allows you to directly reply to them via WhatsApp's reply feature. This will show the quoted message above the reply.

To test this out, let's build a simple ping/pong command:

```javascript
client.on('message', message => {
	if(message.body === '!ping') {
		message.reply('pong');
	}
});
```

![](./images/ping-reply.png)

You could also choose **not** to send it as a quoted reply by using the `sendMessage` function available on the client:

```javascript
client.on('message', message => {
	if(message.body === '!ping') {
		client.sendMessage(message.from, 'pong');
	}
});
```

![](./images/ping.png)

In this case, notice that we had to specify which chat we were sending the message to.

