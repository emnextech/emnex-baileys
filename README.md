<div align="center">
  <h1>Emnex-Baileys</h1>
  <p>A WebSocket-based JavaScript library for interacting with the WhatsApp Web API</p>

  [![npm version](https://img.shields.io/npm/v/emnex-baileys.svg)](https://www.npmjs.com/package/emnex-baileys)
  [![npm downloads](https://img.shields.io/npm/dm/emnex-baileys.svg)](https://www.npmjs.com/package/emnex-baileys)
  [![License](https://img.shields.io/npm/l/emnex-baileys.svg)](https://github.com/emnextech/emnex-baileys/blob/main/LICENSE)
</div>

## Disclaimer

This project is not affiliated, associated, authorized, endorsed by, or in any way officially connected with WhatsApp or any of its subsidiaries or affiliates. Use at your own discretion. Do not spam people with this. We discourage any stalkerware, bulk or automated messaging usage.

## Credits & Acknowledgements

`emnex-baileys` (maintained by **Emnex Tech**) stands on the shoulders of others:

- **[Baileys](https://github.com/WhiskeySockets/Baileys)** by **WhiskeySockets** — the original, brilliant WhatsApp Web API implementation that makes all of this possible.
- **gifted-baileys** by **Gifted Tech** — the fork this project is based on. Big thanks to Gifted Tech for their work.

This library is MIT-licensed and retains the copyright notices of the projects above. Please support and credit the upstream authors.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Authentication](#authentication)
  - [QR Code](#qr-code)
  - [Pairing Code](#pairing-code)
- [Connection & Reconnection](#connection--reconnection)
- [Events](#events)
- [Sending Messages](#sending-messages)
- [Receiving & Downloading Media](#receiving--downloading-media)
- [Group Management](#group-management)
- [Channels / Newsletters](#channels--newsletters)
- [Status / Stories](#status--stories)
- [Profile & Privacy](#profile--privacy)
- [License](#license)

## Installation

```bash
npm install emnex-baileys
```

Or using yarn:

```bash
yarn add emnex-baileys
```

> **Requirements:** Node.js 20 or newer.

## Quick Start

### CommonJS (Recommended)

```javascript
const { default: makeWASocket, useMultiFileAuthState, Browsers, DisconnectReason } = require('emnex-baileys')

async function start() {
  const { state, saveCreds } = await useMultiFileAuthState('auth')

  const sock = makeWASocket({
    auth: state,
    browser: Browsers.ubuntu('Chrome'),
  })

  // persist updated credentials
  sock.ev.on('creds.update', saveCreds)

  sock.ev.on('connection.update', (update) => {
    const { connection } = update
    if (connection === 'open') console.log('✅ connected')
  })

  sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0]
    if (!msg.message || msg.key.fromMe) return
    if (msg.message.conversation === 'hi') {
      await sock.sendMessage(msg.key.remoteJid, { text: 'Hello 👋' })
    }
  })
}

start()
```

### ES Modules / TypeScript

```javascript
import pkg from 'emnex-baileys'
const { default: makeWASocket, useMultiFileAuthState, Browsers } = pkg
```

## Authentication

`useMultiFileAuthState('auth')` stores the session in a folder (here `./auth`) so you only
authenticate once. There are two ways to link a device.

### QR Code

The `printQRInTerminal` option is **deprecated and no longer prints automatically**. Listen
to the `connection.update` event and render the `qr` string yourself, e.g. with
[`qrcode-terminal`](https://www.npmjs.com/package/qrcode-terminal):

```javascript
const qrcode = require('qrcode-terminal')

sock.ev.on('connection.update', (update) => {
  const { qr } = update
  if (qr) qrcode.generate(qr, { small: true })
})
```

Then open WhatsApp → **Settings → Linked Devices → Link a Device** and scan it.

### Pairing Code

Instead of a QR, request an 8-character pairing code for a phone number (international
format, digits only):

```javascript
if (!sock.authState.creds.registered) {
  const code = await sock.requestPairingCode('254700000000')
  console.log('Pairing code:', code) // enter this in WhatsApp → Link a Device → Link with phone number
}

// optional: supply your own 8-char custom code
// const code = await sock.requestPairingCode('254700000000', 'EMNEX123')
```

## Connection & Reconnection

Handle drops and re-link based on the disconnect reason:

```javascript
const { DisconnectReason } = require('emnex-baileys')

sock.ev.on('connection.update', (update) => {
  const { connection, lastDisconnect } = update
  if (connection === 'close') {
    const code = lastDisconnect?.error?.output?.statusCode
    if (code !== DisconnectReason.loggedOut) {
      start() // reconnect
    } else {
      console.log('Logged out — delete the ./auth folder and re-authenticate.')
    }
  }
})
```

## Events

Subscribe with `sock.ev.on(eventName, handler)`. Commonly used events:

| Event | Fires when |
|-------|------------|
| `connection.update` | Connection state changes (incl. `qr`, `connection`, `lastDisconnect`) |
| `creds.update` | Auth credentials change — pair with `saveCreds` |
| `messages.upsert` | New or appended messages arrive |
| `messages.update` | Existing messages are edited / status changes |
| `message-receipt.update` | Read / delivery receipts |
| `groups.update` | Group metadata changes |
| `group-participants.update` | Members added / removed / promoted |
| `contacts.update` | Contact info changes |
| `presence.update` | Online / typing presence |

## Sending Messages

All sends go through `sock.sendMessage(jid, content, options?)`. The `jid` is
`number@s.whatsapp.net` for users, `id@g.us` for groups, or `id@newsletter` for channels.

```javascript
const jid = '254700000000@s.whatsapp.net'

// Text
await sock.sendMessage(jid, { text: 'Hello from emnex-baileys' })

// Reply to a message
await sock.sendMessage(jid, { text: 'replying' }, { quoted: someMsg })

// Image (Buffer or { url })
await sock.sendMessage(jid, { image: { url: './photo.jpg' }, caption: 'nice 📸' })

// Video
await sock.sendMessage(jid, { video: { url: './clip.mp4' }, caption: 'watch' })

// Audio / voice note
await sock.sendMessage(jid, { audio: { url: './voice.mp3' }, mimetype: 'audio/mp4', ptt: true })

// Document
await sock.sendMessage(jid, { document: { url: './file.pdf' }, mimetype: 'application/pdf', fileName: 'file.pdf' })

// Location
await sock.sendMessage(jid, { location: { degreesLatitude: -1.286389, degreesLongitude: 36.817223 } })

// Contact (vCard)
await sock.sendMessage(jid, {
  contacts: {
    displayName: 'Emnex',
    contacts: [{ vcard: 'BEGIN:VCARD\nVERSION:3.0\nFN:Emnex\nTEL;type=CELL:254700000000\nEND:VCARD' }],
  },
})

// Reaction (emoji on a message key)
await sock.sendMessage(jid, { react: { text: '🔥', key: someMsg.key } })
```

> **Note:** For interactive buttons, use the separate [`gifted-btns`](https://npmjs.com/package/gifted-btns) package.

## Receiving & Downloading Media

```javascript
const { downloadMediaMessage } = require('emnex-baileys')

sock.ev.on('messages.upsert', async ({ messages }) => {
  const msg = messages[0]
  const type = Object.keys(msg.message || {})[0]
  if (['imageMessage', 'videoMessage', 'audioMessage', 'documentMessage'].includes(type)) {
    const buffer = await downloadMediaMessage(msg, 'buffer', {})
    // write `buffer` to disk, re-upload, etc.
  }
})
```

## Group Management

```javascript
// Create a group
const group = await sock.groupCreate('My Group', ['254700000000@s.whatsapp.net'])

// Add / remove / promote / demote participants
await sock.groupParticipantsUpdate(group.id, ['254711111111@s.whatsapp.net'], 'add')    // 'remove' | 'promote' | 'demote'

// Update subject
await sock.groupUpdateSubject(group.id, 'New Name')

// Read metadata
const meta = await sock.groupMetadata(group.id)

// Invite links
const code = await sock.groupInviteCode(group.id)   // -> https://chat.whatsapp.com/<code>
await sock.groupAcceptInvite('<code>')

// Leave
await sock.groupLeave(group.id)
```

## Channels / Newsletters

Channels use `xxxxxxxxxxxx@newsletter` JIDs.

```javascript
// Resolve a channel invite link to its metadata (incl. the @newsletter id)
const meta = await sock.newsletterMetadata('invite', '0029Vb...')   // the code after /channel/
console.log(meta.id, meta.name)

// By JID
const byJid = await sock.newsletterMetadata('jid', '120363xxxxxxxxx@newsletter')

// Manage
await sock.newsletterCreate('My Channel', 'Description here')
await sock.newsletterFollow('120363xxxxxxxxx@newsletter')
await sock.newsletterUnfollow('120363xxxxxxxxx@newsletter')
await sock.newsletterMute('120363xxxxxxxxx@newsletter')
await sock.newsletterFetchMessages('120363xxxxxxxxx@newsletter', 50)
```

### How to find your channel ID

Every message that arrives from a channel has a `remoteJid` ending in `@newsletter`.
Log it from `messages.upsert`:

```javascript
sock.ev.on('messages.upsert', ({ messages }) => {
  for (const m of messages) {
    const jid = m.key?.remoteJid
    if (jid?.endsWith('@newsletter')) console.log('channel:', jid)
  }
})
```

Alternatively, resolve a channel's invite link with `newsletterMetadata('invite', code)` (above).

## Status / Stories

Send to your status broadcast by targeting `status@broadcast` and listing the recipients
who should see it via `statusJidList`:

```javascript
await sock.sendMessage(
  'status@broadcast',
  { image: { url: './story.jpg' }, caption: 'my status' },
  { statusJidList: ['254700000000@s.whatsapp.net'] },
)
```

## Profile & Privacy

```javascript
await sock.updateProfileName('Emnex Bot')
await sock.updateProfileStatus('Powered by emnex-baileys')
await sock.updateProfilePicture('me@s.whatsapp.net', { url: './avatar.jpg' })
await sock.sendPresenceUpdate('available') // 'available' | 'unavailable' | 'composing' | 'recording'
```

## License

Released under the [MIT License](./LICENSE). Copyright © Emnex Tech, with retained notices
for Gifted Tech and Baileys (WhiskeySockets). See [Credits & Acknowledgements](#credits--acknowledgements).
