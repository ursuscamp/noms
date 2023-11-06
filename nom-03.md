# NOM-03

## Recognized record pairs

`author:ursuscamp`

Nomen record event types `38300` (see [NOM-02](nom-02.md)) contain a serialized JSON object on the `content` field. The key-value pairs in the object represent the "records" associated with the name.

Formally, these can be anything, and it is is expected many will arise naturally. However, presented here are some formally recognized record types:

| KEY NAME | DESCRIPTION                       |
|----------|-----------------------------------|
| `IP4`    | IPv4 address for a host or server |
| `IP6`    | IPv6 address for a host or server |
| `NPUB`   | Nostr NPUB                        |
| `EMAIL`  | Owner email address               |
| `MOTD`   | A general message from the owner  |
| `WEB`    | Full link for website             |

Take special note of the `NPUB` value. The pubkey that owns a Nomen name need not necessariliy be the same name that someone uses publicly to make posts. If a Nomen name was an `NPUB` key, that public key should instead be the user's public presence. The owning key may then just be a keep kept in cold storage to prevent thefts, and only brought out to sign events for record changes or a transfer.

## Changes and updates

**2023-11-04**:
  - This protocol document has been re-issued as part of the set of `NOM` specifications.