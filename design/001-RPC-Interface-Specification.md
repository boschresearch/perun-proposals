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

    1. Add Contact
    2. Get Contacts
    3. Open Payment Channel
    4. Get Payment Channels
    5. Subscribe to Payment Channel Requests
    6. Cancel Subscription To Payment Channel Requests
    7. Respond to Payment Channel Request
    7. Subscribe To Channel Close
    8. Cancel Subscription To Channel Close
    9. Close Session

3. Channel APIS - For channel related functionality. Each request will include
a Session ID and Channel ID.

    1. Send Payment
    2. Subscribe to Payment Channel Updates
    3. Cancel Subscription To Payment Channel Updates
    4. Respond to Payment Channel Update
    5. Get Balance
    6. Close Channel

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
* `Challenge Duration`: [uint64] Challenge Duration for channel in seconds.

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

* `Open channels`: [List of Channel]

*Errors*

* `Unknown Session ID`

#### 5. Subscribe To Payment Channel Requests

Subscribe to notifications on new incoming payment channel requests.  A
subscription id will be returned immediately and it will be included in the
notification sent to the user on incoming payment channel requests. 
All pending notifications, not yet timed out will be delivered to the client.

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Subscription Already Exists`

*Notification* Each notification sent to the user should contain the
following data:

* `ProposalID`: [String] Unique ID of this proposal.
* `Proposing Peer`: [Peer] If the peer is missing in the contacts of the user,
  `Peer Alias` field will be empty string.
* `Initial Channel State`: [Channel] Initial state of the proposed channel.
* `Challenge Duration`: [uint64] Challenge Duration for channel in seconds.

#### 6. Cancel Subscription To Payment Channel Requests

Cancel the subscription for payment channel requests corresponding to the
specified subscription ID.

*Parameters*

* `Session ID`: [String] Unique ID of the session.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Subscription ID`
* `No Active Subscription`

#### 7. Respond To Payment Channel Request

Respond to a payment channel request.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `ProposalID`: [String] Unique ID of this proposal.
* `Subscription ID`:[String] Subscription in which the channel request was
  received.
* `Response`:[Bool] True if Accept, False if Reject.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Unknown Subscription ID`
* `Peer Not Responding`

#### 7. Subscribe To Channel Close

Subscribe to notifications on channels that are closed by the peer. User need
not respond to these notifications. If the channel was already closed by peer
before making this subscription, it will be delivered to the client as
notification.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Subscription Already Exists`

*Notification* Each notification sent to the user should contain the
following data:

* `Final Channel Balance`: [List of Balance Info]

#### 8. Cancel Subscription To Channel Close

Cancel the subscription for channel close events corresponding to the
specified subscription ID.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `No Active Subscription`


#### 9. Close Session

Close the specified session. All session data are persisted to disk.
If Persiste Open Channels is true, returns success = false if there are open channels.
If it is false, persists all the open channel data to disk and closes the session.

*Parameters*

* `Session ID`: [String]
* `Persist Open Channels`: [bool]

*Return*

* `Success`: [bool]
* `Open Channels`: [List of Channels]

*Errors*

* `Unknown Session ID`

### Channel

#### 1. Send Payment

Send a payment to the specified peer on the channel.
Using negative value in amount to request payment from the user.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.
* `Alias`: [String] Alias of the peer to which amount should be sent.
* `Amount`: [String] Amount can be specified in currency string format (as
  specified in data formats section).

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Peer Not Responding`

#### 2. Subscribe To Payment Channel Updates

Subscribe to notifications on new incoming payment channel updates.  A
subscription id will be returned immediately and it will be included in the
notification sent to the user on incoming payment channel updates.
All pending notifications, not yet timed out will be delivered to the client.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Success`: [bool] This id will be included in the notifications.

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Subscription Already Exists`


*Notification* Each notification sent to the user should contain the
following data:

* `Proposing Peer`: [Peer] If the peer is missing in the contacts of the user,
  `Peer Alias` field will be empty string.
* `Proposed Balance`: [Balance Info]
* `Version`: [String] New version for the proposed channel.
* `IsClosingUpdate`: [bool]

#### 3. Cancel Subscription To Payment Channel Updates

Cancel the subscription for payment channel updates corresponding to the
specified subscription ID.

*Parameters*

* `Session ID`: [String] Unique ID of the session.
* `Channel ID`: [String] Unique ID of the channel.

*Return*

* `Success`: [bool]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `No Active Subscription`

#### 4. Respond To Payment Channel Update

Respond to a payment channel request.

*Parameters*

* `Session ID`: [String]
* `Channel ID`: [String]
* `Subscription ID`:[String] Subscription in which the channel request was
  received.
* `Version`: [String] Version for the proposed channel.
* `Response`:[Bool] True if Accept, False if Reject.

*Return*

* `Success`: [Boolean]

*Errors*

* `Unknown Session ID`
* `Unknown Channel ID`
* `Unknown Subscription ID`
* `Unknown Version`
* `Peer Not Responding`

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

#### 6. Close Channel

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
and implement a protocol agnostic abstract User API layer according to this spec.
