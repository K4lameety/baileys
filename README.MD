# <div align='center'>Baileys</div>

<div align='center'>

[![GitHub](https://img.shields.io/github/stars/K4lameety/baileys?style=social)](https://github.com/K4lameety/baileys)
[![License](https://img.shields.io/badge/license-GPL%203-blue.svg)](LICENSE)
[![GitHub Issues](https://img.shields.io/github/issues/K4lameety/baileys)](https://github.com/K4lameety/baileys/issues)

</div>

---

## ✨ Key Features

- 🚀 **Modern & Fast** - Built with TypeScript and the latest technology
- 🔧 **@lid & @jid Fix** - Resolves @lid to @pn issues in WhatsApp groups
- 📱 **Multi-Device Support** - Supports WhatsApp multi-device connections
- 🔐 **End-to-End Encryption** - Fully encrypted communication
- 📨 **All Message Types** - Supports text, media, polling, etc.

## ⚠️ Warning

This project is not affiliated, associated, authorized, endorsed, or officially connected with WhatsApp or any of its subsidiaries. The official WhatsApp website is at whatsapp.com.

The Baileys maintainers do not support the use of this application for violating WhatsApp's Terms of Service. We emphasize users' personal responsibility to use it fairly and responsibly.

Use wisely. Avoid spam. Do not use excessive automation.

## 📦 Installation

### Install from GitHub

```bash
npm i github:K4lameety/baileys
# or
yarn add github:K4lameety/baileys
```

### Import in Code

```javascript
const { default: makeWASocket } = require("baileys")
// or ES6
import makeWASocket from "baileys"
```

## 🚀 Quick Start

### Basic Example

```javascript
const { default: makeWASocket, DisconnectReason, useMultiFileAuthState } = require('baileys')
const { Boom } = require('@hapi/boom')

async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')
    
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true,
        browser: ['Baileys', 'Desktop', '3.0']
    })

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update
        if(connection === 'close') {
            const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut
            console.log('Connection closed due to ', lastDisconnect.error, ', reconnecting ', shouldReconnect)
            if(shouldReconnect) {
                connectToWhatsApp()
            }
        } else if(connection === 'open') {
            console.log('✅ Successfully connected to WhatsApp!')
        }
    })

    sock.ev.on('messages.upsert', async ({ messages }) => {
        for (const m of messages) {
            if (!m.message) continue
            
            console.log('📱 New message:', JSON.stringify(m, undefined, 2))
            
            // Auto reply
            await sock.sendMessage(m.key.remoteJid!, { 
                text: 'Hello! I am a WhatsApp bot using Baileys 🤖' 
            })
        }
    })

    sock.ev.on('creds.update', saveCreds)
}

connectToWhatsApp()
```

## 📋 Table of Contents

Disclaimer: This documentation is still in beta, so there may be errors or inconsistencies 

## 🔌 Account Connection

WhatsApp provides a multi-device API that allows Baileys to authenticate as a secondary WhatsApp client via QR code or pairing code.

### Connect with QR Code

> [!TIP]  
> Customize the browser name using the `Browsers` constant. See available configurations below.

```javascript
const { default: makeWASocket, Browsers } = require("baileys")

const sock = makeWASocket({
    browser: Browsers.ubuntu('My App'),
    printQRInTerminal: true
})
```

After successful connection, a QR code will appear in the terminal. Scan it with WhatsApp on your phone to login.

### Connect with Pairing Code

> [!IMPORTANT]  
> Pairing code is not part of the Mobile API. It allows WhatsApp Web connection without QR code, but only one device. See [WhatsApp FAQ](https://faq.whatsapp.com/).

Phone number must be without `+`, `()`, or `-`, and include the country code.

```javascript
const { default: makeWASocket } = require("baileys")

const sock = makeWASocket({
    printQRInTerminal: false
})

// Normal Pairing
if (!sock.authState.creds.registered) {
    const number = '6281251767935'
    const code = await sock.requestPairingCode(number)
    console.log('🔑 Pairing Code:', code)
}

// Custom Pairing
if (!sock.authState.creds.registered) {
    const pair = "BAIL1234" // 8 characters
    const number = '6281251767935'
    const code = await sock.requestPairingCode(number, pair)
    console.log('🔑 Custom Pairing Code:', code)
}
```

### Receive Full History

1. Set `syncFullHistory` to `true`.
2. By default, Baileys uses Chrome configuration. For desktop-like connection (for more message history), use:

```javascript
const { default: makeWASocket, Browsers } = require("baileys")

const sock = makeWASocket({
    browser: Browsers.macOS('Desktop'),
    syncFullHistory: true
})
```

## ⚙️ Important Socket Configuration Notes

### Group Metadata Caching (Recommended)

For group usage, implement group metadata caching:

```javascript
const { default: makeWASocket } = require("baileys")
const NodeCache = require('node-cache')

const groupCache = new NodeCache({ stdTTL: 5 * 60, useClones: false })

const sock = makeWASocket({
    cachedGroupMetadata: async (jid) => groupCache.get(jid)
})

sock.ev.on('groups.update', async ([event]) => {
    const metadata = await sock.groupMetadata(event.id)
    groupCache.set(event.id, metadata)
})

sock.ev.on('group-participants.update', async (event) => {
    const metadata = await sock.groupMetadata(event.id)
    groupCache.set(event.id, metadata)
})
```

### Fix Retry System & Poll Vote Decryption

Improve message sending and poll vote decryption with store:

```javascript
const sock = makeWASocket({
    getMessage: async (key) => await getMessageFromStore(key)
})
```

### Receive Notifications in WhatsApp App

Disable online status to receive notifications:

```javascript
const sock = makeWASocket({
    markOnlineOnConnect: false
})
```

## 💾 Save Auth Info

Avoid repeated QR code scanning by saving credentials:

```javascript
const makeWASocket = require("baileys").default
const { useMultiFileAuthState } = require("baileys")

async function connect() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')
    const sock = makeWASocket({ auth: state })
    sock.ev.on('creds.update', saveCreds)
}

connect()
```

> [!IMPORTANT]  
> `useMultiFileAuthState` saves auth state in a folder. For production, use a SQL/No-SQL database and manage key updates carefully.

```javascript
const sock = makeWASocket()
sock.ev.on('messages.upsert', ({ messages }) => {
    console.log('Got messages:', messages)
})
```

### Getting Started Example

```javascript
const makeWASocket = require("baileys").default
const { DisconnectReason, useMultiFileAuthState } = require("baileys")
const { Boom } = require('@hapi/boom')

async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update
        if(connection === 'close') {
            const shouldReconnect = (lastDisconnect.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut
            console.log('Connection closed due to ', lastDisconnect.error, ', reconnecting ', shouldReconnect)
            if(shouldReconnect) {
                connectToWhatsApp()
            }
        } else if(connection === 'open') {
            console.log('Connection opened')
        }
    })

    sock.ev.on('messages.upsert', async ({ messages }) => {
        for (const m of messages) {
            console.log(JSON.stringify(m, undefined, 2))
            console.log('Reply to', m.key.remoteJid)
            await sock.sendMessage(m.key.remoteJid!, { text: 'Hello World' })
        }
    })

    sock.ev.on('creds.update', saveCreds)
}

connectToWhatsApp()
```

### Decrypt Poll Votes

By default, poll votes are encrypted and handled in `messages.update`:

```javascript
sock.ev.on('messages.update', event => {
    for(const { key, update } of event) {
        if(update.pollUpdates) {
            const pollCreation = await getMessage(key)
            if(pollCreation) {
                console.log(
                    'poll update received, aggregation: ',
                    getAggregateVotesInPollMessage({
                        message: pollCreation,
                        pollUpdates: update.pollUpdates,
                    })
                )
            }
        }
    }
})
```

`getMessage` is a store implementation (on your side).

### First Connection Event Summary

1. On first connection, `connection.update` will be triggered requesting sock restart.
2. Then, history messages are received in `messaging.history-set`.

## 🗄️ Data Store Implementation

Baileys does not include default storage for chats, contacts, or messages. However, a simple in-memory implementation is provided. The store listens to chat updates, new messages, etc., to keep data up to date.

> [!IMPORTANT]  
> I highly recommend building your own data store, as storing entire chat history in memory is very wasteful of RAM.

```javascript
const makeWASocket = require("baileys").default
const { makeInMemoryStore } = require("baileys")

const store = makeInMemoryStore({ })

store.readFromFile('./baileys_store.json')

setInterval(() => {
    store.writeToFile('./baileys_store.json')
}, 10_000)

const sock = makeWASocket({ })
store.bind(sock.ev)

sock.ev.on('chats.upsert', () => {
    console.log('got chats', store.chats.all())
})

sock.ev.on('contacts.upsert', () => {
    console.log('got contacts', Object.values(store.contacts))
})
```

The store also provides simple functions like `loadMessages` to speed up data retrieval.

## 🆔 WhatsApp ID Explanation

`id` is the WhatsApp ID, also called `jid`, for the person or group you're sending a message to.

Format: `[country code][phone number]@s.whatsapp.net`  
Example for a person: `+19999999999@s.whatsapp.net`.  

For groups: `123456789-123345@g.us`.  

For broadcast lists: `[creation timestamp]@broadcast`.  

For stories: `status@broadcast`.

## 🔧 Utility Functions

- `getContentType` - Returns the content type of a message
- `getDevice` - Returns the device from a message
- `makeCacheableSignalKeyStore` - Speeds up auth store
- `downloadContentFromMessage` - Download content from a message

## 📤 Send Messages

Send all types of messages with a single function.

See supported message content in the section below.  
See options like quote in the example below.

```javascript
const jid: string
const content: AnyMessageContent
const options: MiscMessageGenerationOptions

sock.sendMessage(jid, content, options)
```

### Non-Media Messages

#### Text Message

```javascript
await sock.sendMessage(jid, { text: 'hello world' })
```

#### Quote Message (works with all types)

```javascript
await sock.sendMessage(jid, { text: 'hello world' }, { quoted: message })
```

#### Mention Users (works with most types)

`@number` to mention in text, optional.

```javascript
await sock.sendMessage(
    jid,
    {
        text: '@12345678901',
        mentions: ['12345678901@s.whatsapp.net']
    }
)
```

#### Forward Message

Requires a message object, get from store or use message object.

```javascript
const msg = getMessageFromStore() // implement yourself
await sock.sendMessage(jid, { forward: msg })
```

#### Location Message

```javascript
await sock.sendMessage(
    jid, 
    {
        location: {
            degreesLatitude: 24.121231,
            degreesLongitude: 55.1121221
        }
    }
)
```

#### Contact Message

```javascript
const vcard = 'BEGIN:VCARD\n' 
            + 'VERSION:3.0\n' 
            + 'FN:Jeff Singh\n' 
            + 'ORG:Ashoka Uni;\n' 
            + 'TEL;type=CELL;type=VOICE;waid=911234567890:+91 12345 67890\n' 
            + 'END:VCARD'

await sock.sendMessage(
    id,
    { 
        contacts: { 
            displayName: 'Jeff', 
            contacts: [{ vcard }] 
        }
    }
)
```

#### Reaction Message

Requires message key, get from store or use key object.

```javascript
await sock.sendMessage(
    jid,
    {
        react: {
            text: '💖', // empty to remove reaction
            key: message.key
        }
    }
)
```

#### Pin Message

Requires message key.

Time:

| Time | Seconds    |
|------|------------|
| 24h  | 86,400     |
| 7d   | 604,800    |
| 30d  | 2,592,000  |

```javascript
await sock.sendMessage(
    jid,
    {
        pin: {
            type: 1, // 0 to unpin
            time: 86400,
            key: message.key
        }
    }
)
```

#### Poll Message

```javascript
await sock.sendMessage(
    jid,
    {
        poll: {
            name: 'My Poll',
            values: ['Option 1', 'Option 2', ...],
            selectableCount: 1,
            toAnnouncementGroup: false // or true
        }
    }
)
```

### Send with Link Preview

1. By default, WA doesn't have link generation from web.
2. Baileys has a function for link preview.
3. Add `link-preview-js` with `npm i link-preview-js`.
4. Send a link:

```javascript
await sock.sendMessage(
    jid,
    {
        text: 'Hello, this is sent using https://npmjs.com/wileys'
    }
)
```

### Media Messages

Sending media (video, sticker, image) is easier & more efficient.

> [!NOTE] In media messages, you can use `{ stream: Stream }` or `{ url: Url }` or Buffer directly, see examples below.

Baileys doesn't load the entire buffer into memory; it encrypts as a stream.

> [!TIP] Use Stream or Url to save memory.

#### Gif Message

WA doesn't support .gif, send as .mp4 with `gifPlayback` flag.

```javascript
await sock.sendMessage(
    jid, 
    { 
        video: fs.readFileSync('Media/ma_gif.mp4'),
        caption: 'hello world',
        gifPlayback: true
    }
)
```

#### Video Message

```javascript
await sock.sendMessage(
    id, 
    { 
        video: {
            url: './Media/ma_gif.mp4'
        },
        caption: 'hello world',
        ptv: false // true for video note
    }
)
```

#### Audio Message

Convert with ffmpeg: `codec: libopus`, `ac: 1`, `avoid_negative_ts`, `make_zero`.

Example: `ffmpeg -i input.mp4 -avoid_negative_ts make_zero -ac 1 output.ogg`

```javascript
await sock.sendMessage(
    jid, 
    {
        audio: {
            url: './Media/audio.mp3'
        },
        mimetype: 'audio/mp4'
    }
)
```

#### Image Message

```javascript
await sock.sendMessage(
    id, 
    { 
        image: {
            url: './Media/ma_img.png'
        },
        caption: 'hello world'
    }
)
```

#### View Once Message

Add `viewOnce: true` for all messages above.

```javascript
await sock.sendMessage(
    id, 
    { 
        image: {
            url: './Media/ma_img.png'
        },
        viewOnce: true, // works with video, audio too
        caption: 'hello world'
    }
)
```

## ✏️ Modify Messages

### Delete Message (for everyone)

```javascript
const msg = await sock.sendMessage(jid, { text: 'hello world' })
await sock.sendMessage(jid, { delete: msg.key })
```

Note: Delete for self is supported via `chatModify`, see [Modify Chats](#modify-chats).

### Edit Message

Use editable content.

```javascript
await sock.sendMessage(jid, {
      text: 'updated text here',
      edit: response.key,
    });
```

## 🖼️ Media Message Manipulation

### Thumbnail in Media Messages

Thumbnail is automatic for images & stickers if you add `jimp` or `sharp` (`npm i jimp` or `npm i sharp`).  

For video, you need `ffmpeg` installed.

### Download Media Messages

To save received media:

```javascript
import { createWriteStream } from 'fs'
import { downloadMediaMessage, getContentType } from 'baileys'

sock.ev.on('messages.upsert', async ({ messages: [m] }) => {
    if (!m.message) return
    const messageType = getContentType(m)

    if (messageType === 'imageMessage') {
        const stream = await downloadMediaMessage(
            m,
            'stream', // or 'buffer'
            { },
            { 
                logger,
                reuploadRequest: sock.updateMediaMessage
            }
        )
        const writeStream = createWriteStream('./my-download.jpeg')
        stream.pipe(writeStream)
    }
})
```

### Re-upload Media Messages to WhatsApp

WA deletes old media from servers. Re-upload with:

```javascript
await sock.updateMediaMessage(msg)
```

## 📞 Reject Calls

Get `callId` and `callFrom` from the `call` event.

```javascript
await sock.rejectCall(callId, callFrom)
```

## 💬 Send Status in Chat

### Read Messages

A set of message keys must be explicitly marked as read.

```javascript
const key: WAMessageKey
await sock.readMessages([key]) // can be multiple keys
```

Message ID is a unique identifier. Access with `message.key.id`.

### Update Presence

`presence` can be: available, composing, recording, paused, unavailable.

Valid for 10 seconds. Let jid know whether you're online, offline, typing, etc.

```javascript
await sock.sendPresenceUpdate('available', jid) 
```

> [!NOTE] If desktop client is active, WA doesn't send push notifications. To receive notifications, set offline with `sock.sendPresenceUpdate('unavailable')`.

## 📁 Modify Chats

WA uses encrypted communication for chat/app updates.

> [!IMPORTANT] If you update incorrectly, WA may logout from all devices.

### Archive Chat

```javascript
const lastMsgInChat = await getLastMessageInChat(jid) // implement yourself
await sock.chatModify({ archive: true, lastMessages: [lastMsgInChat] }, jid)
```

### Mute/Unmute Chat

Supported times:

| Time   | Milliseconds |
|--------|-------------|
| Remove | null        |
| 8h     | 86,400,000  |
| 7d     | 604,800,000 |

```javascript
// Mute 8 hours
await sock.chatModify({ mute: 8 * 60 * 60 * 1000 }, jid)
// Unmute
await sock.chatModify({ mute: null }, jid)
```

### Mark Chat Read/Unread

```javascript
const lastMsgInChat = await getLastMessageInChat(jid)
// mark unread
await sock.chatModify({ markRead: false, lastMessages: [lastMsgInChat] }, jid)
```

### Delete Message for Me

```javascript
await sock.chatModify(
    {
        clear: {
            messages: [
                {
                    id: 'ATWYHDNNWU81732J',
                    fromMe: true, 
                    timestamp: '1654823909'
                }
            ]
        }
    }, 
    jid
)
```

### Delete Chat

```javascript
const lastMsgInChat = await getLastMessageInChat(jid)
await sock.chatModify({
        delete: true,
        lastMessages: [
            {
                key: lastMsgInChat.key,
                messageTimestamp: lastMsgInChat.messageTimestamp
            }
        ]
    },
    jid
)
```

### Pin/Unpin Chat

```javascript
await sock.chatModify({
        pin: true // or false to unpin
    },
    jid
)
```

### Star/Unstar Message

```javascript
await sock.chatModify({
        star: {
            messages: [
                {
                    id: 'messageID',
                    fromMe: true // or false
                }
            ],
            star: true // true: Star; false: Unstar
        }
    },
    jid
)
```

### Disappearing Messages

Ephemeral:

| Time   | Seconds     |
|--------|-------------|
| Remove | 0           |
| 24h    | 86,400      |
| 7d     | 604,800     |
| 90d    | 7,776,000   |

Use seconds, default is 7 days.

```javascript
// Enable disappearing messages
await sock.sendMessage(
    jid, 
    { disappearingMessagesInChat: WA_DEFAULT_EPHEMERAL }
)

// Send as disappearing message
await sock.sendMessage(jid, { text: 'hello' }, { ephemeralExpiration: WA_DEFAULT_EPHEMERAL })

// Disable
await sock.sendMessage(
    jid, 
    { disappearingMessagesInChat: false }
)
```

## 👤 Query Users

### Check if ID Exists on WhatsApp

```javascript
const [result] = await sock.onWhatsApp(jid)
if (result.exists) console.log(`${jid} exists on WhatsApp, as jid: ${result.jid}`)
```

### Query Chat History (groups too)

Requires oldest message in chat.

```javascript
const msg = await getOldestMessageInChat(jid) // implement yourself
await sock.fetchMessageHistory(
    50, // count (max: 50 per query)
    msg.key,
    msg.messageTimestamp
)
```

Messages are received in the `messaging.history-set` event.

### Fetch Status

```javascript
const status = await sock.fetchStatus(jid)
console.log('status: ' + status)
```

### Fetch Profile Picture (groups too)

```javascript
// low resolution
const ppUrl = await sock.profilePictureUrl(jid)
console.log(ppUrl)

// high resolution
const ppUrl = await sock.profilePictureUrl(jid, 'image')
```

### Fetch Business Profile (description or category)

```javascript
const profile = await sock.getBusinessProfile(jid)
console.log('business description: ' + profile.description + ', category: ' + profile.category)
```

### Fetch Someone's Presence (typing or online)

```javascript
sock.ev.on('presence.update', console.log)

await sock.presenceSubscribe(jid) 
```

## 🔄 Modify Profile

### Update Profile Status

```javascript
await sock.updateProfileStatus('Hello World!')
```

### Update Profile Name

```javascript
await sock.updateProfileName('My Name')
```

### Update Display Picture (groups too)

> [!NOTE] Like media messages, use `{ stream: Stream }` or `{ url: Url }` or Buffer, see [Media Messages](#media-messages).

```javascript
await sock.updateProfilePicture(jid, { url: './new-profile-picture.jpeg' })
```

### Remove Display Picture (groups too)

```javascript
await sock.removeProfilePicture(jid)
```

## 👥 Groups

To modify group properties, you must be an admin.

### Create Group

```javascript
const group = await sock.groupCreate('My Group', ['1234@s.whatsapp.net', '4564@s.whatsapp.net'])
console.log('group created with id: ' + group.gid)
await sock.sendMessage(group.id, { text: 'hello everyone' })
```

### Add/Remove or Demote/Promote

```javascript
await sock.groupParticipantsUpdate(
    jid, 
    ['abcd@s.whatsapp.net', 'efgh@s.whatsapp.net'],
    'add' // replace with 'remove', 'demote', or 'promote'
)
```

### Update Subject (name)

```javascript
await sock.groupUpdateSubject(jid, 'New Subject!')
```

### Update Description

```javascript
await sock.groupUpdateDescription(jid, 'New Description!')
```

### Update Settings

```javascript
// Only admins can send messages
await sock.groupSettingUpdate(jid, 'announcement')
// Everyone can send
await sock.groupSettingUpdate(jid, 'not_announcement')
// Everyone can modify group settings
await sock.groupSettingUpdate(jid, 'unlocked')
// Only admins can modify settings
await sock.groupSettingUpdate(jid, 'locked')
```

### Leave Group

```javascript
await sock.groupLeave(jid)
```

### Get Invite Code

Create link: `'https://chat.whatsapp.com/' + code`.

```javascript
const code = await sock.groupInviteCode(jid)
console.log('group code: ' + code)
```

### Revoke Invite Code

```javascript
const code = await sock.groupRevokeInvite(jid)
console.log('New group code: ' + code)
```

### Join Using Invite Code

Code without `https://chat.whatsapp.com/`.

```javascript
const response = await sock.groupAcceptInvite(code)
console.log('joined to: ' + response)
```

### Get Group Info from Invite Code

```javascript
const response = await sock.groupGetInviteInfo(code)
console.log('group info: ' + response)
```

### Query Metadata (participants, name, description...)

```javascript
const metadata = await sock.groupMetadata(jid) 
console.log(metadata.id + ', title: ' + metadata.subject + ', description: ' + metadata.desc)
```

### Join Using groupInviteMessage

```javascript
const response = await sock.groupAcceptInviteV4(jid, groupInviteMessage)
console.log('joined to: ' + response)
```

### Get Join Request List

```javascript
const response = await sock.groupRequestParticipantsList(jid)
console.log(response)
```

### Approve/Reject Join Request

```javascript
const response = await sock.groupRequestParticipantsUpdate(
    jid,
    ['abcd@s.whatsapp.net', 'efgh@s.whatsapp.net'],
    'approve' // or 'reject'
)
console.log(response)
```

### Get All Participating Group Metadata

```javascript
const response = await sock.groupFetchAllParticipating()
console.log(response)
```

### Toggle Ephemeral

Ephemeral:

| Time   | Seconds     |
|--------|-------------|
| Remove | 0           |
| 24h    | 86,400      |
| 7d     | 604,800     |
| 90d    | 7,776,000   |

```javascript
await sock.groupToggleEphemeral(jid, 86400)
```

### Update Add Mode

```javascript
await sock.groupMemberAddMode(
    jid,
    'all_member_add' // or 'admin_add'
)
```

## 🔒 Privacy

### Block/Unblock User

```javascript
await sock.updateBlockStatus(jid, 'block') // Block
await sock.updateBlockStatus(jid, 'unblock') // Unblock
```

### Fetch Privacy Settings

```javascript
const privacySettings = await sock.fetchPrivacySettings(true)
console.log('privacy settings: ' + privacySettings)
```

### Fetch Block List

```javascript
const response = await sock.fetchBlocklist()
console.log(response)
```

### Update Last Seen Privacy

```javascript
const value = 'all' // 'contacts' | 'contact_blacklist' | 'none'
await sock.updateLastSeenPrivacy(value)
```

### Update Online Privacy

```javascript
const value = 'all' // 'match_last_seen'
await sock.updateOnlinePrivacy(value)
```

### Update Profile Picture Privacy

```javascript
const value = 'all' // 'contacts' | 'contact_blacklist' | 'none'
await sock.updateProfilePicturePrivacy(value)
```

### Update Status Privacy

```javascript
const value = 'all' // 'contacts' | 'contact_blacklist' | 'none'
await sock.updateStatusPrivacy(value)
```

### Update Read Receipts Privacy

```javascript
const value = 'all' // 'none'
await sock.updateReadReceiptsPrivacy(value)
```

### Update Group Add Privacy

```javascript
const value = 'all' // 'contacts' | 'contact_blacklist'
await sock.updateGroupsAddPrivacy(value)
```

### Update Default Disappearing Mode

Ephemeral:

| Time   | Seconds     |
|--------|-------------|
| Remove | 0           |
| 24h    | 86,400      |
| 7d     | 604,800     |
| 90d    | 7,776,000   |

```javascript
const ephemeral = 86400 
await sock.updateDefaultDisappearingMode(ephemeral)
```

## 📢 Broadcast Lists & Stories

### Send Broadcast & Stories

Messages can be sent to broadcasts & stories. Add message options:

```javascript
await sock.sendMessage(
    jid,
    {
        image: {
            url: url
        },
        caption: caption
    },
    {
        backgroundColor: backgroundColor,
        font: font,
        statusJidList: statusJidList,
        broadcast: true
    }
)
```

Message body can be `extendedTextMessage`, `imageMessage`, etc., see [Media Messages](#media-messages).  

`broadcast: true` enables broadcast mode.  

`statusJidList`: list of recipients.  

Broadcast ID: `12345678@broadcast`.

### Query Recipients & Broadcast List Name

```javascript
const bList = await sock.getBroadcastListInfo('1234@broadcast')
console.log(`list name: ${bList.name}, recipients: ${bList.recipients}`)
```

## 🛠️ Custom Functions

Baileys is designed for custom functions. Add your own extensions without forking.

### Enable Debug Level in Baileys Logs

```javascript
const sock = makeWASocket({
    logger: P({ level: 'debug' }),
})
```

This will display WhatsApp messages in the console.

### How WhatsApp Communicates

> [!TIP] Learn about WhatsApp protocol with Libsignal Protocol and Noise Protocol.

Example: Track phone battery percentage. Enable logs, battery message appears:

```json
{
    "level": 10,
    "fromMe": false,
    "frame": {
        "tag": "ib",
        "attrs": {
            "from": "@s.whatsapp.net"
        },
        "content": [
            {
                "tag": "edge_routing",
                "attrs": {},
                "content": [
                    {
                        "tag": "routing_info",
                        "attrs": {},
                        "content": {
                            "type": "Buffer",
                            "data": [8,2,8,5]
                        }
                    }
                ]
            }
        ]
    },
    "msg":"communication"
}
```

`'frame'` has: `tag` (frame type), `attrs` (metadata), `content` (data).  

### Register Callback for Websocket Events

> [!TIP] See `onMessageReceived` function in `socket.ts`.

```javascript
// For 'edge_routing' tag
sock.ws.on('CB:edge_routing', (node: BinaryNode) => { })

// With id 'abcd'
sock.ws.on('CB:edge_routing,id:abcd', (node: BinaryNode) => { })

// With routing_info node
sock.ws.on('CB:edge_routing,id:abcd,routing_info', (node: BinaryNode) => { })
```

## 💡 Tips & Best Practices

- **Use Caching**: Cache group metadata for better performance.
- **Handle Reconnection**: Implement auto-reconnect for stability.
- **Manage Store**: Use a database (MongoDB, PostgreSQL) for production.
- **Error Handling**: Wrap socket calls with try-catch.
- **Rate Limits**: Respect WhatsApp limits to avoid bans—don't spam.
- **Security**: Don't share auth credentials; use environment variables.

## 📄 License

Distributed under the GPL-3.0 License. See [LICENSE](LICENSE) for more information.

---

<div align="center">
  Forked and modified by Baileys Team.  
  Baileys - Modern WhatsApp Web API with @lid to @pn Fix
</div>