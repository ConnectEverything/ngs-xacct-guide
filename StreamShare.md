# Cross-Account Message Stream Share

Using the `add export`, `generate activation`, and `add import` commands of the `nsc` tool, a message stream may be made available to other accounts.

> Note: an API provider may choose to make their message stream available to _any_ account in NGS (_public_ export) or to
> selected accounts only (_private_ export). Private export is shown.

## Shopping Cart Events Scenario

For illustration, we consider a click stream of UX navigation event notifications from a system (within ACCTA) handling real time
shopping cart customer experience for a large online retailer. These events trigger immediate (session relevant) actions
in downstream systems and are also consumed and analyzed to improve and optimize future customer interaction. 

Cart events of multiple types are published to the subject `retail.v1.cart.<event type>` by the front-end application.

An analytics system that resides in ACCTB needs to process cart events into its learning models. The steps
below show how to grant ACCTB access to subscribe and receive cart events.

> See also [ngs-xacct-demo/service](https://github.com/ConnectEverything/ngs-xacct-demo/service/) for
> scenario scripts.

## As the stream publishing account (ACCTA)

###### Publish Cart Events
You may use the NATS command-line tool to provide a mock Cart service that publishes Cart Started event notifications.
```bash
nats pub --context "<ACCTA USER CONTEXT>" "retail.v1.cart.started" "Customer initiated Cart 1234!"
14:13:32 Published 29 bytes to "retail.v1.cart.started"
```

> Note: ACCTB will not receive any messages published until the following steps are complete. Only publishes made when
> ACCTB has access and is actively subscribed will be delivered. Consider using [JetStream](https://docs.nats.io/nats-concepts/jetstream) if you require
> durable message streams.

###### Make your message stream eligible to be visible to other accounts
Add a private export of the Cart Events message stream on subject `retail.v1.cart.>` using the `nsc add export` command:

```bash
nsc add export --private --name "CARTEVENTS-GRANT-STREAM" --account "<ACCTA NAME>" --subject "retail.v1.cart.>"
```

> Note: an export only makes a subject _eligible_ to be imported into another account's namespace. Other accounts must
> explicitly import the subject. If the export is private (as here), other accounts must be in possession of an
> import grant activation token generated and provided by the exporter.

###### Explicitly allow ACCTB to receive published Cart Event messages 

Generate an import grant activation token for ACCTB using the `nsc generate activation` command:
```bash
nsc generate activation --output-file "CARTEVENTS-GRANT-STREAM-ACCTB.tok" --account "<ACCTA NAME>" --subject "retail.v1.cart.>" --target-account "<ACCTB PUBLICKEY>"
```
> Note: ACCTA must known ACCTB's public key.

Provide the generated token file to the ACCTB owner.

## As the stream subscriber account (ACCTB)

###### Accept access to the ACCTA stream
Add an import of the Cart Events stream using the grant activation token provided by the ACCTA owner and the `nsc add import` command:

```bash
nsc add import --token "CARTEVENTS-GRANT-STREAM-ACCTB.tok" --account "<ACCTB NAME>" --name "CARTEVENTS-GRANT-STREAM" --local-subject "retail.v1.cart.>"
```

> Note: In this example, the ACCTB owner is electing to make the Cart Events stream visible to ACCTB users at `retail.v1.cart.>` but the local subject need not be the same.

###### Subscribe to Cart Events

You may use the NATS command-line tool to create a mock service that processes Cart Events.  NATS will load balance multiple subscribers of the same queue name (optional).
```bash
nats sub --context "<ACCTB USER CONTEXT>" --queue "cart-processor" "retail.v1.cart.>"
14:13:21 Subscribing on retail.v1.cart.>
[#1] Received on "retail.v1.cart.started"
Customer initiated Cart 1234!
```