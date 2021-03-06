---
title: Anyfetch provider
subtitle: Data, datas, moar datas!
layout: doc
---

> Before reading about creating a new provider, you may be interested in [the hello world tutorial](/guides/hello-world.html)

Creating a provider is quite simple. Even better, most of the anyFetch default providers are open-source: you can take a [quick peek at them](https://github.com/search?q=%40Papiel+provider) if you have any trouble.

Basically, a provider is a simple gateway between some data-source (Dropbox, a folder on your computer, Gmail) and the Fetch API. You just need to take the data and send them using the `/documents` and `/documents/:id/file` endpoints.

## Sending document
Let's say we have a file on our local drive we want to provide.
We also have a token to communicate with the Fetch API.

The first step will then be to create a new document. To do this, we'll send the following JSON to `/documents`:

```json
{
	"identifier": "some-unique-identifier",
	"document_type": "file",
	"metadatas": {
		"path": "/home/laptop/document.txt"
	},
	"no_hydration": true
}
```

* `identifier` is a unique identifier you can choose, which can later be used to retrieve or update the document.
* `document_type` is set to `file` (some providers may use semantic information, for instance "customer". Here, we chose not to deal with the complexity: hydration stack will retrieve relevant datas).
* `no_hydration` is important. For now, we've not sent any datas, so we want to skip hydration until sending the real file.

Although not mandatory, we also chose to send `metadatas.path` to improve search relevance and help the hydration getting started the right way.

Alright. Anyfetch should reply with 200 and informations about our document.
Now we can send the file, using a standard multipart POST request including the file (in `file` key) and our previous `identifier`. This will be a multipart POST call on `/documents/identifier/some-unique-identifier/file` (note you can also ping `/documents/:id/file` if you want shorter url).

If everything went well, we'll get `204 No Content` -- our document was stored, and hydration has begun.

## OAuth 2.0
> You should read [how to use a provider](/guides/using/provider.md) for more details about authentication.

Before being able to send datas, you need to get a provider token.
The user need to click on the name of your provider in the frontend. He'll then be redirected to the page you registered for initial setup, with a `?code` parameter. Setup everything you need to access your datas (maybe there will be OAuth on the other side too, maybe you need to ask for some configuration).

You can then initiate the OAuth flow by trading the code for an `access_token`.
To do so, send to `http://settings.anyfetch.com/oauth/token` the following values:

```javascript
{
    client_id: appId,
    client_secret: appSecret,
    redirect_uri: redirect_uri,
    code: code,
    grant_type: 'authorization_code',
}
```

## Lib
Here at anyFetch, we use node.js for our providers to improve latency and send multiple files at once. You can use the [anyFetch](https://npmjs.org/package/anyfetch) library from npm, or [anyFetch Provider](https://npmjs.org/package/anyfetch-provider). You'll find additional documentation directly on those packages.
