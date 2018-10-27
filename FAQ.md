# FAQ

## How are Record Protocols different than RDF?

Self-describing schemas like JSON-LD and other RDF formats work by encoding every attribute as a URL (eg `bob["foaf.com/name"]`). This solves the problem of ambiguously-defined attributes, but does not provide a solution to overall schema definition (aka "which attributes should be used?") or a solution to security-critical decisions (aka "how should access be described in permission prompts?"). Self-describing schemas are also developer hostile &mdash; nobody wants to reference values by URL-attributes.

Record Protocols instead work by enforcing schema definitions which have been published on the Web. Applications ask the browser to import the schema definitions and then interact with the browser's APIs to read/write data under those definitions. By sharing the globally-published schemas, applications can ensure interoperability, and the browser can ensure accurate metadata and safe permissioning.

## How do Record Protocol solve the issue of trust?

Record Protocol are motivated by complexities around trust and authority. Questions like:

 - What do the user's files contain?
 - Which applications can be trusted to describe them?
 - Who do we trust to describe permissions around accessing that data?
 - etc.

To help clarify this challenge, let's consider a hypothetical:

 > If app A tells you that `/foo.json` contains a cake recipe, and app B tells you it contains car schematic, which is telling the truth?

Obviously you could open `/foo.json` manually and try to suss out the content yourself, but this isn't always so easy for a user to do. Sometimes files are jumbles of structured data. The disputes can also be much more subtle: it might be a dispute about the right way to encode information, or about which attributes are required. It's very hard for the browser/network/ecosystem to answer this dispute. We need a way to ultimately rule in favor of app A or app B.

A natural solution might be to give priority to the creating app. For instance, if app A was the original author of `/foo.json`, then surely it's the authority on the file! The problem with that solution is, there's no correlation between "being first" and "being right" in a shared data space. Consider what might happen around common file-paths such as `/profile.json`. Every other app will want to write to that file! Which one is right? (In a way, they *all* are, because we have no strong definition of "right" in this setup.)

This challenge extends into security as well. Not only do we need a trustworthy description of the data, but we need a trustworthy description of the permissions for accessing the data too! Since we need permissions to provide security, it's obviously important to be able to trust the permission descriptors.

Record Protocols solve all of these challenges by forming an authority model. A Record Protocol is an authority over its own recordset. Consuming apps have to follow the Protocol's decisions. Therefore there's much less potential for disputes; within a protocol's recordset, the protocol has the last word.

## Why use static definitions instead of just callable JS workers?

Q: The definition files will need to describe possible access patterns and schemas, and there may be a limit to what can be expressed this way. Wouldn't it be easier to have the Record Protocol run a ServiceWorker or something similar? Consuming applications could then simply call to the worker via an exported RPC API!

A: This is a good question that I'm still debating myself. Let's call this "Static" vs "Live," where "Static" uses description JSON files (as described in this proposal) and "Live" uses JS provided by the Record Protocol to act as a service.

 - Live is more flexible and requires much less to be specced by the browser.
 - Static is lower overhead. No userland processes need to be run.
 - Static gives the browser more opportunity to get involved (e.g. to provide the perms prompts).
 - Static lets us focus on specific use-cases, whereas Live becomes a challenge of process-management and APIs for the workers to handle things like perms.
 - Static might be easier for people to develop because it's more structured.
 - Static makes it possible for consumers of the proto to provide alternative API definitions.

My current thinking is that we should start with Static and let Live become part of the solution if we feel it's needed. It doesn't hurt us to combine the approaches, and it's always an option to deprecate the Static solutions if we find they're not effective.
