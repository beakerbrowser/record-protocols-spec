# Record Protocols Spec

Record Protocols are a standard for building interoperable applications on the Web. Its features include:

 - Strongly-enforced data schemas,
 - User-friendly metadata, and
 - Fine-grained permissioning.

Record Protocols are an alternative to self-describing schemas such as JSON-LD and RDF. Self-describing schemas work by encoding every attribute as a URL (eg `bob["foaf.com/name"]`). This solves the problem of ambiguously-defined attributes, but does not provide a solution to overall schema definition (aka "which attributes should be used?") or a solution to security-critical decisions (aka "how should access be described in permission prompts?"). Self-describing schemas are also developer hostile &mdash; nobody wants to reference values by URL-attributes.

Record Protocols instead work by enforcing schema definitions which have been published on the Web. Applications ask the browser to import the schema definitions and then interact with the browser's APIs to read/write data under those definitions. By sharing the globally-published schemas, applications can ensure interoperability, and the browser can ensure accurate metadata and safe permissioning.

### Background

For more information on the background of this spec, open the details below:

<details>

The [Dat Identities Spec](https://github.com/beakerbrowser/dat-identities-spec) establishes that:

 - Identities are dat sites
 - User data is files on the dat sites
 - The browser will provide UIs and APIs to help manage access to the identities by apps. This will include a "sign in" metaphor which provides read & write access to the data.

This spec is about a next step: data semantics.

#### The data semantics challenge

Browsers need high-level semantics in order to give the user a clear sense of what applications are doing. For example, users need to be told "This app wants to manage your Fritter contacts" rather than "This app wants write-access to /data/fritter/contacts."

This is important! Users need to understand what their applications are doing to their data and they need to be able to make informed decisions about permissions. Dat-verse apps also need to be able to share data between themselves, and that coordination is quite important. An app needs to be able to look in a dat profile and say, "Yes I understand this data, let's go!" or "No, this is foreign to me." Ideally this will happen with minimal kludge or bugginess.

#### What's wrong with files?

Files are very powerful and simple, but without metadata they aren't user-friendly. They include very little information about what they contain or what purpose they serve. They have no way to describe possible actions except at the "file" or "blob" level; you can describe access in terms of reading or writing chunks of the file, but you can't describe access in terms of modifying the objects or object-relationships it contains. For that, you really need higher level semantics.

Coordination between apps is also a big challenge. When two apps are trying to interact with the same data, you get complicated questions about trust and correctness. You have to ask: 

 - Do the apps totally understand each others' data? 
 - Is it possible a misunderstanding could create bugs? 
 - How could the shared ownership of the spec be gamed in order to attack the user? 
 - How do we coordinate changes to the schemas which naturally occur as each app matures at their own pace?
 
Historically, these kinds of questions have been answered by using standards, but standards processes can be extremely slow and irritating to developers. We want to build apps, not committees! So what's the strategy for solving cross-app coordination?

#### Could the browser do it?

In winter of 2017/18, we proposed adding high-level data semantics to the browser itself. We would create a set of standard data formats, schemas, and APIs which everybody shares. This was somewhat reminiscent of Schema.org, in that it would try to create "one entology to rule them all."

Access would be mediated by Web APIs that wrap the identity-dat's filesystem. For instance:

```js
var user = (await UserSession.fetch()).profile

await user.feed.list()
await user.feed.post({text: 'Hello world!'})

await user.friends.follow('dat://bob.com')

await user.photoAlbums.list()
await user.photoAlbums.create()
// etc
```

Because the browser managed the semantics, this solution made it very easy to explain to the user what apps are trying to do. If the app wanted to add photos to your photo album, it had to use the `photoAlbums` API, and the browser knew exactly what permission to ask from the user.

Perhaps unsurprisingly, this proposal got a bad reception. Developers pointed out that this would politically centralize the development of apps, because the schemas & formats would all go through the browser. It would effectively politicize everything and put the standards bodies (led by the browsers) at the helm. We thought the convenience of the builtin APIs and a focus on completeness would offset that concern, but were resoundingly told no.

After that a bad reception, I started to think about how we could maintain the good parts while solving the political centralization. What if we could describe the data semantics and permissioning in userland? That idea has ultimately led to this spec.

</details>

### Requirements

|Domain|Requirement|
|-|-|
|**Data&nbsp;semantics**|Data **MUST** be stored as files on a dat.|
||Data **MUST** be identified to the user with high-level semantics. (What records the files represent rather than the files' paths.)|
||Applications **MUST** have an interoperable understanding of data.|
|**Application&nbsp;access&nbsp;control**|Applications **MUST** be able to share data with each other.|
||Applications **MUST** request permission to read or write data.|
||Permissions **MUST** be easy for the user to understand.|
||Permissions **MUST** be fine-grained enough to control the records within files.|

### Definitions

|Term|Definition|
|-|-|
|**Record&nbsp;Protocol**|A set of schema definitions published at a Web address which describe the data-model semantics and permissions.|
|**Record**|A data object contained in a JSON file.|
|**Recordset**|A collection of records contained in a dat. It may refer to all the records in the dat, or it may refer to the records under a specific protocol. For instance, the "unwalled.garden recordset" refers to the records which use the "unwalled.garden protocol."|

## Record Protocol definitions

Record Protocols are identified by a domain name. They are expected to publish a set of definition-files in a dat under that domain.

### Dat type

Record Protocol definition dats must include the [`recordproto`](https://github.com/beakerbrowser/dat-types-spec#recordproto) type. This type must be set manually by the definition-dat's author.

### Definition files

A Record Protocol dat must follow this file structure:

```
/dat.json        - Standard metadata about this site.
/proto.json      - Information about the record protocol.
/schemas/*.json  - Individual JSON-Schema definitions.
*.js             - (optional) API modules for accessing the recordset.
```

### The `proto.json` file

TODO- this file should provide high level descriptions of the recordset, describe how/when each schemas is used, and outline the different permissions which can be requested by apps.

### JSON-Schema files

The schema-definition files found under `/schemas/*.json` are [JSON Schema draft-07](https://json-schema.org) files.

### API modules

Optionally (but recommended) a Record Protocol can include javascript modules for consuming apps to import, providing high level APIs for accessing the recordset. See the [unwalled.garden record protocol](https://github.com/beakerbrowser/unwalled.garden) for an example of this.

## Using Record Protocols

### Dat type

Dats which store records using record protocols must include the [`recordset`](https://github.com/beakerbrowser/dat-types-spec#recordset) type. This type will be set automatically by the browser when records are first written to the dat.

### Importing the protocol

Applications use Record Protocols by pointing to the domain name (TODO- how?). This directs the browser to download the definition files and read their instructions for managing the recordset.

Once loaded and activated, the browser creates a dedicated directory for the recordset in a target dat. This directory must follow the following path:

```
/records/{protodomain}
```

For example, a Record Protocol at `dat://fooproto.com` would be assigned the `/records/fooproto.com` folder.

### Web APIs

TODO- web apis for using record protocols
