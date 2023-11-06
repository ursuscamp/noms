# NOM-02

## Protocol v1

`mandatory` `author:ursuscamp`

The protocol described in this document is a strict improvement over v0. It resolves data availability issues by putting all name ownership on chain, and it also enables transfers.

## Terminology

  * __Indexer__: A server which maintains an index of name claims made on Bitcoin and correlates them to records events on public Nostr relays.
  * __Claim__: A claim to a new name published in an `OP_RETURN` output on the Bitcoin blockchain.
  * __Public Key__: A 32-byte X-Only Public Key, following the Nostr and Taproot styles.
  * __Signature__: A 64-byte Schnorr signature, following the Nostr and Taproot styles.

## Workflow summary

When a new name is claimed, it is published to the Bitcoin blockchain in an `OP_RETURN` output. When a new v1 claim is found, an indexer should recalculate the v0 `NSID`, and if the it matches a previous v0 claim, then that v0 claim is updated to v1. Otherwise if it is not the first claim, it is invalid.

Additionally, indexers monitor for transfers, which require two transactions (due to size limits on `OP_RETURN` outputs).

Indexer monitor the blockchain for new blocks with Nomen claims, indexing them in order. Indexers also monitor public Nostr relays to find corresponding record events to associate with the name.

## On-chain claims: Creating a name

Claims are indexed in blockchain order. In other words: `Block height -> Transaction height (inside block) -> Output height (inside transaction)`.

Claims are published inside `OP_RETURN` outputs and follow this format: `"NOM" 0x01 0x00 <PUBKEY:32> <NAME:3-43>`. This is a pure binary output, formatted here readability.

  * `"NOM"`: This is a protocol tag. It is a 3-byte bytestring consisting of ASCII characters This helps indexers identity claims.
  * `0x01`: Protocol v1 tag.
  * `0x00`: The transaction byte. __0x00__ indicated a `CREATE`.
  * `PUBKEY`: 32-byte X-Only Public Key.

## On-chain claims: Transferring a name

To initiate a transfer, someone must publish a transaction with an `OP_RETURN` output as such: `"NOM" 0x01 0x01 <PUBKEY:32> <NAME:3-43>`. This is similar to a create except:

  * `PUBKEY` must be the pubkey for the __NEW__ owner, not the original. 
  * The transaction type is `0x01` for a `TRANSFER`.

Upon seeing this output, the indexer will cache it and watch for a signature in a later output or later transaction. If the transfer isn't completed by a signature in `TRANSFER_BLOCK_HEIGHT <= (CURRENT_BLOCK_HEIGHT - 100)` then the indexer will forget about the transfer and it must be reinitiated again. This is simply to prevent the transfer cache from growing unbounded forever.

## On-chain claims: Finalizing a transfer

To complete a transfer, a signature from the original owner authorizing the transfer must be published in an `OP_RETURN` output. The output will take this form: `"NOM" 0x01 0x02 <SIGNATURE:64>`. In this case, the transaction type is `0x02` for `TRANSFER SIGNATURE`.

Upon seeing a transfer signature, an indexer will compare the signature to all cached transfers, and if it exists, it will update the name to match the cached transfer, and remove the transfer from the cache.

To obtain a signature, we use a dummmy Nostr event, which must be signed by the previous owner. After the owner signs the event, the `sig` value is removed from the signed event, and that is the signature that is published on chain.

```json
{
  "pubkey": "previous owner pubkey",
  "created_at": 1,
  "kind": 1,
  "tags": [],
  "content": "<NEW OWNER PUBLIC KEY><NAME>"
}
```

The rationale behind using a dummy Nostr event as the message format is that this is intended to be a Nostr native protocol. Nostr's ecosystem is built around browser extensions that conform to the NIP-07 standard, which can sign only Nostr events, not generic messages. Using this dummy Nostr event allows users to make use of the existing tooling in the ecosystem.

## Nostr

Nostr is the propogation layer of the protocol. The only required information on-chain is the information necessary to determination ownership of a name.

There is one new kind of Nostr event. It is a parameterized replaceable event (all events are idempotent and thus replaceable).

| Event kind | Event type    | Description                                                   |
|------------|---------------|---------------------------------------------------------------|
| 38300      | NAME          | Matches `0x00` tranaction type. Publishes records for a name. |

__NOTE__: This is unchanged from protocol v0.

## New Name

After publishing a `0x00` name transaction, publish a `38300` kind Nostr event. The `d` tag for the event should be the lower case hex representation of the `NAMESPACE ID`. Additionally, there should be a `nom` tag with the `name` value as the parameter. `content` must be a JSON-serialized object of key/value pairs. These key/value pairs represent the records for the name. For example, `NPUB` might be the owner's Nostr npub, `EMAIL` might be the owner's email, etc.

When the records need to be updated, the owner may just publish another name event with different records and it will be replaced.

When receiving new events, an indexer should recalculate the namespace ID and compare to the `d` tag to validate the event. Valid records for a name can only be accepted when published by the name's owner.

## Name format

It is necessary to limit the characters used in names. While it might be tempting to allow any valid UTF-8 string, there are good reasons not to do this. In the Unicode standards, there are sometimes different ways to the construct the same character, invisible characters, or "whitespace" characters that may not necessarily be rendered, etc. This could allow for malicious individuals to trick unsuspecting users into clicking/pasting incorrect names.

While it is desirable to have a wide range of characters and languages be usable, for the time being it is necessary to restrict the use of characters to the basic characters typically used in domain names today.

Names must match the following regular expression `[0-9a-z\-]{3,43}` and must be ignored by indexers otherwise.

## Changes and updates

**2023-11-04**:
  - This protocol document has been re-issued as part of the set of `NOM` specifications.