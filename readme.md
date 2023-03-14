# ChatGPT API <!-- omit in toc -->

> Node.js client for the official [ChatGPT](https://openai.com/blog/chatgpt/) API. Fork of the original to work better with CommonJS and TypeScript!

- [Intro](#intro)
- [Updates](#updates)
- [Install](#install)
- [Usage](#usage)
  - [Usage - ChatGPTAPI](#usage---chatgptapi)
  - [Usage - ChatGPTUnofficialProxyAPI](#usage---chatgptunofficialproxyapi)
    - [Reverse Proxy](#reverse-proxy)
    - [Access Token](#access-token)
- [Docs](#docs)
- [Compatibility](#compatibility)
- [Credits](#credits)
- [License](#license)

## Intro

This package is a Node.js wrapper around [ChatGPT](https://openai.com/blog/chatgpt) by [OpenAI](https://openai.com). TS batteries included. ✨

<p align="center">
  <img alt="Example usage" src="/media/demo.gif">
</p>

## Updates

<details open>
<summary><strong>March 1, 2023</strong></summary>

<br/>

The [official OpenAI chat completions API](https://platform.openai.com/docs/guides/chat) has been released, and it is now the default for this package! 🔥

| Method                      | Free?  | Robust?  | Quality?                |
| --------------------------- | ------ | -------- | ----------------------- |
| `ChatGPTAPI`                | ❌ No  | ✅ Yes   | ✅️ Real ChatGPT models |
| `ChatGPTUnofficialProxyAPI` | ✅ Yes | ☑️ Maybe | ✅ Real ChatGPT         |

**Note**: We strongly recommend using `ChatGPTAPI` since it uses the officially supported API from OpenAI. We may remove support for `ChatGPTUnofficialProxyAPI` in a future release.

1. `ChatGPTAPI` - Uses the `gpt-3.5-turbo-0301` model with the official OpenAI chat completions API (official, robust approach, but it's not free)
2. `ChatGPTUnofficialProxyAPI` - Uses an unofficial proxy server to access ChatGPT's backend API in a way that circumvents Cloudflare (uses the real ChatGPT and is pretty lightweight, but relies on a third-party server and is rate-limited)

</details>

<br/>

If you run into any issues, we do have a pretty active [Discord](https://discord.gg/v9gERj825w) with a bunch of ChatGPT hackers from the Node.js & Python communities.

Lastly, please consider starring this repo and <a href="https://twitter.com/transitive_bs">following the original creator on twitter <img src="https://storage.googleapis.com/saasify-assets/twitter-logo.svg" alt="twitter" height="24px" align="center"></a> to help support the project.

Thanks && cheers,
[Travis](https://twitter.com/transitive_bs)


## Install

```bash
npm install chatgpt-artem
```

Make sure you're using `node >= 18` so `fetch` is available (or `node >= 14` if you install a [fetch polyfill](https://github.com/developit/unfetch#usage-as-a-polyfill)).

## Usage

To use this module from Node.js, you need to pick between two methods:

| Method                      | Free?  | Robust?  | Quality?                |
| --------------------------- | ------ | -------- | ----------------------- |
| `ChatGPTAPI`                | ❌ No  | ✅ Yes   | ✅️ Real ChatGPT models |
| `ChatGPTUnofficialProxyAPI` | ✅ Yes | ☑️ Maybe | ✅ Real ChatGPT         |

1. `ChatGPTAPI` - Uses the `gpt-3.5-turbo-0301` model with the official OpenAI chat completions API (official, robust approach, but it's not free). You can override the model, completion params, and system message to fully customize your assistant.

2. `ChatGPTUnofficialProxyAPI` - Uses an unofficial proxy server to access ChatGPT's backend API in a way that circumvents Cloudflare (uses the real ChatGPT and is pretty lightweight, but relies on a third-party server and is rate-limited)

Both approaches have very similar APIs, so it should be simple to swap between them.

**Note**: We strongly recommend using `ChatGPTAPI` since it uses the officially supported API from OpenAI. We may remove support for `ChatGPTUnofficialProxyAPI` in a future release.

### Usage - ChatGPTAPI

Sign up for an [OpenAI API key](https://platform.openai.com/overview) and store it in your environment.

```ts
import { ChatGPTAPI } from 'chatgpt'

async function example() {
  const api = new ChatGPTAPI({
    apiKey: process.env.OPENAI_API_KEY
  })

  const res = await api.sendMessage('Hello World!')
  console.log(res.text)
}
```

You can override the default `model` (`gpt-3.5-turbo-0301`) and any [OpenAI chat completion params](https://platform.openai.com/docs/api-reference/chat/create) using `completionParams`:

```ts
const api = new ChatGPTAPI({
  apiKey: process.env.OPENAI_API_KEY,
  completionParams: {
    temperature: 0.5,
    top_p: 0.8
  }
})
```

If you want to track the conversation, you'll need to pass the `parentMessageId` like this:

```ts
const api = new ChatGPTAPI({ apiKey: process.env.OPENAI_API_KEY })

// send a message and wait for the response
let res = await api.sendMessage('What is OpenAI?')
console.log(res.text)

// send a follow-up
res = await api.sendMessage('Can you expand on that?', {
  parentMessageId: res.id
})
console.log(res.text)

// send another follow-up
res = await api.sendMessage('What were we talking about?', {
  parentMessageId: res.id
})
console.log(res.text)
```

You can add streaming via the `onProgress` handler:

```ts
const res = await api.sendMessage('Write a 500 word essay on frogs.', {
  // print the partial response as the AI is "typing"
  onProgress: (partialResponse) => console.log(partialResponse.text)
})

// print the full text at the end
console.log(res.text)
```

You can add a timeout using the `timeoutMs` option:

```ts
// timeout after 2 minutes (which will also abort the underlying HTTP request)
const response = await api.sendMessage(
  'write me a really really long essay on frogs',
  {
    timeoutMs: 2 * 60 * 1000
  }
)
```

If you want to see more info about what's actually being sent to [OpenAI's chat completions API](https://platform.openai.com/docs/api-reference/chat/create), set the `debug: true` option in the `ChatGPTAPI` constructor:

```ts
const api = new ChatGPTAPI({
  apiKey: process.env.OPENAI_API_KEY,
  debug: true
})
```

We default to a basic `systemMessage`. You can override this in either the `ChatGPTAPI` constructor or `sendMessage`:

```ts
const res = await api.sendMessage('what is the answer to the universe?', {
  systemMessage: `You are ChatGPT, a large language model trained by OpenAI. You answer as concisely as possible for each responseIf you are generating a list, do not have too many items.
Current date: ${new Date().toISOString()}\n\n`
})
```

Note that we automatically handle appending the previous messages to the prompt and attempt to optimize for the available tokens (which defaults to `4096`).

### Usage - ChatGPTUnofficialProxyAPI

The API for `ChatGPTUnofficialProxyAPI` is almost exactly the same. You just need to provide a ChatGPT `accessToken` instead of an OpenAI API key.

```ts
import { ChatGPTUnofficialProxyAPI } from 'chatgpt'

async function example() {
  const api = new ChatGPTUnofficialProxyAPI({
    accessToken: process.env.OPENAI_ACCESS_TOKEN
  })

  const res = await api.sendMessage('Hello World!')
  console.log(res.text)
}
```

See [demos/demo-reverse-proxy](./demos/demo-reverse-proxy.ts) for a full example:

```bash
npx tsx demos/demo-reverse-proxy.ts
```

`ChatGPTUnofficialProxyAPI` messages also contain a `conversationid` in addition to `parentMessageId`, since the ChatGPT webapp can't reference messages across different accounts & conversations.

#### Reverse Proxy

You can override the reverse proxy by passing `apiReverseProxyUrl`:

```ts
const api = new ChatGPTUnofficialProxyAPI({
  accessToken: process.env.OPENAI_ACCESS_TOKEN,
  apiReverseProxyUrl: 'https://your-example-server.com/api/conversation'
})
```

Known reverse proxies run by community members include:

| Reverse Proxy URL                                | Author                                       | Rate Limits       | Last Checked |
| ------------------------------------------------ | -------------------------------------------- | ----------------- | ------------ |
| `https://bypass.duti.tech/api/conversation`        | [@acheong08](https://github.com/acheong08)   | 5 req / 10 seconds by IP | 3/11/2023    |
| `https://gpt.pawan.krd/backend-api/conversation` | [@PawanOsman](https://github.com/PawanOsman) | ?                 | 2/19/2023    |

Note: info on how the reverse proxies work is not being published at this time in order to prevent OpenAI from disabling access.

#### Access Token

To use `ChatGPTUnofficialProxyAPI`, you'll need an OpenAI access token from the ChatGPT webapp. To do this, you can use any of the following methods which take an `email` and `password` and return an access token:

- Node.js libs
  - [ericlewis/openai-authenticator](https://github.com/ericlewis/openai-authenticator)
  - [michael-dm/openai-token](https://github.com/michael-dm/openai-token)
  - [allanoricil/chat-gpt-authenticator](https://github.com/AllanOricil/chat-gpt-authenticator)
- Python libs
  - [acheong08/OpenAIAuth](https://github.com/acheong08/OpenAIAuth)

These libraries work with email + password accounts (e.g., they do not support accounts where you auth via Microsoft / Google).

Alternatively, you can manually get an `accessToken` by logging in to the ChatGPT webapp and then opening `https://chat.openai.com/api/auth/session`, which will return a JSON object containing your `accessToken` string.

Access tokens last for days.

**Note**: using a reverse proxy will expose your access token to a third-party. There shouldn't be any adverse effects possible from this, but please consider the risks before using this method.

## Docs

See the [auto-generated docs](./docs/classes/ChatGPTAPI.md) for more info on methods and parameters.


## Compatibility

- This package is a fork of the original chatgpt npm package, made to work with CommonJS and Typescript.
- This package supports `node >= 14`.
- This module assumes that `fetch` is installed.
  - In `node >= 18`, it's installed by default.
  - In `node < 18`, you need to install a polyfill like `unfetch/polyfill` ([guide](https://github.com/developit/unfetch#usage-as-a-polyfill)) or `isomorphic-fetch` ([guide](https://github.com/matthew-andrews/isomorphic-fetch#readme)).
- If you want to build a website using `chatgpt`, we recommend using it only from your backend API

## Credits

- Thanks to the [original author of the library](https://github.com/transitive-bullshit)! 
- Huge thanks to [@waylaidwanderer](https://github.com/waylaidwanderer), [@abacaj](https://github.com/abacaj), [@wong2](https://github.com/wong2), [@simon300000](https://github.com/simon300000), [@RomanHotsiy](https://github.com/RomanHotsiy), [@ElijahPepe](https://github.com/ElijahPepe), and all the other contributors 💪
- The original browser version was inspired by this [Go module](https://github.com/danielgross/whatsapp-gpt) by [Daniel Gross](https://github.com/danielgross)
- [OpenAI](https://openai.com) for creating [ChatGPT](https://openai.com/blog/chatgpt/) 🔥

## License

MIT © [Travis Fischer](https://transitivebullsh.it)

If you found this project interesting, please consider [sponsoring the creator](https://github.com/sponsors/transitive-bullshit) or <a href="https://twitter.com/transitive_bs">following them on twitter <img src="https://storage.googleapis.com/saasify-assets/twitter-logo.svg" alt="twitter" height="24px" align="center"></a>
