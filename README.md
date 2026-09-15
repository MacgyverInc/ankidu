# Ænkidu

<sub>**Ankidu.de** - web: [ankidu.de](https://ankidu.de)</sub>

## About

Ænkidu turns Anki into a **cloud service** - your decks, cards, and review scheduling run entirely on our server, so there's **no desktop Anki process to install, launch, or keep running**.

It also supports ankiconnect too! Point your existing AnkiConnect tools, browser extensions, importers, and even AI assistants at Ænkidu over the network and they just work - your collection lives in the cloud. Spin on!

## Ænkidu - Hosted Anki Service

Ænkidu is a **process-less, multi-tenant Anki server**. It faithfully implements the familiar **AnkiConnect** JSON-RPC API (cards, decks, models, notes, search, and stats) plus a pure Anki REST HTTP API, so the tools you already use keep working - no local Anki app, no plugins required.

<!-- <a href="https://ko-fi.com/">
  <img
  src="https://storage.ko-fi.com/cdn/brandasset/v2/support_me_on_kofi_blue.png" alt="Support me on Ko-fi"
  width="275"
   />
</a> -->

> **Alpha access to the Ænkidu SaaS opens in late September 2026.** Stay tuned!

## Why is a hosted service necessary?

Anki traditionally requires a desktop process running locally for AnkiConnect clients to talk to. That's limiting - the machine has to be on, awake, and running Anki. Ænkidu ships an **always-on cloud engine** that hosts each user's collection server-side. You can use your existing AnkiConnect-compatible apps against it from anywhere, without your PC running.

## Key Features of Ænkidu

- **Process-less Anki** - no desktop Anki install and no local Anki process required

- **Drop-in AnkiConnect compatibility** - existing clients (e.g. Yomitan and other AnkiConnect tools) work unchanged

- **Faithful AnkiConnect API** - cards, decks, models, notes, search, and stats, similar behavior and quirks

- **Modern REST HTTP API too** - clean `v1/anki` endpoint for building your own integrations - /collection/1/decks, /collection/1/cards, etc

- **Multi-tenant & isolated** - every account's collections are scoped and isolated per user

- **Multiple collections per account** - each collection behaves like its own Anki profile

- **Import / export your `.colpkg`** - bring your existing collection in, take it out any time

- **Modern scheduling** - powered by Anki's current spaced-repetition scheduler

- **AI-ready via MCP** - a built-in Model Context Protocol server lets LLM clients (like ChatGPT) read and edit your collection through natural language

- **Secure by design** - OAuth 2.1 authorization and API keys

- **Always on** - your collection is available around the clock, from any device, anywhere

- **Future proof** - a maintained, cloud-native service that evolves with the Anki ecosystem

- **Fully tested** - Anki tests are run against the server to achieve the same reliability as the Desktop app

## Why Ænkidu?

There are ways to expose Anki over the network already. **What sets Ænkidu apart?**

Ænkidu doesn't just tunnel to a copy of desktop Anki - it **replaces the local Anki process entirely** with a purpose-built, multi-tenant cloud service. No fragile local setup, no "is my PC awake?" - real availability from any device at any time.

It's **faithful to AnkiConnect on purpose**: the goal is that the tools people already love keep working, while giving developers a modern REST API and AI assistants a first-class MCP interface to the same collection.

Ænkidu was named after the sumerian legend, Enkidu. But his domain was taken. It intends to invoke the image of the [battle between Gilgamesh and Enkidu from Fate](https://www.youtube.com/watch?v=Ti1TjrSg9-g)

---

## API Reference

<sub>Base URL: `https://api.ankidu.de` - Auth: `Bearer <JWT>` for REST, `keyId.secret` in the `key` field for AnkiConnect</sub>

<details>
<summary><code>GET</code> <b>/v1/anki/collections</b> — List your collections</summary>

<br>

**Auth:** `Bearer <JWT>`

**Response `200`**
```json
{
  "collections": ["1", "2", "3"]
}
```
</details>

<details>
<summary><code>GET</code> <b>/v1/anki/collections/{collectionId}/search/cards</b> — Find cards by Anki query</summary>

<br>

**Auth:** `Bearer <JWT>`

**Parameters**

| In | Name | Type | Example |
|------|--------|--------|-----------|
| path | `collectionId` | string | `1` |
| query | `query` | string | `deck:Japanese is:due` |

**Request**
```http
GET /v1/anki/collections/1/search/cards?query=deck:Japanese%20is:due
Authorization: Bearer <JWT>
```

**Response `200`**
```json
{
  "ids": ["1712345678901", "1712345678902"],
  "count": 2
}
```
</details>

<details>
<summary><code>POST</code> <b>/v1/anki/collections/{collectionId}/notes</b> — Add a note to a deck</summary>

<br>

**Auth:** `Bearer <JWT>`

**Request**
```http
POST /v1/anki/collections/1/notes
Authorization: Bearer <JWT>
Content-Type: application/json
```
```json
{
  "note": {
    "notetypeId": "1607392319495",
    "fields": ["犬", "dog"],
    "tags": ["animals", "n5"]
  },
  "deckId": "1607392319000"
}
```

**Response `200`**
```json
{
  "noteId": "1712345679001",
  "count": 1
}
```
</details>

<details>
<summary><code>POST</code> <b>/</b> — AnkiConnect-compatible JSON-RPC (point Yomitan &amp; friends here)</summary>

<br>

**Auth:** carried in the `key` field (`keyId.secret`) — no header needed

**Request — `deckNames`**
```json
{
  "action": "deckNames",
  "version": 6,
  "key": "5e7d1f8e2e04.8a1f...b22f"
}
```

**Response `200`**
```json
{
  "result": ["Default", "Japanese"],
  "error": null
}
```

**Request — `addNote`**
```json
{
  "action": "addNote",
  "version": 6,
  "key": "5e7d1f8e2e04.8a1f...b22f",
  "params": {
    "note": {
      "deckName": "Japanese",
      "modelName": "Basic",
      "fields": { "Front": "犬", "Back": "dog" },
      "tags": ["animals"]
    }
  }
}
```

**Response `200`**
```json
{
  "result": 1712345679001,
  "error": null
}
```
</details>


## Secure AnkiConnect API Key access

Ænkidu keys use an AnkiConnect-friendly `keyId.secret` bearer format, so you paste one string into any AnkiConnect client (Yomitan, etc.) and you're done. You can create and manage keys two ways:

### Option 1 - The `ankidu` CLI

The CLI logs you in securely (your password is exchanged for a short-lived token — the key itself is shown **only once**, and only its hash is ever stored server-side):

```bash
# Log in securely (opens the ankidu.de sign-in, no password stored locally)
ankidu login

# Mint an AnkiConnect key bound to a collection
ankidu keys create --name "yomitan-laptop"
# → ankidu key issued (copy it now, it won't be shown again):
#   4f9e1c3eee04.9f4e...c75a

# Manage existing keys
ankidu keys list
ankidu keys revoke 4f9e1c3eee04
```

Then point any AnkiConnect client at your Ænkidu endpoint and set the API key field to the `keyId.secret` string.

### Option 2 - The Web Dashboard

Prefer clicking? Sign in at **[ankidu.de](https://ankidu.de)** and open **Settings → API Keys**:

- **Create key** - name it, bind it to a collection, and copy the `keyId.secret` (shown once)
- **List keys** - see each key's prefix, creation date, last-used time, and status
- **Revoke** - instantly disable any key

> Notes: keys are long-lived (unlimited, or an expiry of ≥ 90 days), you can hold up to **10 active keys** per account, and the full secret is never stored or shown twice — revoke and re-mint if you lose it.


## Upcoming

- **Alpha of the Ænkidu SaaS - late September 2026**

- Expanded MCP tool coverage for AI-assisted studying

- Additional import/sync workflows

- Support for media and attachments

- Broader compatibility with AnkiConnect tools and extensions

- Encryption of collections at rest

- Collection backup system

## FAQ

**Q: Do I need to install Anki or keep a PC running?**
A: No. Ænkidu is process-less - your collection runs on our servers, so there's no desktop Anki to install or keep awake.

**Q: Will my existing AnkiConnect tools work?**
A: Yes. Ænkidu re-implements the AnkiConnect JSON-RPC API faithfully, so existing clients (like Yomitan and other AnkiConnect-based tools) work unchanged - you just point them at Ænkidu.

**Q: Can I bring my existing collection?**
A: Yes. You can import your `.colpkg`, and export it again whenever you like.

**Q: How does the AI integration work?**
A: Ænkidu exposes a Model Context Protocol (MCP) server, so LLM clients such as ChatGPT can list, add, and manage cards in your collection through natural language, secured with OAuth 2.1.

**Q: When can I try it?**
A: An alpha of the Ænkidu SaaS becomes available in **late September 2026**.

## Contact

Open a GitHub issue or email me at the org. I'm open to feature suggestions!
