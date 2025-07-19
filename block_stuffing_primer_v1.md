# Bridge Protocols Block-Stuffing Vulnerability Primer

## Version
1.0

## Last Updated
2025-07-13

## Vulnerability Type
Block-Stuffing Denial of Service (DoS)

## Description
Block-stuffing vulnerabilities in bridge protocols occur when an attacker exploits the message processing mechanisms within smart contracts to congest the blockchain network, leading to inefficient use of block space and potential disruptions. This is achieved by submitting multiple duplicate or redundant messages that are processed unnecessarily.

## Protocol Context
Bridge protocols facilitate cross-chain communication and asset transfers between different blockchain networks. They typically consist of a series of interconnected smart contracts designed to handle message routing and validation across chains. The vulnerability arises from the lack of proper deduplication mechanisms in these processes, which can be exploited by attackers to perform block-stuffing attacks.

## Critical Vulnerability Patterns
1. **Lack of Message Deduplication:** Contracts that do not check for duplicate messages before processing them.
2. **Inefficient Storage Operations:** Contracts that redundantly store and process messages without validating their uniqueness.
3. **External Calls Without Validation:** Messages passed between contracts through external calls without checking for duplicates.

### Example Code Pattern
```rust
ExecuteMsg::RouteMessages(msgs) => {
    if info.sender == router.address {
        execute::route_outgoing_messages(deps.storage, msgs)
    } else {
        // Messages initiated via call contract can be routed again
        execute::route_incoming_messages(deps.storage, &router, msgs)
    }
}

pub(crate) fn route_incoming_messages(
    store: &mut dyn Storage,
    router: &Router,
    msgs: Vec<Message>,
) -> Result<Response, Error> {
    for msg in msgs.iter() {
        let stored_msg = state::may_load_incoming_msg(store, &msg.cc_id)
            .change_context(Error::InvalidStoreAccess)?;

        match stored_msg {
            Some(message) if msg != &message => {
                Err(report!(Error::MessageMismatch(msg.cc_id.clone())))
            }
            Some(_) => Ok(()), // Duplicate messages are ignored
            None => Err(report!(Error::MessageNotFound(msg.cc_id.clone()))),
        }?
    }

    let (wasm_msg, events) = route(router, msgs)?;

    Ok(Response::new().add_message(wasm_msg).add_events(events))
}
```

## Common Attack Vectors
1. **Submitting Duplicate Messages:** An attacker can repeatedly submit the same message to exploit inefficient validation and processing mechanisms.
2. **Spamming with Redundant Transactions:** By submitting many redundant transactions, an attacker can congest the network, leading to higher gas costs for other users.

## Detection Heuristics
- Identify functions handling message routing that lack deduplication checks.
- Check if incoming messages are compared against stored duplicates without rejection.
- Look for redundant external calls and storage operations in message processing logic.

## Code Examples
### Vulnerable Code Pattern
```rust
pub(crate) fn route_outgoing_messages(
    store: &mut dyn Storage,
    msgs: Vec<Message>,
) -> Result<Response, Error> {
    for msg in msgs.iter() {
        state::save_outgoing_msg(store, msg.cc_id.clone(), msg.clone())
            .change_context(Error::SaveOutgoingMessage)?;
    }

    Ok(Response::new().add_events(
        msgs.into_iter()
            .map(|msg| AxelarnetGatewayEvent::Routing { msg }.into()),
    ))
}
```

### Mitigated Code Pattern
```rust
pub(crate) fn save_outgoing_msg(
    storage: &mut dyn Storage,
    cc_id: CrossChainId,
    msg: Message,
) -> Result<(), Error> {
    let existing = OUTGOING_MESSAGES
        .may_load(storage, cc_id.clone())
        .map_err(Error::from)?;

    match existing {
        Some(MessageWithStatus { msg: existing_msg, .. }) if msg != existing_msg => Err(Error::MessageMismatch(msg.cc_id.clone())),
        Some(_) => Ok(()), // New message is identical, no need to store it again
        None => OUTGOING_MESSAGES
            .save(
                storage,
                cc_id,
                &MessageWithStatus {
                    msg,
                    status: MessageStatus::Approved,
                },
            )
            .map_err(Error::from)?
            .then(Ok),
    }
}
```

## Exploit Scenarios
- **Block Congestion:** An attacker submits a high volume of duplicate messages, filling up the block space and making it difficult for legitimate transactions to be processed.
- **Gas Price Manipulation:** By congesting the network with redundant transactions, an attacker can artificially inflate gas prices, making it more expensive for other users to transact.

## Mitigation Strategies
1. **Implement Deduplication Checks:** Ensure that message routing functions check for duplicates and reject them early.
2. **Strict Validation Rules:** Introduce strict validation rules to prevent processing of duplicate messages.
3. **Retry Mechanism with Cooldown:** Implement a retry mechanism with increasing cooldown intervals to limit the frequency of repeated messages.

## Detector Prompt
"Analyze the smart contract code for functions that handle message routing and check if there are any missing deduplication checks or redundant storage operations. Ensure that incoming messages are compared against stored duplicates and rejected if they are found to be identical."

By following this primer, developers can better understand and mitigate block-stuffing vulnerabilities in bridge protocols, ensuring more secure and efficient cross-chain communication.