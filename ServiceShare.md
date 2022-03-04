# Share a Service API

Using the `add export`, `generate activation`, and `add import` commands of the `nsc` tool, a service API may be made available to other accounts.

> Note: an API provider may choose to make their API available to _any_ account in NGS (_public_ export) or to
> selected accounts only (_private_ export). Private export is shown.

## Payment Service Scenario

For illustration, we consider a payment service (within ACCTA) exposing multiple retail
payment operations including payment tender. Payment tender is an API that takes tender requests on the subject
`retail.v1.payment.tender` and replies with a response message indicating the result of tender. 

The payment service is leveraged by multiple API client applications, including a point-of-sale checkout application that lives in ACCTB.
The steps below show how to grant ACCTB access to use the tender API in its checkout application.

## As the API provider account (ACCTA)

###### Implement and start your service
You may use the NATS command-line tool to provide a mock Payment service that implements a Tender operation. The (optional)
`--queue` parameter allows you to start multiple load-balanced services.
```bash
nats reply --context "<ACCTA USER CONTEXT>" --queue "payment-responder" "retail.v1.payment.tender" "Payment tendered!"
08:48:36 Listening on "retail.v1.payment.tender" in group "payment-responder"
```
> Note: This is an optional step. Cross-account grants may be made at any time and do not depend on a running service. 

###### Make your Payment service eligible to be visible to other accounts
Add a private export of the Payment service that responds to request messages on subject `retail.v1.payment.>` using the 
`nsc add export` command:

```bash
nsc add export --private --name "PAYMENTAPI-GRANT-SERVICE" --account "<ACCTA NAME>" --subject "retail.v1.payment.>" --service
```

> Note: an export only makes a subject _eligible_ to be imported into another account's namespace. Other accounts must
> explicitly import the subject. If the export is private (as here), other accounts must be in possession of an
> import grant activation token generated and provided by the exporter.

###### Explicitly allow ACCTB access to invoke the Payment service

Generate an import grant activation token for ACCTB using the `nsc generate activation` command:
```bash
nsc generate activation --output-file "PAYMENTAPI-GRANT-SERVICE-ACCTB.tok" --account "<ACCTA NAME>" --subject "retail.v1.payment.>" --target-account "<ACCTB PUBLICKEY>"
```
> Note: ACCTA must know ACCTB's NGS public key

Provide the generated token file to the ACCTB owner. 

## As the API using account (ACCTB)

###### Accept access to the ACCTA Payment service
Add an import of the Payment service using the grant activation token provided by the ACCTA owner and the `nsc add import` command:
```bash
nsc add import --token "PAYMENTAPI-GRANT-SERVICE-ACCTB.tok" --account "<ACCTB NAME>" --name "PAYMENTAPI-GRANT-SERVICE" --local-subject "retail.v1.payment.>"
```
> Note: In this example, the ACCTB owner is electing to make the Payment service visible to ACCTB users at `retail.v1.payment.>` but the local subject need not be the same.

###### Make a client request to tender payment
You may use the NATS command-line tool to provide a mock Payment client that requests a Tender operation.
```bash
nats request --context "<ACCTB USER CONTEXT>" "retail.v1.payment.tender" "Tender my payment please!"
08:49:23 Sending request on "retail.v1.payment.tender"
08:49:23 Received on "_INBOX.qH22EvqBXH70dpuV4E6oWJ.iGVnfLLO" rtt 48.544434ms
Payment tendered!
```

Completing the scenario, ACCTA would see the following output at the mock Payment service:
```bash
08:49:23 [#0] Received on subject "retail.v1.payment.tender":
08:49:23 Nats-Request-Info: {"acc":"AD7T74QPICDLJYKEJQ37RKP2VXUJYUUIHZ7XMGMWFEOZNKVO3VBF46F7","rtt":88255126}
```