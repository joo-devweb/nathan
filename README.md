# nathan - Baileys Fork

This package is a modified fork of the popular Baileys library, maintained and enhanced by Nathan. It provides a robust and comprehensive API for interacting with WhatsApp Web, enabling developers to build custom WhatsApp bots, automation scripts, and other applications. This fork aims to offer specific modifications and improvements while retaining the core functionality of Baileys.

## Features

As a fork of Baileys, this package inherits and potentially enhances a wide range of features for WhatsApp automation:

*   **Multi-Device Support**: Connect seamlessly with WhatsApp's multi-device architecture.
*   **Sending & Receiving Messages**: Full support for text, media (images, videos, audio, documents), and contact messages.
*   **Group Management**: Create, manage, and interact with WhatsApp groups.
*   **Presence Updates**: Handle online/offline status, typing, and recording indicators.
*   **Message Parsing & Serialization**: Tools for working with WhatsApp's protobuf messages.
*   **Advanced Authentication**: Secure session management with multi-file authentication.
*   **Utilities**: A comprehensive set of helper functions for various WhatsApp operations.

## Installation

Ensure you have Node.js version `20.0.0` or higher installed.

```bash
npm install nathan
```

## Usage

Here's a basic example demonstrating how to connect to WhatsApp and send a simple message using `@nathan`. This package leverages `makeWASocket` to establish a connection, similar to the original Baileys library.

```javascript
const { makeWASocket, useMultiFileAuthState, DisconnectReason, fetchLatestBaileysVersion } = require('nathan');
const { Boom } = require('@hapi/boom');
const pino = require('pino');

async function connectToWhatsApp() {
    // Load authentication state from file
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys');

    // Fetch the latest Baileys version
    const { version, isLatest } = await fetchLatestBaileysVersion();
    console.log(`Using Baileys v${version.join('.')}. Is latest: ${isLatest}`);

    // Create a new WhatsApp socket instance
    const sock = makeWASocket({
        logger: pino({ level: 'silent' }), // Set to 'debug' for more verbose logging
        printQRInTerminal: true, // Display QR code in the terminal for pairing
        browser: ['Ubuntu', 'Chrome', '1.0'], // Custom browser identifier
        auth: state, // Pass the authentication state
        version: version // Use the fetched Baileys version
    });

    // Handle connection updates
    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        if (connection === 'close') {
            // Reconnect if not logged out
            const shouldReconnect = (lastDisconnect.error instanceof Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
            console.log('Connection closed due to', lastDisconnect.error, ', reconnecting:', shouldReconnect);
            if (shouldReconnect) {
                connectToWhatsApp();
            }
        } else if (connection === 'open') {
            console.log('Connection opened successfully!');
        }
    });

    // Save authentication credentials when updated
    sock.ev.on('creds.update', saveCreds);

    // Handle incoming messages
    sock.ev.on('messages.upsert', async ({ messages, type }) => {
        if (type === 'notify') {
            for (const msg of messages) {
                if (!msg.key.fromMe && msg.message) {
                    const senderName = msg.pushName || 'User';
                    const messageText = msg.message.conversation || msg.message.extendedTextMessage?.text || 'Media Message';
                    console.log(`Received message from ${senderName}: ${messageText}`);

                    // Simple echo bot example
                    if (messageText.toLowerCase() === 'ping') {
                        await sock.sendMessage(msg.key.remoteJid, { text: 'pong' });
                        console.log('Sent: pong');
                    } else if (messageText.toLowerCase() === 'hello') {
                        await sock.sendMessage(msg.key.remoteJid, { text: `Hello ${senderName}! How can I help you?` });
                        console.log('Sent: Hello response');
                    }
                }
            }
        }
    });
}

connectToWhatsApp();
```

## API

The `@nathan` package exposes a comprehensive API consistent with the Baileys design. Key exports include:

*   `makeWASocket`: The primary function to create a WhatsApp socket connection.
*   `proto`: Protocol buffer definitions for WhatsApp messages and other data structures.
*   `Utils`: A collection of utility functions for common operations.
*   `Types`: TypeScript definitions for various data types used within the library.
*   `Store`: Modules for managing session data and message history.
*   `WABinary`: Tools for handling WhatsApp's binary protocol.

For detailed API documentation, please refer to the Baileys official documentation as this fork largely maintains its structure, or consult the `lib/index.d.ts` for specific type definitions in this fork.

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/joo-devweb/nathan/blob/main/LICENSE) file for details.

## Author

*   **Nathan** - [GitHub Repository](https://github.com/joo-devweb/nathan)
