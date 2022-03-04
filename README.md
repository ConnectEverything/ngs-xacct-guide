# NGS Cross-Account Sharing Guide

## Introduction

NGS enables account owners to share services and message streams with other accounts. Enabling a service or stream for
sharing is called a namespace _export_.

> Note: All services and streams of an account are isolated from (and invisible to) other accounts unless mutually 
> shared (an export/import pair).

Likewise, NGS enables an account owner to selectively uptake an exported service or stream. Uptaking a service or
stream into your own account is called a namespace _import_.

### Relationship to subject namespace

Export and import shares, or _grants_, are scoped by message subject (with or without wildcard).  Fundamentally, a
subject import means that the exported subject is made visible (or addressable) as part of the importer's 
subject _namespace_.  The importer may map the exported subject -- the _remote_ subject -- to an alternative _local_ 
subject in its own namespace to avoid name collisions across two independent account namespaces. 

NGS maps subjects between accounts at message time and enforces entitlements.

### Service and stream grants

Exports and imports have two types, _service_ and _stream_.  These types map directly to the two interaction styles 
in the underlying NATS protocol. 

##### Service
Request-reply exchange between requesting client and replying client (or _service responder_). The request 
subject is exported by the service responder's account (forming a service address). An importer may **only** publish messages
to this subject (requests). The service responder is dynamically entitled to publish a cross-account response to the
REPLY subject if passed in the request.

> Note: In a service exchange there need not be a reply message, i.e. a 1-way service invocation. Likewise, there may
> be multiple reply messages for a single request, forming a response stream.

##### Stream 
Publish-subscribe exchange between publishing client and subscribing client on the publishing account's
exported subject. An importer may **only** subscribe to the exported subject. 

### Private and public exports

An export may be made available for import by _any_ account in NGS; this is known as a _public_ export.  

Alternatively, an account owner may make a _private_ export.  A private export may be imported by another 
account _only_ if that account is in possession of a digitally-signed _activation token_ from the exporter entitling
them to do so. An activation token specifies the allowed import claims that may be applied in the target account.

Activation tokens are minted by the exporting account owner as necessary to permit each target account. 
Tokens are provided to the importing account owner out-of-band of NGS.

> Note: Both public and private exports may be selectively _revoked_ from a specific importing account through a
> _revocations_ list. Removing an account export grant completely immediately revokes visibility and access for all 
> existing imports.

## Pre-requisites

**TODO**: What are NGSv2 launch links to NGS account setup?  Both Account exporter and Account importer sides.

## Cross-Account Share Scenarios 

###### Basic Shares  
* [Enable an account to invoke your service](ServiceShare.md)
* [Enable an account to subscribe your stream](StreamShare.md)

###### JetStream Consumption Shares
* [Enable an account to consume from your JetStream streaming service (Pull JS Consumer)](JetStreamServiceShare.md) 
* [Enable an account to consume from your JetStream-backed stream (Push JS Consumer)](JetStreamStreamShare.md)

> Note: JetStream "Publish Share" is a "basic" share scenario. Export the JetStream-enabled
> subject as a _service_ to allow the remote account to publish on the subject and (optionally) receive an acknowledgement reply
> from JetStream (At-Least-Once publishing).  