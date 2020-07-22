<!-- This is a template for proposing design changes to the dst-go project. -->

# Proposal: Descriptive specification for user API for Payment Channels

* Author(s): Manoranjith
* Status: propose
* Related issue:
  [dst-go#45](https://github.com/direct-state-transfer/dst-go/issues/45)

<!-- Use the above format for issues on GitHub and full links for issues on
other platforms. -->

## Summary

<!-- Provide a tl;dr summary --> This specification describes the different
APIs that should be provided by the node for enabling the user to open and use
payment channels.

## Motivation

To have a standardized definition of the API interface, which can be used to
derive protocol specific format specifications like Protocol Buffers (for
gRPC) or JSON Schema for JSON RPC.

## Details

The node shall provide three sets of APIs:

1. Node APIs - For accessing node related functionality of the node. Following
APIs should be provided:

    1. Get Config
    2. Help

2. Session APIs - For session related functionality. Each request will include
a Session ID. Following APIs should be provided:
    1. Add Contact
    2. Get Contacts
    3. Open Payment Channel
    4. Get Payment Channels

3. Channel APIS - For channel related functionality. Each request will include
a Session ID and Channel ID.
    1. Send payment
    2. Request payment
    3. Get Balance
    4. Close Channel

The structure of peer and channel data are defined below and will be used in
the API descriptions.

* `Peer` has the following fields:
  * `PeerAlias`: [String] Alias for the peer. It is a value set by the user to
    reference the peer during API calls.
  * `Perun ID`: [String] Permanent Identity of the peer in the off-chain
    network.
  * `Network Address`: [String] Network address for off-communications.
  * `Protocol Versions`: [String] Perun protocol versions supported by the
    peer.

* `Balance Info` has the following fields:
  * `Currency`: [String] Currency used in the balance.
    * `Note`: Currency will determine the letters used to represent the
      `Balance`. E.g., 
      * 2e50c for currency "Euro" that represents 2 euros and 50 cents and,
      * 2e50w for currency "Ethereum" that represents 2 ethers and 50 wei.
  * List of `Alias` and `Balance` pairs, where
    * `Alias`: [String] Alias of the peer represented in the channel.
    * `Balance`: [String] Balance of the peer.
      * `Note`: For balance of the user, a special alias "self" will be used.
        It cannot be used for any other peer.

* `Channel` has following fields:
  * `Channel ID`: [String] Unique ID of the channel.
  * `Channel Balance`: [List of Balance Info]
  * `Version`: [Integer] Current Version of the channel.

### Node

#### 1. Get Config

Returns the configuration parameters of the node.

**Parameters** none

**Return**

* `Node config`: (list of configurable parameters is yet to be defined)

#### 2. Help

Returns the help message.

**Parameters** none

**Return**

* `Help Message`: list of available commands.

### Session

#### 1. Add Contact

Add a peer to the contacts in the specified session.

**Parameters**

* `Session ID`: [String] Unique ID for the session.
* `Contact`: [Peer] Details of peer to be added to contacts.

**Return**

* `Success`: [Boolean]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.
* `Peer Exists`: A contact entry exits for the given peer and cannot be
  updated.

#### 2. Get Contacts

Get the contacts list of the user in the specified session.

**Parameters**

* `Session ID`: [String] Unique ID for the session.

**Return**

* `Contacts`: [List of Peer]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.

#### 3. Open Payment Channel

Open a payment channel with the peer with the specified initial balance.

**Parameters**

* `Session ID`: [String] Unique ID for the session.
* `Peer Alias`: [String] Alias of the peer with whom channel is to be opened.
* `Initial Channel Balance`: [Balance Info]

##### Return

* `Success`: [Boolean]

##### Errors

* `Unknown Session ID`: No session corresponding to the specified ID.
* `Peer Not Responding`: Peer did not respond within expected time period.

#### 4. Get Payment Channels

Get the list of all payment channels in the specified session, that are open
for off-chain transactions.

**Parameters**

* `Session ID`: [String] Unique ID for the session.

##### Return

* Open channels: [List of channel]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.

### Channel

#### 1. Send Payment

Send a payment to the specified peer on the channel.

**Parameters**

* `Session ID`: [String] Unique ID for the session.
* `Channel ID`: [String] Unique ID of the channel.
* `Alias`: [String] Alias of the peer to which amount should be sent.
* `Amount`: [String] Amount to be sent. See the definition of `balance info`
  for how this field should be formatted.

**Return**

* `Updated Channel Balance`: [Balance Info]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.
* `Unknown Channel ID`: No channel corresponding to the specified channel ID.
* `Peer Not Responding`: Peer did not respond within expected time period.

#### 1. Request Payment

Request a payment from the specified peer on the channel.

**Parameters**

* `Session ID`: [String] Unique ID for the session.
* `Channel ID`: [String] Unique ID of the channel.
* `Alias`: [String] Alias of the peer from which amount should be requested.
* `Amount`: [String] Amount to be requested. See the definition of `balance
  info` for how this field should be formatted.

**Return**

* `Updated Channel Balance`: [Balance Info]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.
* `Unknown Channel ID`: No channel corresponding to the specified channel ID.
* `Peer Not Responding`: Peer did not respond within expected time period.

#### 3. Get Balances

Get balance for the specified payment channel.

**Parameters**

* `Session ID`: [String] Unique ID for the session.
* `Channel ID`: [String] Unique ID of the channel.

**Return**

* `Channel Balance`: [Balance Info]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.
* `Unknown Channel ID`: No channel corresponding to the specified channel ID.

#### 4. Close

Close the specified payment channel.

**Parameters**

* `Session ID`: [String] Unique ID for the session.
* `Channel ID`: [String] Unique ID of the channel.

**Return**

* `Final Channel Balance`: [List of Balance Info]

**Errors**

* `Unknown Session ID`: No session corresponding to the specified ID.
* `Unknown Channel ID`: No channel corresponding to the specified channel ID.

## Rationale

<!-- Provide a discussion of alternative approaches and trade offs; advantages
and disadvantages of the specified approach.  -->

1. An alternate approach to defining a dedicated API for payment channel would
be to defined an API for channels that can also be used for payment channel.
But having a dedicated API makes improves the usability of the API for payment
use cases. While this approach still allows another end point in the node to
provide an API for generalized state channels.

## Impact

<!-- Choose the level of impact this proposal will have: -->

<!-- Minor (Does not impact any existing features) --> <!-- Major (Breaks one
or more existing features) --> <!-- New Feature (Introduces a functionality)
--> Architecture (Requires a modification of the architecture)

## Implementation

<!-- Provide a description of the implementation aspects. --> Define protocol
specific format definitions such as Protocol Buffers for gRPC and create an
issue for implementing an protocol agnostic abstract User API layer according
to this spec.
