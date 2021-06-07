<!-- This is a template for proposing design changes to the perun project. -->

# Proposal: <!-- Title of the proposal-->

* Author(s): Manoranjith
* Status: accepted
* Related issue: [perun-proposals#008](https://github.com/hyperledger-labs/perun-proposals/pull/008),


<!-- Use the above format for issues on github and full links for issues on other platforms. -->

## Summary

<!-- Provide a tl;dr summary -->
This proposal presents a list to functional changes necessary to implement the
scenario described in in [perun-proposal#003](./003-IoT-Adoption.md) and the
design for approaching these changes.

This focus of this proposal is to implement the changes in go-perun.

## Motivation

<!-- introduction to the problem being solved & its background.  -->

To describe the functional changes implementing the scenario described in
[perun-proposal#003](./003-IoT-Adoption.md) for adopting the perun framework for
IoT use cases. And to present a design for implementing those changes.

## Details

<!-- Provide a detailed description of the proposal. -->

go-perun client which currently runs the complete perun-protocol and manages the
perun state channels througout their life cycle should be split into three
components: transacting component, funding component and watching component.

1. A transacting component which proposes the inital state of channel to all the
   participants, obtains their signatures and generates the initial state.

2. A funding service which takes the initial state of a channel with necessary
   signatures and funds the channel on-chain. On successful funding, it returns
   the funding confirmation.

3. The transacting component (described in 1) takes the initial state of the
   channel with all signatures and funding confirmation; and set up the channel
   for off-chain transactions. It should provide an interface for user to send
   transactions and accept/reject incoming transactions.
   
4. The transacting component should periodically update the latest agreed
   off-chain state to the watching component. The requirements for this time
   period are yet to be worked out.

5. The watching service takes the channel ID, the initial state of the channel
   and is able to watch for events.which is able to start watching for on-chain
   on-chain registeration events. It receives the latest agreed state of the
   channel periodically from the transacting component. If a state is registered
   on-chain and the version of the state is less than the version of the latest
   known by the watcher, watcher should refute with the latest state.

   For sending this transaction, watcher should use its own funds. How watcher
   will be compensated will be be worked out later and is out of scope of this
   change.

6. When the transacting component decides to close the channel, it should be
   able to send a final update to to all channel participants. If this state is
   agreed, channel will be closed "collaboratively" using it. If not, channel
   will be closed "non-collaboratively" using the latest non-final state.

   In either case, the transacting component will send the closing state to the
   funding service. After receiving a response, the trasacting component will
   close the channel.
   
7. The funding service will register the state on-chain (only in case of
   non-collaborative close), settle it and withdraw the funds back to user's
   account.
   
It should be possible to run these components together as a single instance or
as three independent instance. When running them as a single instance,
communication between these components can be implemented as direct funtion
calls or any such mechanism. When running them as three independent instance,
communication should be through a remote interface. Flexibility to plugin
different interfaces should be provided.


Steps for realizing the changes described above:

The core ideas is extracting out the components for each functionality decribed
above, starting from the bottom.

**Watching component:**

1. Separate out the watcher component. So that it is run independently and
   states are periodically updated to it.

2. We can then avoid starting the watcher and use this external component.

**Settling component:**

1. We extract out the settling component. This component does not maintain any
   state. It will take any given state of the channel, register and settle it.

2. We can then avoid making settle calls on the existing client and use this
   external component.

**Transacting component:**

1. We can extract out the transacting state machine, so that it can take the
   initial state of the channel, the funding confirmation and initialize itself.

2. Once initialized, we should be able to do transactions using this component.

3. We can now avoid doing transactions using the main component and use this.
   Which will also use the other external components for settling and watching.

**Funding component:**

1. We can extract the initial channel proposal logic from the main component.
   The output of this component should be the initial state of the channel with all
   necessary signatures.

2. We can then modify the Propose Channel call to take this initial state with
   signatures and fund the channel.

3. We could finally extract the funding component out.

**Impact :**

Changes should be done to the state machine in go-perun to split up. In the end, only the transaction client will maintain a state and the rest of the calls will be stateless.
To run the client in the current constellation, the API will be more or less that same. For interaction between components, we could set it up using some features, say go channels or callbacks.

**Pros:**
We end up with having a set of modular components for each functionality. Of these only the transacting component maintains state. Rest of the components are stateless.

**Cons:**
This approach will require more effort than the other one. Though the exact amount of effort should be quantified.

## Rationale

<!-- Provide a discussion of alternative approaches and trade offs; advantages
and disadvantages of the specified approach.  -->

## Impact

<!-- Choose the level of impact this proposal will have: -->

<!-- Minor (Does not impact any existing features) -->
<!-- Major (Breaks one or more existing features) -->
<!-- New Feature (Introduces a functionality) -->
<!-- Architecture (Requires a modification of the architecture) -->

## Implementation

<!-- Provide a description of the implementation aspects. -->