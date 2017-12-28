# Global Messaging Protocol

Version: 1.0, 2017-12-27

## Introduction

- Suggestion to add Global Messaging Protocol over the blockchain protocol, in which we can add things like direct messages, or global messaging.
- What we are trying to achieve here is to build a Global Messaging ability, like Twitter but without centralized authority. Also having to pay for those messages will keep them short in number, and increase the usability of crypto currencies in general.

## Direct Message

- Example:

```
{"type":"dm",
"message":"<message encoded using public key of the receiver>"
}
```

## Global Message
- Example:
```
{"type":"gm",
"latitude":137.567,
"longitude":43.7698,
"name":"Jack",
"topic":"Hello",
"message":"Hello world"
}
```
- The reason we have latitude and longitude is if the user wants to associate this global message to a specific location in the world.
- In case of Global Message, there will be a regular transaction fee, but there is no need to specify sending money info from one address to another, and if this is required by the existing protocol, 
so we can just say something like transfer 0 coins from sender address to sender address.

## Other types
- We can add any other message types later as needed.

## Storage
- These messages will be stored in the blockchain, either in the memo field or any other field. Currently the Bitcoin blockchain has memo field with size 80 bytes only, which is too small. So we need to figure out another field. Otherwise we can implement this protocol in other forks like Zcash, which has memo field with 512 bytes.

## Retrieval
- Users can retrieve the messages they interested in based on location, name, or topic. The transactions which has global message could be identified by having a money transfer of amount 0.0 from one address to the same address.
