<img src="static/Synadia_Logo_new_font_only_black.png" alt="Synadia Communications logo" width="200"/>

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
> _revocations_ list. Removing an account export grant immediately revokes visibility and access for all 
> existing imports.

## Guide Pre-requisites

The guide assumes three configured NGS accounts referred to as _ACCTA_, _ACCTB_, and _ACCTC_. To execute
privileged administration steps as an account, NGS account setup must be complete and the user's administrative
environment prepared with NGS security principles and private account credentials.

See [the Synadia site](https://synadia.com/ngs/signup) for more information on how to sign up for Synadia's service and
the [NATS documentation site](https://nats-io.github.io/docs/nats_tools/nsc/) for information about using `nsc` to 
manage your NATS account.

The administrative environment may be prepared using the [NGS command-line setup](https://github.com/ConnectEverything/ngs-cli/).

> Note: NGS JetStream capability not publicly available at time of initial guide publish.

## Cross-Account Share Scenarios 

###### Basic Shares  

The following guides illustrate account owner steps to cross-account share a single subject such as a business API or
non-durable event message channel (i.e. At-Most-Once delivery):

* [Enable an account to invoke your service](ServiceShare.md)
* [Enable an account to subscribe your message stream](StreamShare.md)

###### JetStream Consumption Shares

These guides illustrate account owner steps to cross-account share a durable message stream that leverages 
[JetStream](https://docs.nats.io/nats-concepts/jetstream) technology providing At-Least-Once delivery capability to
consuming applications.

Sharing JetStream Consumers cross-account requires two account exports. JetStream offers two architectural options for 
stream message delivery which is illustrated in the following guides.

> Note: JetStream always streams messages from server to client, but there are two options for message delivery and
> flow-control architecture. Consuming JetStream as a service (Pull JS Consumer) is recommended as the default choice. 

Both guides illustrate a third _optional_ export that provides a remote account application the additional entitlement to
obtain JS Consumer information (e.g. pending unprocessed messages, configuration settings, etc.) via API request.  

* [Enable an account to consume from your JetStream service (Pull JS Consumer)](JetStreamServiceShare.md) 
* [Enable an account to consume from your JetStream-backed stream (Push JS Consumer)](JetStreamStreamShare.md)

> Note: Granting another account permission to publish to your JetStream is a "basic" share scenario. Export a JetStream-ingested
> subject as a _service_ to allow the remote account to publish on the subject and (optionally) receive an acknowledgement reply
> from JetStream (At-Least-Once publishing).

###### Use of JetStream Prefix

When JetStream is shared between accounts, it is assumed that each account is JetStream enabled. A JetStream-enabled
account has JetStream APIs (subjects starting with `$JS`) present in its account namespace.

To disambiguate API calls in a cross-account scenario, JetStream APIs imported into an account from another account 
must be mapped to a local subject with a subject prefix different from `$JS`.  This alternative local subject prefix
is called the _JS Prefix_ and must be set accordingly in the importing account's client application code.

The JetStream cross-account guides show usage of JS prefix.

> Note: JS Prefix is a special case of general capability in subject export/import to map an alternative subject name
> in the importing account to avoid namespace collision.

<hr>
&copy; 2022 Synadia Communications. All rights reserved.