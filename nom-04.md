# NOM-04

## Relay-published indexes

Nomen indexers may provide their data in different ways, such as an API, a web page or an archive. However, as Nomen is intended to be a Nostr-native protocol, it would make sense for indexers to publish their data to Nostr relays in a well understood format. This will allow Nostr clients that support Nomen to go to relays as the sole source of data without needing to make further external network calls.

## Event structure

Nomen indexers should publish a single event per name. It will be a __parameterized replaceable__ event, per the [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) specification. When name data changes (e.g. new records or ownership transfers) then the event will be republished and replaced.

The fields to be aware of:

| Event field | Expected value                         |
|-------------|----------------------------------------|
| `pubkey`    | The __indexer's__ pubkey               |
| `kind`      | `38301`                                |
| `d` tag     | the indexed name                       |
| `content`   | JSON serialized name object (see blow) |

The `content` will be a serialized form of the following data:

```json
{
  "name": "indexed named, should match the d tag",
  "pubkey": "hex-encoded pubkey of name's owner",
  "records": {
    "KEY1": "VALUE1",
    "KEY2": "VALUE2"
  }
}
```

`r` should be replaced with the canonical records as understood by the indexer.

## DNS-based indexer pubkey

For an indexer with a web presence, such as the [Nomen Explorer](https://nomenexplorer.com), they may respond to `GET` requests under: `https://<domain>/.well-known/nomen.json` with a JSON object like this:

```json
{
  "indexer": {
    "pubkey": "indexer's hex-encoded pubkey"
  }
}
```

Clients may choose to check this convenience to locate the pubkey to use when searching for relay-published indexes.