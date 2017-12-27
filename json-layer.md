# JSON Layer Protocol

Version: 1.0, 2017-12-27

## Introduction

Suggestion to add the JSON Layer as option in transactions, in which we can add things like direct messages, or global messaging.

## Direct Message

- Example:

```
{"type":"dm"
"version": "1.0"
"encoding":"pk-sha256"
"message":"<message encoded using public key of the receiver>"
}
```
- The reason we have version number, is to be able to change the json protocol under a specific type later on.

## Global Message
- Example:
```
{"type":"gm"
"version":"1.0"
"latitude":137.567
"longitude":43.7698
"message":"Hello world"
}
```
- The reason we have latitude and longitude is if the user wants to associate this global message to a specific location in the world.
- In case of Global Message, there will be a regular transaction fee, but there is no need to specify sending money info from one address to another, and if this is required by the existing protocol, 
so we can just say something like transfer 0 coins from sender address to sender address.

## Other types
- We can add any other message types later as needed.
