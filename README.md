# Nomen NOMs

These documents present proposed standards for Nomen implementations. It's modeled after the [NIP](https://github.com/nostr-protocol/nips) system, since Nomen is intended to be a Nostr-native protocol.


| #               | Description             |                          |
|-----------------|-------------------------|--------------------------|
| [01](nom-01.md) | Protocol v0             | `mandatory` `deprecated` |
| [02](nom-02.md) | Protocol v1             | `mandatory`              |
| [03](nom-03.md) | Recognized record pairs |                          |
| [04](nom-04.md) | Relay-published indexes |                          |


## Nostr events

| Kind    | Description                | NOM             |
|---------|----------------------------|-----------------|
| `38300` | Name records               | [02](nom-02.md) |
| `38301` | Relay-published index data | [04](nom-04.md) |
