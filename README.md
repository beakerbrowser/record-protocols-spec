# Record Protocols Spec

---

**Status**: Experimental! This spec is still being drafted and should not be taken too seriously yet.

---

Record Protocols are a standard for building interoperable applications on the Web. Its features include:

 - Strongly-enforced data schemas,
 - User-friendly metadata, and
 - Fine-grained permissioning.

Record Protocols enforce schema definitions which have been published on the Web. Applications ask the browser to import the schema definitions and then interact with the browser's APIs to read/write data under those definitions. By sharing the globally-published schemas, applications can ensure interoperability, and the browser can ensure accurate metadata and safe permissioning.

### Example Flow

An application wants to sign into the user's profile-dat using the [`unwalled.garden` Record Protocol](https://github.com/beakerbrowser/unwalled.garden). It initiates the signin flow with this code:

```js
// Request access to the 'unwalled.garden' dats and contacts records
var session = await UserSession.get()
await session.requestSignin({
  records: [{
    url: 'unwalled.garden',
    permissions: {
      dats: ['read', 'create', 'delete'],
      contacts: ['read', 'create', 'update', 'delete']
    }
  }]
})
```

Beaker will fetch the [`dat://unwalled.garden/recordproto.json`](https://github.com/beakerbrowser/unwalled.garden/blob/master/recordproto.json) file, which looks something like this:

```json
{
  "version": 1.0,
  "title": "Unwalled Garden Record Protocol",
  "description": "Common data types for Web browser applications",
  "records": {
    "dats": {
      "schema": "/schemas/dat.json",
      "permissions": {
        "read": "Read your published dats",
        "create": "Publish new dats to your profile",
        "delete": "Unpublish dats from your profile"
      }
    },
    "contacts": {
      "schema": "/schemas/profile.json",
      "permissions": {
        "read": "Read your contacts",
        "create": "Create new contacts",
        "update": "Modify your existing contacts",
        "delete": "Remove contacts"
      }
    }
  }
}
```

This will prompt the user to confirm that the application would like to sign in, which identity to use, and the following permissions:

 - Know your identity
 - unwalled.garden dats
   - Read your published dats
   - Publish new dats to your profile
   - Unpublish dats from your profile
 - unwalled.garden contacts
   - Read your contacts
   - Create new contacts
   - Modify your existing contacts
   - Remove contacts

After the user approves, the app will be able to:

 - Read, create, and delete files in `/records/unwalled.garden/dats`
 - Read, create, update, and delete in `/records/unwalled.garden/contacts`

All files written to those folders will validated by their respective schemas. The `/records/unwalled.garden/dats` files will validated against [`dat://unwalled.garden/schemas/dat.json`](https://github.com/beakerbrowser/unwalled.garden/blob/master/schemas/dat.json) on write, and the `/records/unwalled.garden/contacts` files will be validated against [`dat://unwalled.garden/schemas/contact.json`](https://github.com/beakerbrowser/unwalled.garden/blob/master/schemas/contact.json) on write.

### Requirements

|Domain|Requirement|
|-|-|
|**Data&nbsp;semantics**|Data **MUST** be stored as files on a dat.|
||Data **MUST** be identified to the user with high-level semantics.|
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

Record Protocols are identified by a domain name. They are expected to publish a set of definition-files in a dat under that domain. Definition dats must include the [`recordproto`](https://github.com/beakerbrowser/dat-types-spec#recordproto) type.

### Definition files

A Record Protocol dat must follow this file structure:

```
/dat.json          - Standard metadata about the site.
/recordproto.json  - Information about the record protocol.
/schemas/*.json    - Individual JSON-Schema definitions.
*.js               - (optional) API modules for accessing the recordset.
```

### The `recordproto.json` file

This file provides a high-level description of the recordset. The browser uses this file to describe the data to the user, drive permissions, and enforce validation.

The file can contain the following fields, see [this example](/examples/fritter.com/recordproto.json) for more information:

  - `version`: optional Number, the version of the record protocol. Can be specified as Major.Minor, e.g. `1.0`, `1.1`, `2.0`. Defaults to `1.0`.
  - `title`: optional String, the name of the record protocol.
  - `description`: optional String, a short description of the record protocol.
  - `records`: required Object, a description of the recordsets present in the protocol. The keys of this object represent the IDs of recordsets, and the values represent descriptions of the recordsets.
    - `schema`: optional String, a path to a [JSON-Schema draft-07](https://json-schema.org) file which will be used to validate all records written to the recordset's folder. If specified, only `.json` files can be written to the recordset folder. If not specified (or `false`) then the recordset may contain any file.
    - `permissions`: required Object, a description of the permissions which may be requested by applications. The keys of this object must be one of `read`, `create`, `update`, or `delete`. The value must be a String which describes in user-friendly language what the permission means. These key-values map to the following internal meanings:
      - `"read"`: The files in the recordset may be accessed using `readDir()`, `stat()`, and `readFile()`.
      - `"create"`: New files in the recordset may be created using `writeFile()`.
      - `"update"`: Existing files in the recordset may be modified using `writeFile()`.
      - `"delete"`: Existing files in the recordset may be deleted using `unlink()`.

### JSON-Schema files

The schema-definition files found under `/schemas/*.json` are [JSON Schema draft-07](https://json-schema.org) files.

### API modules

Optionally (but recommended) a Record Protocol can include javascript modules for consuming apps to import, providing high level APIs for accessing the recordset. See the [unwalled.garden record protocol](https://github.com/beakerbrowser/unwalled.garden) for an example of this.

## Using Record Protocols

Dats which store records using record protocols must include the [`recordset`](https://github.com/beakerbrowser/dat-types-spec#recordset) type. This type will be set automatically by the browser when records are first written to the dat.

### Importing the protocol

Applications use Record Protocols by specifying it during the [`requestSignin()`](https://github.com/beakerbrowser/dat-identities-spec/tree/updates#apis) flow. This directs the browser to download the definition files and read their instructions for managing the recordset.

Once loaded and activated, the browser creates a dedicated directory for the recordset in a target dat. This directory must follow the following path:

```
/records/{protodomain}
```

For example, a Record Protocol at `dat://fooproto.com` would be assigned the `/records/fooproto.com` folder.

### Web APIs

There are no additional Web APIs for Record Protocols. Applications read and write the record files located under `/records` using the `DatArchive` APIs. This is to ensure that the builtin APIs are minimal and unopinionated. Instead, applications should write their own high-level APIs. Record Protocols are recommended to provide their own API modules.

Internally, Beaker uses the Record Protocols to provide special permissions enforcement and file validation. Applications are only able to read & write files under a `/record` folder after going through the [`requestSignin()`](https://github.com/beakerbrowser/dat-identities-spec/tree/updates#apis) flow and specifying the correct permissions.

Example:

```js
// Request access to the 'unwalled.garden' dats and contacts records
var session = await UserSession.get()
await session.requestSignin({
  records: [{
    url: 'unwalled.garden',
    permissions: {
      dats: ['read', 'create', 'delete'],
      contacts: ['read', 'create', 'update', 'delete']
    }
  }]
})
// After receiving permission from the user,
// the app can:
// - Read, create, and delete files in /records/unwalled.garden/dats
// - Read, create, update, and delete in /records/unwalled.garden/contacts
```

If a record protocol specifies a "schema" file for a recordset, then the app will only be able to write `.json` files to the folder, and writes to those folders will be rejected if the JSON files do not validate against the specified schema.
