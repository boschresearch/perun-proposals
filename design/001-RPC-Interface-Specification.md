<!-- This is a template for proposing design changes to the dst-go project. -->

# Proposal: Descriptive specification of the User API for Two Party Payment Channels

* Author(s): Manoranjith
* Status: propose
* Related issue:
  [perun-node#45](https://github.com/hyperledger-labs/perun-node/issues/45)

<!-- Use the above format for issues on GitHub and full links for issues on
other platforms. -->

## Summary

<!-- Provide a tl;dr summary -->

This specification describes the different APIs that should be provided by the
node for the user to open, use and close two party payment channels.

## Motivation

To have a common specification of the API interface, which can be used to
derive protocol specific formal specifications like Protocol Buffers (for gRPC)
or JSON Schema for JSON RPC.

## Details

The node shall provide three sets of APIs:

1. Node APIs - For accessing the node related functionality. Following APIs
   should be provided:

    1. Get config
    2. Open Session
    3. Time
    4. Help

2. Session APIs - For accessing the session related functionality. Each request
   should include a Session ID. Following APIs should be provided:

    1. Add Contact
    2. Get Contact
    3. Open Channel
    4. Get Channels
    5. Subscribe To Channel Requests
    6. Unsubscribe To Channel Requests
    7. Respond To Channel Request
    7. Subscribe To Channel Closes
    8. Unsubscribe To Channel Closes
    9. Close Session

3. Channel APIS - For accessing the channel related functionality. Each request
   should include a Session ID and Channel ID.

    1. Send Channel Update
    2. Subscribe To Channel Updates
    3. Unsubscribe To Channel Updates
    4. Respond To Channel Update
    5. Get Balance
    6. Close Channel

### Data formats

The following 3 data formats will be used in the APIs.

1. `Peer`

    * `Alias`: [String] Alias for the peer. This will be used to reference a
      peer in all API calls.
    * `Off-Chain Address`: [String] Permanent address (cryptographic) of the
      peer in the off-chain network.
    * `Comm Address`: [String] Address (network) of the peer for off-chain
      communication.
    * `Comm Type`: [String] Type of communication protocol supported by the
      peer for off-chain communication.

2. `Balance Info`

    * `Currency`: [String] Currency used for specifying the amount.
    * `Balance`: [Map of `Alias` to `Balance`] 
      * `Alias`: [String] `Self` is a special alias for the user of the node.
      * `Balance`: [String] Amount held by the `Peer` corresponding to the Alias.

3. `Payment Channel`

    * `Channel ID`: [String] Unique ID of the channel.
    * `Balance`: [Balance Info]
    * `Version`: [int64] Current Version of the channel.

The following error messages will be used in the API description.

1. `Unknown Session ID`: No session corresponding to the specified ID.
2. `Unknown Alias`: No entry in contacts for the given alias.
3. `Unknown Proposal ID`
4. `Unknown Channel ID`: No channel corresponding to the specified ID.
5. `Peer Alias Not Available`: Given alias exists used for another Peer.
6. `Peer Exists`: A contact entry for the given peer already exists.
7. `Peer Not Responding`: Peer did not respond within expected time period.
8. `Invalid Configuration`: Invalid configuration detected.

### Node

#### 1. Get Config

Returns the configuration parameters of the node.

*Parameters* none

*Return*

* `Chain Address`: [String] Address of the default blockchain node used by the
  perun node.
* `Adjudicator Address`: [String] Address of the default Adjudicator contract
  used by the perun node.
* `Asset Address`: [String] Address of the default Asset Holder contract used
  by the perun node.
* `Comm Types`: [List of String] Communication protocols supported by the node
  for off-chain communication.
* `Contact Types`: [List of String] Contacts Provider backends supported by the
  node.

#### 2. Open Session

Open a new session for the given user with the specified configuration file.

*Parameters*

* `Config File`: [String] Path to the config file for the session. This should
   be present on a file system path accessible by the node.

*Return*

* `Session ID`: [String] Unique ID of the session.

*Errors*

* `Invalid Configuration`
* `Session Initialization Error`

#### 3. Time

Returns the time as per perun node's clock in unix format. This time should be used
to check if the timeout in a notification has expired or not.

*Parameters* none

*Return*

* `Time`: [int64] Time in unix format.

#### 3. Help

Returns the help message.

*Parameters* none

*Return*

* `APIs`: [String] List of available APIs.

### Session

#### 1. Add Contact

Add a peer to the contacts in the specified session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Peer`: [Peer] Peer to be added to the contacts.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Peer Exists`
* `Peer Alias Not Available`
* `Invalid Off-Chain Address`

#### 2. Get Contact

Get the peer corresponding to the given alias from the contacts in the
specified session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Alias`: [String] Alias of the peer whose details should be retrieved.

*Return*

* `Peer`: [Peer] Peer retrieved from contacts.

*Errors*

* `Unknown Session ID`
* `Unknown Alias`

#### 3. Open Payment Channel

Open a payment channel with the peer (corresponding to the `Alias`) with the
specified opening balance.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Alias`: [String] Alias of the peer with whom channel is to be opened.
* `Opening Balance`: [Balance Info]
* `Challenge Duration`: [uint64] Challenge Duration for the channel in seconds.

*Return*

* `Channel`: [Payment Channel]

*Errors*

* `Unknown Session ID`
* `Unknown Alias`
* `Invalid Balance`
* `Peer Not Responding`
* `Peer Rejected`

#### 4. Get Payment Channels

Get the list of all payment channels that are open for off-chain transactions
in the specified session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Open channels`: [List of Payment Channel]

*Errors*

* `Unknown Session ID`

#### 5. Subscribe To Channel Requests

Subscribe to notifications on new incoming channel requests in the specified
session.

Only one subscription can be made at a time. Making a repeated subscription
without canceling the previous one will return an error.

The incoming channel requests received when there was no subscription will have
been cached by the node. Once a new subscription is made, node will send these
cached requests (if any), as individual notifications. It will then continue to
send a notification for each new incoming channel request.

Response to the notifications can be sent using the `Respond To Channel
Request` API before the `timeout` expires.

If the notification was received from a `Peer` that is not found in the
contacts provider of the session, the request will be automatically rejected by
the node. User will still receive a notification of this request with
`Proposing Peer` field set to empty string and these notifications should not
be responded to. If the user still responds to it, an `Unknown Proposal ID`
error will be returned. 

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Subscription Already Exists`

*Notification*

Each notification sent to the user should contain the following data:

* `Session ID`: [String] Unique ID of the session.
* `Proposal ID`: [String] Unique ID of this channel proposal.
* `Alias`: [String] Alias of the peer who proposed the channel.
* `Opening Balance`: [Balance Info]
* `Challenge Duration`: [uint64] Challenge Duration for the channel in seconds.
* `Timeout`: [int64] Time (in unix format) before which response should be sent.

#### 6. Unsubscribe To Channel Requests

Unsubscribe to notifications on new incoming channel requests in the specified
session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `No Active Subscription`

#### 7. Respond To Channel Request

Respond to an incoming channel request for which a notification was received.
Response should be sent before the timeout in the notification expires. Use the
`Time` API to fetch current time of the perun node.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Proposal ID`: [String] Unique ID of this proposal as received in the notification.
* `Accept`: [Bool] If True, the proposal will be accepted, else rejected.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Unknown Proposal ID`
* `Peer Not Responding`
* `Response Timeout Expired`

#### 7. Subscribe To Channel Closes

Subscribe to notifications when channels in the specified session are closed by
the peer. User need not respond to these notifications. 

Only one subscription can be made at a time. Making a repeated subscription
request without canceling the previous one will return an error.

The channel close events occurred when there was no subscription will have been
cached by the node. Once a new subscription is made, node will send these
cached events (if any), as individual notifications. It will then continue to
send a notification for each channel closed by a peer. 

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Subscription Already Exists`

*Notification*

Each notification sent to the user should contain the following data:

* `Closing State`: [Payment Channel]

#### 8. Unsubscribe To Channel Closes

Unsubscribe to notifications when channels in the specified session are closed
by the peer. 

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `No Active Subscription`


#### 9. Close Session

Close the specified session. All session data are persisted to disk.

`Force` parameter determines what happens when there are unclosed channels in
the session (in open, dispute or closing in progress state).
  * If it is set to `False` the API returns an error when there are such
    channels. This should be used by default.
  * If `True`, the session is forcibly closed and the API returns list of
   unclosed channels. Use this with caution.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Force`: [bool] Forcibly close the session.

*Return*

* `Unclosed Channels`: [List of Payment Channels] This is relevant only when
  using `Force` = True.

*Errors*

* `Unknown Session ID`
* `Unclosed Payment Channels Exist`

### Payment Channel

#### 1. Send Payment

Send a payment to the specified peer on the channel. Use negative value in
amount to request payment from the peer.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.
* `Alias`: [String] Alias of the peer to which amount should be sent.
* `Amount`: [String] Amount to send. Use negative sign to request payments.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Unknown Alias`
* `Invalid Amount`
* `Peer Not Responding`

#### 2. Subscribe To Channel Updates

Subscribe to notifications on new incoming updates for the specified channel in
the specified session.

Only one subscription can be made at a time. Making a repeated subscription
without canceling the previous one will return an error.

The incoming channel updates received when there was no subscription will have
been cached by the node. Once a new subscription is made, node will send these
cached update (if any), as individual notifications. It will then continue to
send a notification for each new incoming channel update.

Response to the notifications can be sent using the `Respond To Channel
Update` API. 

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Subscription Already Exists`


*Notification*

Each notification sent to the user should contain the following data:

* `Proposing Peer`: [String] Alias of the peer proposing the update. 
* `Proposed Balance`: [Balance Info]
* `Version`: [String] Version of the channel state for proposed update.
* `Final`: [bool] Indicates if this is a final update. Accepted final updates
  can be used to the closed without challenge period.
* `Timeout`: [int64] Time (in unix format) before which response should be
  sent.

#### 3. Unsubscribe To Channel Updates

Unsubscribe to notifications on new incoming updates for the specified channel
in the specified session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `No Active Subscription`

#### 4. Respond To Channel Update

Respond to a channel update. Response should be sent before the timeout in the
notification expires. Use the `Time` API to fetch current time of the perun
node.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]
* `Version`: [String] Version of the channel state received in the notification.
* `Accept`: [Bool] If True, the update will be accepted, else rejected.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Unknown Version`
* `Peer Not Responding`
* `Response Timeout Expired`

#### 5. Get Balance

Get balance for the specified channel.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]

*Return*

* `Channel Balance`: [Balance Info]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`

#### 6. Close Channel

Close the specified payment channel.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]

*Return*

* `Closing Balance`: [Balance Info]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`

## Rationale

<!-- Provide a discussion of alternative approaches and trade offs; advantages
and disadvantages of the specified approach.  -->

1. An alternate approach to defining a dedicated API for payment channel would
   be to define an API for generalized state channels that can also be used for
   payment channel. But having a dedicated API improves the usability of the API
   for payment use cases. While it still allows another end point in the node to
   provide an API for generalized state channels.

## Impact

<!-- Choose the level of impact this proposal will have: -->

<!-- Minor (Does not impact any existing features) --> <!-- Major (Breaks one
or more existing features) --> <!-- New Feature (Introduces a functionality)
--> Architecture (Requires a modification of the architecture)

## Implementation

<!-- Provide a description of the implementation aspects. -->

Define protocol specific format definitions such as Protocol Buffers for gRPC
and implement a protocol agnostic abstract API layer according to this
specification.
