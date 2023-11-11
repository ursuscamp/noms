# NOM-01

## Protocol v0

`mandatory` `deprecated` `author:ursuscamp`

The protocol described in this document is deprecated (no new v0 names should be issues) but mandatory for indexers to implement.

## Terminology

  * __Indexer__: A server which maintains an index of name claims made on Bitcoin and correlates them to records events on public Nostr relays.
  * __Claim__: A claim to a new name published in an `OP_RETURN` output on the Bitcoin blockchain.
  * __Public Key__: A 32-byte X-Only Public Key, following the Nostr and Taproot styles.
  * __Signature__: A 64-byte Schnorr signature, following the Nostr and Taproot styles.
  * __HASH160__: A cryptographic hash function consisting of `RIPEMD-160(SHA-256(<MESSAGE>))`.

## Workflow summary

When a new name is claimed, it is published to the Bitcoin blockchain in an `OP_RETURN` output. If it is not the first claim to such a name, it is invalid. Indexer monitor the blockchain for new blocks with Nomen claims, indexing them in order. Indexers also monitor public Nostr relays to find corresponding record events to associate with the name.

## On-chain claims

Claims are indexed in blockchain order. In other words: `Block height -> Transaction height (inside block) -> Output height (inside transaction)`.

Claims are published inside `OP_RETURN` outputs and follow this format: `"NOM" 0x00 0x00 <NSID:20>`. This is a pure binary output, formatted here readability.

  * `"NOM"`: This is a protocol tag. It is a 3-byte bytestring consisting of ASCII characters This helps indexers identity claims.
  * `0x00`: The first __0x00__ byte is a protocol version (hence the name "Protocol v0").
  * `0x00`: The second __0x00__ byte is a transaction type. In this case, it means a `CREATE` transaction, or a claim to a new name. Protocol v0 only has one transaction type.
  * `NSID` is a 20-byte HASH160 of the name bytestring and 32-byte binary public key concatenated together.

## Nostr

Nostr is the propogation layer of the protocol. The only required information on-chain is the information necessary to determination ownership of a name.

There is one new kind of Nostr event. It is a parameterized replaceable event (all events are idempotent and thus replaceable).

| Event kind | Event type    | Description                                                   |
|------------|---------------|---------------------------------------------------------------|
| 38300      | NAME          | Matches `0x00` tranaction type. Publishes records for a name. |

## New Name

After publishing a `0x00` name transaction, publish a `38300` kind Nostr event. The `d` tag for the event should be the lower case hex representation of the `NAMESPACE ID`. Additionally, there should be a `nom` tag with the `name` value as the parameter. `content` must be a JSON-serialized object of key/value pairs. These key/value pairs represent the records for the name. For example, `NPUB` might be the owner's Nostr npub, `EMAIL` might be the owner's email, etc.

When the records need to be updated, the owner may just publish another name event with different records and it will be replaced.

When receiving new events, an indexer should recalculate the namespace ID and compare to the `d` tag to validate the event. Valid records for a name can only be accepted when published by the name's owner.

## Name format

It is necessary to limit the characters used in names. While it might be tempting to allow any valid UTF-8 string, there are good reasons not to do this. In the Unicode standards, there are sometimes different ways to the construct the same character, invisible characters, or "whitespace" characters that may not necessarily be rendered, etc. This could allow for malicious individuals to trick unsuspecting users into clicking/pasting incorrect names.

While it is desirable to have a wide range of characters and languages be usable, for the time being it is necessary to restrict the use of characters to the basic characters typically used in domain names today.

Names must match the following regular expression `[0-9a-z\-]{3,43}` and must be ignored by indexers otherwise.

## Test vectors

```yaml
# Key data for vectors
keys:
  sk: 2a1a6fe815e69e09c95093ff04b7deee2182a56c55c8d4f07a3872a7f77208dc
  pk: 60de6fbc4a78209942c62706d904ff9592c2e856f219793f7f73e62fc33bfc18

# On chains claims for testing OP_RETURN validity
claims:
  - name: hello-world
    op_return: 4e4f4d0000e5401df4b4273968a1e7be2ef0acbcae6f61d53e73101e2983
    valid: true
  - name: hello-world
    op_return: 4e4f4d0001e5401df4b4273968a1e7be2ef0acbcae6f61d53e73101e2983
    valid: false

# Checking name validity
names:
  - name: hello-world
    valid: true
  - name: hello!
    valid: false
  - name: ld
    valid: false
  - name: abcdefghijklmnopqrztuvwxyzabcdefghijklmnopqrztuvwxyz
    valid: false
```

## Changes and updates

**2023-11-10**:
  - Test vectors added for on-chain claims and name regex validation

**2023-11-04**:
  - This protocol document has been re-issued as part of the set of `NOM` specifications.

**2023-09-23**:
  - Protocol v0 is now deprecated in favor of protocol v1. V1 is much simpler to track on chain, and much simpler to create indexers.
  - Squatting section removed as it is not related to this spec.

**2023-09-19**:
  - Given the issues cited [here](https://github.com/ursuscamp/nomen/issues/6), the design of transfers in `0x00` is not good and has been removed. As of the time of publication, no transfers have been issued by any users, so this is a not a breaking change.
  - A new version will be issued which will enable transfers, and an upgrade path will be available for version `0x00` names to `0x01` names. However, in order to link the `0x00` to `0x01` names on chain, the upgrade transaction will require the names to be put on chain in plain text, necessitating limiting them to a maximum of 43 bytes for now (80 byte OP_RETURN maximum - 5 bytes for Nomen metadata - 32 bytes for a public key). As of the time of publication, no names longer than 43 bytes have been issued, so this is a non-breaking change.