# Share JetStream as a Stream

Using the `add export`, `generate activation` and `add import` commands of the `nsc` tool, a durable JetStream
Consumer may be made available to other accounts as a _stream_. 

> Note: a stream provider may choose to make their stream available to _any_ account in NGS (_public_ export) or to
> selected accounts only (_private_ export). Private export is shown in this guide.

A stream-style JetStream Consumer, also known as a Push JS Consumer, does message delivery to a _static subject_ specified
when the JetStream Consumer is created. All message deliveries are made to the single subject (whether there is one subscribing
application or many).

> Note: The rate and frequency of message delivery to consuming applications with a Push JS Consumer is inherently
> controlled by the rate of underlying JetStream ingestion, the JS Consumer's 
> [ReplayPolicy](https://docs.nats.io/nats-concepts/jetstream/consumers#replaypolicy), 
> [MaxAckPending](https://docs.nats.io/nats-concepts/jetstream/consumers#maxackpending), 
> [RateLimit](https://docs.nats.io/nats-concepts/jetstream/consumers#ratelimit), and
> [FlowControl](https://docs.nats.io/nats-concepts/jetstream/consumers#flowcontrol) policies for the JS Consumer (i.e. not specific to
> one consuming application). Consider sharing [JetStream as a service](JetStreamAPIShare.md) for application-centric delivery management.

## Order Events Scenario

For illustration, we consider a stream of business event notifications from a system (within ACCTA) handling 
submitted retail orders. These event messages trigger actions in multiple downstream systems. Order events are
ingested into a JetStream named `ORDEREVENTS`.

A fulfillment application in ACCTB needs to reliably see and process order event messages.

ACCTA has set up a JetStream Consumer named `ORDEREVENTS-C1` for ACCTB's application to receive order events. The steps
below show how to grant ACCTB access to subscribe to the `ORDEREVENTS-C1` delivery subject in its event processing application.

## As the JetStream Consumer stream provider account (ACCTA)

###### Publish Order Events
You may use the NATS command-line tool to provide a mock Order service that publishes Order Captured event notifications.
```bash
nats request --context "<ACCTA USER CONTEXT>" "retail.v1.order.captured" "Captured order 1234!"
14:36:10 Sending request on "retail.v1.order.captured"
14:36:10 Received on "_INBOX.8T7G6V9J6DksHdJYOYgsXJ.zPnc8UTP" rtt 18.417124ms
{"stream":"ORDEREVENTS", "domain":"ngstest", "seq":3}
```

> Note: ACCTB will not receive any messages until the following steps are complete.

###### Make ACCTA Order Events stream eligible for consumption by other accounts

Add private exports of the following Orders Events stream (and related services) in ACCTA with the `nsc add export` command:

| Export                                    | Grant   | Subject                                                | Export Required |
|-------------------------------------------|---------|--------------------------------------------------------|-----------------|
| Subscribe static message delivery subject | stream  | `deliver.retail.v1.order.events`                       | Yes             |
| Acknowledge a delivered message           | service | `$JS.ACK.RESTOCKEVENTS.RESTOCKEVENTS-C1.>`             | Yes             |
| Request JetStream consumer status         | service | `$JS.API.CONSUMER.INFO.RESTOCKEVENTS.RESTOCKEVENTS-C1` | Optional        |

```bash
nsc add export --private --account "<ACCTA NAME>" --name "ORDEREVENTS-GRANT-DELIVER" --subject "deliver.retail.v1.order.events"
nsc add export --private --account "<ACCTA NAME>" --name "ORDEREVENTS-GRANT-ACK" --subject "\$JS.ACK.ORDEREVENTS.ORDEREVENTS-C1.>" --service
nsc add export --private --account "<ACCTA NAME>" --name "ORDEREVENTS-GRANT-INFO" --subject "\$JS.API.CONSUMER.INFO.ORDEREVENTS.ORDEREVENTS-C1" --service
```
> Note: an export only makes a subject _eligible_ to be imported into another account's namespace. Other accounts must
> explicitly import the subject. If the export is private (as here), other accounts must be in possession of an
> import grant activation token generated and provided by the exporter.
 
###### Specifically allow ACCTB access to Order Events 

Generate grant activation tokens for ACCTB with the `nsc generate activation` command:
```bash
nsc generate activation --output-file "ORDEREVENTS-GRANT-DELIVER-ACCTB.tok" --account "<ACCTA NAME>" --subject "deliver.retail.v1.order.events" --target-account "<ACCTB PUBLICKEY>"
nsc generate activation --output-file "ORDEREVENTS-GRANT-ACK-ACCTB.tok" --account "<ACCTA NAME>" --subject "\$JS.ACK.ORDEREVENTS.ORDEREVENTS-C1.>" --target-account "<ACCTB PUBLICKEY>"
nsc generate activation --output-file "ORDEREVENTS-GRANT-INFO-ACCTB.tok" --account "<ACCTA NAME>" --subject "\$JS.API.CONSUMER.INFO.ORDEREVENTS.ORDEREVENTS-C1" --target-account "<ACCTB PUBLICKEY>"
```
> Note: ACCTA must know ACCTB's NGS public key.

Provide the generated token files to the ACCTB owner.

## As the consuming application account (ACCTB)

###### Accept access to Order Events

Import grants using the ACCTA-provided activation token with the `nsc add import` command:

| Import                                    | Grant   | Import Required | JS Prefix Required |
|-------------------------------------------|---------|-----------------|--------------------|
| Subscribe static message delivery subject | stream  | Yes             | No                 |
| Acknowledge a delivered message           | service | Yes             | No                 |
| Request JetStream consumer status         | service | Optional        | Yes                |

```bash
nsc add import --token "ORDEREVENTS-GRANT-DELIVER-ACCTB.tok" --account "<ACCTB NAME>" --name "ORDEREVENTS-GRANT-DELIVER" --local-subject "retail.v1.order.events"
nsc add import --token "ORDEREVENTS-GRANT-ACK-ACCTB.tok" --account "<ACCTB NAME>" --name "ORDEREVENTS-GRANT-ACK" --local-subject "\$JS.ACK.ORDEREVENTS.ORDEREVENTS-C1.>" --service
nsc add import --token "ORDEREVENTS-GRANT-INFO-ACCTB.tok" --account "<ACCTB NAME>" --name "ORDEREVENTS-GRANT-INFO" --local-subject "ACCTA.API.CONSUMER.INFO.ORDEREVENTS.ORDEREVENTS-C1" --service
```

###### Subscribe to Order Events

You may use the NATS command-line tool to create a mock service that processes Order Events.

```bash
# note: --js-api-prefix not technically required for Push JS Consumer delivery, but is required for ACCTB to request JetStream consumer info 
nats sub --context "<ACCTB USER CONTEXT>" --js-api-prefix "ACCTA.API" --ack --queue "order-processor" "retail.v1.order.events"
14:36:06 Subscribing on retail.v1.order.events with acknowledgement of JetStream messages
[#1] Received JetStream message: consumer: ORDEREVENTS > ORDEREVENTS-C1 / subject: retail.v1.order.events / delivered: 1 / consumer seq: 3 / stream seq: 3 / ack: true
Captured order 1234!
```

> Note: `--queue` is required since the ORDEREVENTS-C1 consumer specifies Deliver Group as "order-processor" to enable multi-consumer load balancing.