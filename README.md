# NGS Cross-Account Sharing Guide

## Introduction

NGS enables account owners to share services and message streams with other accounts. Sharing a service or stream is called an account _export_.

> Note: All services and streams of an account are isolated from (and invisible to) other accounts unless mutually shared.

Likewise, NGS enables an account owner to selectively uptake an exported stream or service. Uptaking
a service or stream made available by another account is called an account _import_.

### Relationship to subject namespace

Export and import shares, or _grants_, are scoped by message subject (with or without wildcard).  Fundamentally, a subject import means that
the exported subject is made visible (or addressable) as part of the importer's subject _namespace_.  The importer may map the exported subject -- the _remote_ subject -- to
an alternative _local_ subject in its own namespace to avoid name collisions across two independent account namespaces.

### Service and stream grants

Exports and imports have two types, _service_ and _stream_.  These types map directly to the two interaction styles 
in the underlying NATS protocol: 

* Service: request-reply interaction with two (or more) message exchanges between requester and responder clients where the responder is in the exporting account and requester in the importing account.
* Stream: publish-subcribe interaction with message publish by a client in the exporting account, subscribed by zero (or more) subscribing clients in an importing account.

> A service-type grant (only), _implicitly and automatically_ allows the service responder to send a reply message on the callback subject provided in the caller's service request (informally known as the inbox).

**TODO**: Other service specific options like response type, response threshold, requester account visibility, and sampling.

### Private and public exports

An export may be made available to any other account in NGS to import; this is known as a _public_ export.  

Alternatively, an account owner
may choose to make an export that is _private_.  A private export may be imported by another account _only_ if that account is
in possession of an _activation token_.  An activation token specifies the export that may be imported and by whom.

Activation tokens are minted by the exporting account owner to permit a specific account import. Tokens are provided to the
importing account owner out-of-band of NGS.

> Note: Either type of export may be selectively _revoked_ from a specific importing account through a _revocations_ list. Removing an account export grant
> completely immediately revokes visibility and access for all existing imports.

## Pre-requisites

**TODO**: What are NGSv2 launch links to NGS account setup?  Both Account exporter and Account importer sides.

## Use case examples

* [Request-Reply](#ReqRply)
* [Publish-Subscribe](#PubSub)
* [Durable Publish-Subscribe](#DurablePubSub)

## Share an API<a name="ReqRply"></a>

* Request-Reply

[command example](APIShare.md)

## Share a message stream<a name="PubSub"></a>

* Publish-Subscribe
* Distributed Work Queue

[command example](StreamShare.md)

## Share a durable message stream API<a name="DurablePubSub"></a>

* Publish Subscribe
* Distributed Work Queue

[command example](JetStreamAPIShare.md)


## See also

## TODO
* --response-threshold (service export)
* --response-type (service export)
* verify (de)activation works for public and private exports
* decision: include stream-type shares (core, js)?