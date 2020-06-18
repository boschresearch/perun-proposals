# List of RPC Methods:

The node shall provide two sets of APIs:
1. Node - APIs that are concerned with the functionality of the node. Eg: GetConfig.
2. Client - APIs that are concerned with a specific client instance of the node. Eg: Open channel, Send update etc.,

## Node :

### 1. GetConfig
Returns the configuration parameters of the node.

#### Parameters
none

#### Return
- `Node config` - struct with following field: (list of parameters to be defined)

### 2. Help
Returns the help message.

#### Parameters
none

#### Return
- `Help Message` - list of available commands.

## Client:

### 1. GetPeers
Get the list of known peers for the user on the specified client.

#### Parameters
- `Client ID`: [String] Unique ID of the client.

#### Return
- `List of peers`: [List of Peer] `Peer` has following fields:
    - `Peer Alias`: [String] Alias for the peer. It is set by the user in the known peers file or when adding the peer to known peers list via rpc interface.
    - `Perun ID`: [String] Perun Address of the peer.
    - `Network Address`: [String] Network address of the peer.
    - `Protocol Versions`: [String] Perun protocol versions supported by the peer. 

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.

### 2. ConnectToPeer
Establish an authenticated connection to the peer using the specified client.

#### Parameters
- `Client ID`: [String] Unique ID of the client.
- `Peer Alias`: [String] Alias for the peer. It is set by the user in the known peers file or when adding the peer to known peers list via rpc interface.

#### Return
- Connected Peers: [List of Peers] `Peer` has following fields:
    - `PeerAlias`: [String] Alias for the peer. It is set by the user in the known peers file or when adding the peer to known peers list via rpc interface.
    - `Perun ID`: [String] Perun Address of the peer.
    - `Network Address`: [String] Network address of the peer.
    - `Protocol Versions`: [String] Perun protocol versions supported by the peer. 

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.
- `Unknown Peer` - No peer in the known peers list, corresponding to specified peer alias.
- `Peer Not Responding` - Peer dID not respond within ResponseTimeout.

### 3. AddPeer
Add a peer to the known peers list on the specified client.

#### Parameters
- `Client ID`: [String] unique ID of the client.
- `Peer Info`: [Peer]. `Peer` has following fields:
    - `PeerAlias`: [String] Alias for the peer. It is set by the user in the known peers file or when adding the peer to known peers list via rpc interface.
    - `Perun ID`: [String] Perun Address of the peer.
    - `Network Address`: [String] Network address of the peer.
    - `Protocol Versions`: [String] Perun protocol versions supported by the peer. 

#### Return
- `Success`: [Boolean]

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.
    
### 4. GetChannels
Get the list all channels in acting phase in the specified client.

#### Parameters
- `Client ID`: [String] unique ID of the client.

#### Return
- Open channels: [List of channel] `Channel` has following fields with string values:
    - `Channel ID`: [String] Unique ID of the channel.
    - `Channel Balance`: [List of Balance Info] `Balance Info` is a struct with following fields:
        - `Alias`: [String] Alias of the peer represented in the channel.
        - `Balance`: [String] Balance of the peer.
        - Note: For balance of the user, alias - "self" will be used. This is a special alias, that cannot be used for any other peer.
    - `Version`: [Integer] Current Version of the channel.

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.

## Channel:

### 1. Open 
Open a channel with the peer with the specified initial balance. User should have an active connecion to the peer before opening the channel.

#### Parameters
- `Client ID`: [String] unique ID of the client.
- Peer Alias
- `Channel Balance`: [List of Balance Info] `Balance Info` is a struct with following fields:
    - `Alias`: [String] Alias of the peer represented in the channel.
    - `Balance`: [String] Balance of the peer.
    - Note: For balance of the user, alias - "self" will be used. This is a special alias, that cannot be used for any other peer.

#### Return
- `Success`: [Boolean]

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.
- `Peer Not Responding` - Peer dID not respond within ResponseTimeout.

### 2. SendPayment
Send a payment to the specified peer on the channel. This is applicable only for payment channels.

#### Parameters
- `Client ID`: [String] unique ID of the client.
- `Channel ID`: [String] Unique ID of the channel.
- `Alias`: [String] Alias of the peer to which amount should be sent.
- `Amount`: [String] Amount to send.

#### Return
- `Channel Balance`: [List of Balance Info] `Balance Info` is a struct with following fields:
    - `Alias`: [String] Alias of the peer represented in the channel.
    - `Balance`: [String] Balance of the peer.
    - Note: For balance of the user, alias - "self" will be used. This is a special alias, that cannot be used for any other peer.

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.
- `Unknown Channel ID` - No channel corresponding to the specified channel ID.
- `Peer Not Responding` - Peer dID not respond within ResponseTimeout.
- `Not a payment channel` - Specified channel is not a payment channel.

### 3. GetBalances
Get balance for the specified channel. This is applicable only for payment channels.

#### Parameters
- `Client ID`: [String] Unique ID of the client.
- `Channel ID`: [String] Unique ID of the channel.

#### Return
- `Channel Balance`: [List of Balance Info] `Balance Info` is a struct with following fields:
    - `Alias`: [String] Alias of the peer represented in the channel.
    - `Balance`: [String] Balance of the peer.
    - Note: For balance of the user, alias - "self" will be used. This is a special alias, that cannot be used for any other peer.

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.
- `Unknown Channel ID` - No channel corresponding to the specified channel ID.
- `Not a payment channel` - Specified channel is not a payment channel.

### 4. Close
Close the specified channel.

#### Parameters
- `Client ID`: [String] Unique ID of the client.
- `Channel ID`: [String] Unique ID of the channel.

#### Return
- `Closed Channel Balance`: [List of Balance Info] `Balance Info` is a struct with following fields:
    - `Alias`: [String] Alias of the peer represented in the channel.
    - `Balance`: [String] Balance of the peer.
    - Note: For balance of the user, alias - "self" will be used. This is a special alias, that cannot be used for any other peer.

#### Errors
- `Unknown Client ID` - No client corresponding to the specified client ID.
- `Unknown Channel ID` - No channel corresponding to the specified channel ID.