![Fixing the Soil Health Tech Stack](./docs/img/fixingsoil-logo.png)

# Fixing the Soil Health Tech Stack - 2022 Hackathon
-----------------------------------------------------

1. [Overview](#overview)  
1. [Hackathon](#hackathon-final-demo)
    1. [Teams](#teams)
    2. [Contingency Planning](#contingency-planning)
    3. [Hackathon Setup and Logistics](#hackathon-setup-and-logistics)  


# Overview
----------
The goal of this hackathon is to help "fix the soil health tech stack" by writing code together that both produces and uses soils data, thereby demonstrating interoperability and improving toolsets available to the digital soil community specifically around soil sample results and related information.

This is different than a normal hackathon where teams compete to out-do one another in particular challenges.  This hackathon is about shared engineering collaboration: we are all working together toward the final demo, and the more that works the better it is for all of us.

## Components
-------------
Tools, libraries, apps, etc. for this hackathon will likely fall into 1 of three categories:
1. **Form**: Models of soil sampling data
2. **Backfill**: Populating the standardized models with past data of various forms
3. **Function**: Apps and Analysis that can consume soil samples in the standardized models and show them in a UI or chart.

## Data
-----------
We have collected soil sampling data from several labs and made it available publicly for this hackathon.  Participants can access this data during and after the hackathon in API form at https://oats1.ecn.purdue.edu/bookmarks/soil-samples.  Details on REST API access are given below.

## Technical pre-work
---------------------
We have chosen the Modus format hosted by Ag Gateway as the main soil sampling result model for this hackathon.  We've made a JSON schema which faithfully represents the originla Modus XML spec, a converter for XML into that JSON schema, and a "slim" version JSON schema with converter.  

### Modus XML Schema and Nomenclature lists: 
These can be found at the Modus Bitbucket repo here: https://bitbucket.org/modus/modus-schema/src/Version-1.0/.

### New JSON-schema version of Modus
There are three components of the JSON versio of Modus (to mirror the original XML structure): `global`, `modus-submit`, and `modus-result`.  `modus-result` uses `global` and is the form in which a soils lab would communicate lab results.  Hence it will be the focus of this hackathon.

These three components are published as JSON schemas (compiled from Typescript), and associated Typescript types.  
1. `global`
Source (TS):  https://github.com/OADA/formats/blob/master/schemas/modus/v1/global.schema.cts 
JSON Schema: https://formats.openag.io/modus/v1/global.schema.json
Typescript Types (npm): `@oada/types/modus/v1/global'

2. `modus-result`
Source (TS):  https://github.com/OADA/formats/blob/master/schemas/modus/v1/modus-result.schema.cts 
JSON Schema: https://formats.openag.io/modus/v1/modus-result.schema.json
Typescript Types (npm): `@oada/types/modus/v1/modus-result'

### Universal and Command-line Javascript Tools
A monorepo of tooling is avaiable here: https://github.com/oats-center/modus.  
1. `examples`: @modusjs/examples.  Directly import-able XML and json examples of modus (great for testing).
2. `convert`: @modusjs/convert.  Javascript library to convert between formats.  Currently supports converting a Modus XML string into Modus JSON.
3. `cli`: @modusjs/cli.  Command-line tool to perform file conversions (i.e. xml to json).

### More....
As we get more ready, we'll drop docs here to introduce them.


## API Access via OADA
----------------------
The Open Ag Data Alliance (OADA) and Trellis projects from the OATS Center at Purdue University comprise a framework for defining API's for agriculture.  We will use this API framework for reading and writing soil samples during the hackathon.  While a full exposition of OADA is beyond the scope of this document, here are basic instructions to authorize, read, and write.

### Installation and Setup
If you want to run `oada` locally for development, you need `docker`, `docker-compose`, and a non-Windows OS (Mac or Linux).  Follow the instructions found at https://github.com/oada/server.  This will result in you having a running instance of OADA and a token you can use to make requests.


### GET /bookmarks
------------------
In OADA, URL's act sort of like a filesystem of resources rooted at path `/bookmarks`.  Each user has their own bookmarks, and the token you pass determines which user your request is referencing.  Send a different user's token and you'll get a different `/bookmarks` resource.

After install, your bookmarks is "empty":
```http
GET /bookmarks
Host: localhost
Authorization: Bearer <token>
```
Response:
```json
{
  "_id": "resources/default:resources_bookmarks_321",
  "_rev": 1,
  "_type": "application/vnd.oada.bookmarks.1+json",
  "_meta": {
    "_id": "resources/default:resources_bookmarks_321/_meta",
    "_rev": 1
  }
}
```
That tells you the bookmarks resource is JSON, it is of `content-type` `application/vnd.oada.bookmarks.1+json`, and it has a `_meta` document (i.e. another JSON document storing arbitrary data _about_ the resource).  The resource at `/bookmarks` has a "canonical" id of `resources/default:resources_bookmarks_321`.  Also, it is currently on its first revision `_rev: 1`.

### Setup the Soil Samples base API
------------------------------------------
**Please note that this API is not set in stone and may not be the final API from the hackathon.**
For the purposes of helping understand how the API works, we will build it by hand in this section.  

This basically means we are creating the "tree" of resources that would map to the URL `/bookmarks/trellisfw/asns`.  i.e. `/bookmarks` will be a resource with a `trellisfw` key, and `/bookmarks/trellisfw` will be a resource with an `asns` key.

We'll start at the bottom: let's make a resource to hold the ASN's (note the `Content-Type` we're using):

```http
POST /resources
Host: localhost
Content-Type: "application/vnd.trellisfw.asns.1+json"
Authorization: Bearer <token>

{}
```
Response (headers only, no body):
```http
Status: 200 Ok
Content-Location: "/resources/1pnl9KMIw9Dbt9WgC5XHWQIikrE"
```

We now have an empty resource with `_id` `resources/1pnl9KMIw9Dbt9WgC5XHWQIikrE` (i.e. the content-location with the leading `/` removed).  For those who are interested, the part after `resources/` is a K-Sortable Universally Unique Id [`ksuid`](https://github.com/segmentio/ksuid).  It's an extremely handy way of generating random id's: the first half of it is the current timestamp, and the second half is a random string that ensures uniqueness.  If you have a list of ksuids, you can sort them lexically which will sort them by creation time.  Any time a random string is generated in OADA (i.e. on a POST), it is a `ksuid`.

Next, we need to make a resource for the `trellisfw` part of the `/bookmarks/trellisfw/asns` URL, and the body of that will contain an `asns` key which links to our new resource.  **A `link` in OADA is just an object with an `_id` key**.  
```http
POST /resources
Host: localhost
Content-Type: "application/vnd.trellisfw.1+json"
Authorization: Bearer <token>

{
  "asns": { "_id": "resources/1pnl9KMIw9Dbt9WgC5XHWQIikrE" }
}
```
Response (headers only, no body):
```http
Status: 200 Ok
Content-Location: "/resources/1pnoUVMJ7nsQpdmb91CcIG781sv"
```

Now, we have a resource containing an `asns` key which links to our `asns` resource.  This means that you can append the key to the end of a URL that ends at this resource, and OADA will take you to the linked resource.  i.e. `/resources/1pnoUVMJ7nsQpdmb91CcIG781sv/asns` will go to the first resource we created.  **You can append any keys in a resource to the URL (nested keys joined with `/`) and get the value at that key.**

Finally, let's link the trellisfw resource into bookmarks to complete the setup.  We're going to switch from POST to PUT. **All writes to OADA deep merge the "body" of the request into the existing resource,** and increment the `_rev` on the resource.  In fact, a POST is actually just a PUT that appends a `ksuid` on the end of the URL.
```http
PUT /bookmarks
Host: localhost
Content-Type: "application/vnd.oada.bookmarks.1+json"
Authorization: Bearer <token>

{
  "trellisfw": { "_id": "resources/1pnoUVMJ7nsQpdmb91CcIG781sv" }
}
```

That's it!  Now we have an API for ASN's all set and ready to roll, and we know how to read, write, and link together resources.  If you do a `GET /bookmarks/trellisfw/asns`, now you'll get the empty resource that will hold the ASN's.

### Create an ASN
-----------------

For this tutorial, we'll assume an ASN looks like this for simplicity (obviously the real ASN's will have all kinds of other information):
```json
{
  "shipdate": "2021-03-15"
}
```

We would like to POST this ASN to our `/bookmarks/trellisfw/asns` list.  However, if we put all our ASN's into this same resource, over time it is going to become very large.  Therefore, we often choose a _canonical indexing_ structure to group things in a list.  Additional indexes can be created, but the canonical one is assured to be the place you always write the initial link.

The natural indexing for ASN's is by ship date.  Therefore, let's create a `day-index` bucket to put this ASN into.  Our final URL where we will POST this ASN will be: `/bookmark/trellisfw/asns/day-index/2021-03-15`.  We'll need to create this resource which means we need to know a content type with which to create the resource.  By convention, the content-type of any index resource is the same as the original list (`application/vnd.trellisfw.asns.1+json`).  This is because an index is also a list, just a subset of the original list.  

You can create this by hand, or we have a convenient Javascript library that can simplify this. Executing the calls to build API trees like this has a tricky concurrency gotcha: this is handled in the `@oada/client` library.  Not to mention, it can be tedious to do this for large trees.

To use the `tree put` of `@oada/client`, let's first create a JSON tree that holds the structure and `content-types` for our API.  Every level that is supposed to be its own resource needs to have the `_type` specified:
```javascript
const tree = {
  bookmarks: {
    _type: "application/vnd.oada.bookmarks.1+json",
    trellisfw: {
      _type: "application/vnd.trellisfw.1+json",
      asns: {
        _type: "application/vnd.trellisfw.asns.1+json",
        'day-index': {
          '*': {
            _type: "application/vnd.trellisfw.asns.1+json",
            _rev: 0,
            '*': {
              _type: "application/vnd.trellisfw.asn.porkhack.1+json",
              _rev: 0
} } } } } } };
```
Note that every level is its own resource (i.e. has an `_type`) except `day-index`.  This means that when you GET `/bookmarks/trellisfw/asns`, the response will include every day that you have an ASN with a `shipdate` of that day.  Also, the `'*'` means that it will use the same types for each day.  The bottom level is the actual ASN's, for those we'll use `_type` `application/vnd.trellisfw.asn.porkhack.1+json`.  We'll get into why the `_rev: 0` is in there when we discuss live streaming change feeds below.  For now, just make sure it's there.

Now, you'll want to `npm install @oada/client` and we can go ahead and POST our dummy ASN all in one go and it will ensure the path exists (assuming you have your domain, token, and that tree in variables):
```javascript
import { connect } from '@oada/client'

(async () => {
  const oada = await connect({domain,token});
  const { headers } = await oada.post({ 
    path: "/bookmarks/trellisfw/asns/day-index/2021-03-15",
    data: { shipdate: "2021-03-15" }, // This is the ASN
    tree
  });
  console.log(`Created new ASN with _id ${headers['content-location'].slice(1)}`);
  await oada.disconnect();
})()
```

And voila!  Now you have an ASN in your list, feel free to go look around your new API tree and see the ASN.

### Changes
---------------------------

The real power of Trellis comes in its ability to stream an ordered change feed from an arbitrary sub-tree of resources to any destination you want.  If you have a microservice or app, just set a watch on a resource via websockets.  If you want to trigger an external REST service or serverless functions, simply set a webhook on a resource and it will call your API whenever the resource changes.

To see how this works, first we can just look at what a `change` is.  Let's look at the latest change to our ASN resource:

```http
GET /bookmarks/trellisfw/asns/day-index/2021-03-15/1pp0dS2wxFgnYmcMW68pqLZLRFb/_meta/_changes/2
Host: localhost
Authorization: Bearer <token>
```
Resources
```json
[
  {
    "resource_id": "resources/1pp0dfopoHbTPeVlGj2GSFQpIpT",
    "path": "",
    "body": {
      "shipdate": "2021-03-15",
      "_meta": {
        "modifiedBy": "users/default:users_sam_321",
        "modified": 1615864812.55,
        "_rev": 2
      },
      "_rev": 2
    },
    "type": "merge"
  }
]
```

This rev involved a single change (there is 1 thing in the array), it happened to the `resource_id` listed (our asns resource), that resource_id is at the end of your "watch" point (i.e. where you got `_meta/_changes/2` from), and you can see who modified it and when.  Most importantly, if you strip out all the keys of `body` which start with an underscore `_`, you get the original PUT body:
```json
{
  "shipdate": "2021-03-15"
}
```

If you received this change, you would know that it changed (upserted) the shipdate key on this resource to be the value `2021-03-15`.  We call this an "idempotent merge", because if you applied this same change to the resource over and over, you'd still end up with the same resulting state.  This "idempotent merge" represents the change necessary to transition the resource from `_rev` 1 (its creation) to `_rev` 2 (insert of first data).  If you have a remote copy of this resource at `_rev` 1, just do the same merge to your copy and you get a matching `_rev` 2.

### Change Trees and Versioned Links
------------------------------------
We have a problem: we indexed all our ASN's into `day-index` buckets, but we'd like to just set a `watch` on the whole ASN's list and get the changes streamed to us.  In other words, we know we want all ASN changes, regardless of which ASN they belong to, but in order to do that we'd need to set a separate "watch" on every ASN.

OADA to the rescue.  We want to see a change on just the top-level `/bookmarks/trellisfw/asns` resource that contains the changes of any child ASN resources on down the tree.  Remember those `_rev: 0`'s we had in the tree earlier?  **Adding the `_rev` to the link makes it a _versioned link_**.  OADA fills in the value of the `_rev` as the latest known `_rev` on the linked resource.  Whenver the linked resource changes, it will eventually update the `_rev` on that link.  And here's the magic: since that link's content is actually part of the parent resource, OADA will also update the `_rev` on the parent since it's content changed.  This allows you to batch together child changes and listen for them at any arbitrary node in the tree.  All you need to do is make sure your links are versioned.

Let's see this in action.  Since our links were already versioned when we created our ASN resource, we can just look at the change for the latest `_rev` on the `/bookmarks/trellisfw/asns` resource and we should see the underlying addition of the ASN!

To figure out the "current" rev of the parent, do:
```http
GET /bookmarks/trellisfw/asns/_rev
Host: localhost
Authorization: Bearer <token>
```
Response
```json
7
```
(Note, my latest version is 7, but yours may be different.  Use yours in the subsequent calls)

Now, let's look at the change document for verion 7:
```http
GET /bookmarks/trellisfw/asns/_meta/_changes/7
Host: localhost
Authorization: Bearer <token>
```
Response:
```json
[
  {
    "resource_id": "resources/1pnl9KMIw9Dbt9WgC5XHWQIikrE",
    "path": "",
    "body": {
      "day-index": {
        "2021-03-15": {
          "_rev": 2
        }
      },
      "_meta": {
        "modifiedBy": "system/rev_graph_update",
        "modified": 1615864812.593,
        "_rev": 7
      },
      "_rev": 7
    },
    "type": "merge"
  },
  {
    "resource_id": "resources/1pp0dfzhWGD8fsyiJp1JLiMsfHX",
    "path": "/day-index/2021-03-15",
    "body": {
      "1pp0dS2wxFgnYmcMW68pqLZLRFb": {
        "_rev": 2
      },
      "_meta": {
        "modifiedBy": "system/rev_graph_update",
        "modified": 1615864812.571,
        "_rev": 2
      },
      "_rev": 2
    },
    "type": "merge"
  },
  {
    "resource_id": "resources/1pp0dfopoHbTPeVlGj2GSFQpIpT",
    "path": "/day-index/2021-03-15/1pp0dS2wxFgnYmcMW68pqLZLRFb",
    "body": {
      "shipdate": "2021-03-15",
      "_meta": {
        "modifiedBy": "users/default:users_sam_321",
        "modified": 1615864812.55,
        "_rev": 2
      },
      "_rev": 2
    },
    "type": "merge"
  }
]
```
Whoa, that's a lot.  Notice first that there are 3 items in the array: i.e. the move from `_rev` 6 to `_rev` 7 for this resource involved 3 separate underlying changes.  The "leaf" change (i.e. the original change down at the leaf node which is our newly-added ASN) is the last one. Notice it is the same as the one we looked at above, except that the `path` key is different.  In this case, it means "the path to this changed resource from the current 'root' resource is `/day-index/2021-03-15/1pp0dS2wxFgnYmcMW68pqLZLRFb`.  You can see the `shipdate` data in that change.

The next one up is the change to path `/day-index/2021-03-15` which just updated the `_rev` on the versioned link in that day-index resource.  Finally, the top one is the "root" resource that you are "watching" for changes, and it is just an update to the `_rev` of the day-index resource.

This makes it incredibly easy to react to changes of an arbitrarily complex API tree: just watch for changes, filter the change docs to the resources you care about using the paths, and then grab the underlying data that matters to you.

### Watching for Changes
------------------------
Polling for changes like we did above works just fine, but it's not efficient.  You can actually ask OADA to just push you a stream of changes starting from a `_rev` of your choice on any resource you want.  You can ask it to do this via a websocket, or as a webhook.  You can even ask it to replay the change automatically at a remote OADA service as a sort of OADA-aware webhook (i.e. performing a one-way sync).

To see this work, we'll just use your browser's console to open a websocket to your OADA server.  Note that you'll have to figure out how to get your browser to trust your self-signed `localhost` SSL certificate if you want to try this.  If you are using Chrome or Brave, this is harder than it used to be.  Go to either chrome://flags/#allow-insecure-localhost or brave://flags/#allow-insecure-localhost and check the box to allow your browser to accept the localhost cert.  Then, to make sure it works, go to this URL in your browser: https://localhost/.well-known/oada-configuration.  If you see some JSON show up, you're all set.

To open the websocket in your browser console and set the watch, just paste this into your browser console (**Replace &lt;token&gt; with your token**):
```javascript
var w = new WebSocket('wss://localhost');
   w.onopen = function(evt) { console.log('open!!') };
  w.onerror = function(evt) { console.log('ERROR! evt = ', evt); }
w.onmessage = function(evt) { console.log('message! ', JSON.parse(evt.data)) };
w.send(JSON.stringify({
  requestId: '1',
  method: 'watch',
  headers: { authorization: 'Bearer <token>' },
  path: '/bookmarks/trellisfw/asns',
}));
```
Now, go ahead and re-run your javascript code from above to add another new ASN for today's day-index.  You'll see the change appear as it is pushed from your OADA service to your console.

To set a watch in code using `@oada/client`, you can simply do:
```javascript
const oada = await connect({domain, token});
const requestId = await oada.watch({
  path: '/bookmarks/trellisfw/asns',
  rev: 1, // optional, the _rev from which to resume the watch
  watchCallback: d => {
    console.log(d); // This runs on every change
  },
})
```
To unwatch a resource, use the unwatch request with the returned `requestId` above:
```javascript
const response = await oada.unwatch(requestId);
```

That should be enough to get anyone started working with the OADA API for this hackathon.
