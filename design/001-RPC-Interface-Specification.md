<!-- This is a template for proposing design changes to the dst-go project. -->

# Proposal: Descriptive specification of the User API for Payment Channels

* Author(s): Manoranjith
* Status: propose
* Related issue:
  [dst-go#45](https://github.com/direct-state-transfer/dst-go/issues/45)

<!-- Use the above format for issues on GitHub and full links for issues on
other platforms. -->

## Summary

<!-- Provide a tl;dr summary -->

This specification describes the different APIs that should be provided by the
node for enabling the user to open and use payment channels.

## Motivation

To have a standardized definition of the API interface, which can be used to
derive protocol specific format specifications like Protocol Buffers (for gRPC)
or JSON Schema for JSON RPC.

## Details

The node shall provide three sets of APIs:

1. Node APIs - For accessing node related functionality of the node. Following
APIs should be provided:

    1. Get config
    2. Open Session
    3. Help

2. Session APIs - For session related functionality. Each request will include
a Session ID. Following APIs should be provided:
    1. Add contact
    2. Get contacts
    3. Open payment channel
    4. Get payment channels
    5. Subscribe to payment channel requests
    6. Respond to payment channel request

3. Channel APIS - For channel related functionality. Each request will include
a Session ID and Channel ID.
    1. Send payment
    2. Request payment
    3. Subscribe to payment channel updates
    4. Respond to payment channel update
    5. Get balance
    6. Close channel

### Data formats

The following 4 data formats will be used in the API description.

1. `Peer`

    * `PeerAlias`: [String] Alias for the peer. It is a value set by the user to
  reference the peer during API calls.
    * `Perun ID`: [String] Permanent Identity of the peer in the off-chain
  network.
    * `Network Address`: [String] Network address for off-communications.
    * `Protocol Versions`: [String] Perun protocol versions supported by the
    peer.

2. `Balance Info`

    * `Currency`: [String] Currency used in the balance.
    * List of `Alias`, `Balance` pairs, where
      * `Alias`: [String] Alias of the peer represented in the channel. `Self` is
        a special alias used for the user of the node.
      * `Balance`: [String] Balance of the peer.

3. `Currency String`: The balance of can be specified in currency string format
  as show below. The letters used that can used in the currency string is
determined by the value of currency field.

    * 2e50c for currency "Euro" that represents 2 euros and 50 cents and,
    * 2e50w for currency "Ethereum" that represents 2 ethers and 50 wei.

4. `Channel`

    * `Channel ID`: [String] Unique ID of the channel.
    * `Channel Balance`: [Balance Info]
    * `Version`: [Integer] Current Version of the channel.

The following error messages will be used in the API description.

1. `Unknown Session ID`: No session corresponding to the specified ID.
2. `Unknown Channel ID`: No channel corresponding to the specified ID.
3. `Unknown Subscription ID`: No subscription corresponding to the specified ID.
4. `Peer Exists`: A contact entry exits for the given peer and cannot be
5. `Peer Not Responding`: Peer did not respond within expected time period.

### Node

#### 1. Get Config

Returns the configuration parameters of the node.

*Parameters* none

*Return*

* `Node config`: (list of configurable parameters is yet to be defined)

#### 2. Open Session

Open a new session for the given user with the specified configuration file.

*Parameters*

* `Config File`: [String] Path to the config file for the session. This should
   present on a file system path accesible by the node.

*Return*

* `Session ID`: [String] Unique ID of the session.
* `Success`: [Boolean]

*Errors*

* `Session already exists`

#### 3. Help

Returns the help message.

*Parameters* none

*Return*

* `Help Message`: list of available commands.

### Session

#### 1. Add Contact

Add a peer to the contacts in the specified session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Contact`: [Peer] Details of peer to be added to contacts.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Peer Exists`

#### 2. Get Contacts

Get the contacts list of the user in the specified session.

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Contacts`: [List of Peer]

*Errors*

* `Unknown Session ID`

#### 3. Open Payment Channel

Open a payment channel with the peer with the specified initial balance.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Peer Alias`: [String] Alias of the peer with whom channel is to be opened.
* `Initial Channel Balance`: [Balance Info]

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Peer Not Responding`

#### 4. Get Payment Channels

Get the list of all payment channels in the specified session, that are open
for off-chain transactions.

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Open channels`: [List of channel]

*Errors*

* `Unknown Session ID`

#### 5. Subscribe To Payment Channel Requests

Subscribe to notifications on new incoming payment channel requests.  A
subscription id will be returned immediately and it will be included in the
notification sent to the user on incoming payment channel requests. 

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Subscription ID`: [String] This id will be included in the notifications.

*Errors*

* `Unknown Session ID`

*Notification* Each notification sent to the user should contain the
following data:

* `Proposing Peer`: [Peer] If the peer is missing in the contacts of the user,
  `Peer Alias` field will be empty string.
* `Initial Channel State`: [Channel] Initial state of the proposed channel.

#### 6. Respond To Payment Channel Request

Respond to a payment channel request.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.
* `Subscription ID`:[String] Subscription in which the channel request was
  received.
* `Response`:[String] This value should either be `Accept` or `Decline`.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Unknown Subscription ID`

### Channel

#### 1. Send Payment

Send a payment to the specified peer on the channel.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.
* `Alias`: [String] Alias of the peer to which amount should be sent.
* `Amount`: [String] Amount can be specified in currency string format (as
  specified in data formats section).

*Return*

* `Updated Channel Balance`: [Balance Info]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Peer Not Responding`

#### 2. Request Payment

Request a payment from the specified peer on the channel.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]
* `Peer Alias`: [String] Alias of the peer from which amount should be
  requested.
* `Amount`: [String] Amount can be specified in currency string format (as
  specified in data formats section).

*Return*

* `Updated Channel Balance`: [Balance Info]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Peer Not Responding`

#### 3. Subscribe To Payment Channel Update

Subscribe to notifications on new incoming payment channel updates.  A
subscription id will be returned immediately and it will be included in the
notification sent to the user on incoming payment channel updates.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Subscription ID`: [String] This id will be included in the notifications.

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`

*Notification* Each notification sent to the user should contain the
following data:

* `Proposing Peer`: [Peer] If the peer is missing in the contacts of the user,
  `Peer Alias` field will be empty string.
* `Proposed Balance`: [Balance Info]

#### 4. Respond To Payment Channel Update

Respond to a payment channel request.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]
* `Subscription ID`:[String] Subscription in which the channel request was
  received.
* `Response`:[String] This value should either be `Accept` or `Decline`.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`

#### 5. Get Balances

Get balance for the specified payment channel.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]

*Return*

* `Channel Balance`: [Balance Info]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`

#### 6. Close

Close the specified payment channel.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]

*Return*

* `Final Channel Balance`: [List of Balance Info]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`

## Rationale

<!-- Provide a discussion of alternative approaches and trade offs; advantages
and disadvantages of the specified approach.  -->

1. An alternate approach to defining a dedicated API for payment channel would
be to define an API for generalized state channels that can also be used for
payment channel.  But having a dedicated API makes improves the usability of
the API for payment use cases. While it still allows another end point in the
node to provide an API for generalized state channels.

## Impact

<!-- Choose the level of impact this proposal will have: -->

<!-- Minor (Does not impact any existing features) --> <!-- Major (Breaks one
or more existing features) --> <!-- New Feature (Introduces a functionality)
--> Architecture (Requires a modification of the architecture)

## Implementation

<!-- Provide a description of the implementation aspects. -->

Define protocol specific format definitions such as Protocol Buffers for gRPC
and create an issue for implementing an protocol agnostic abstract User API
layer according to this spec.
