# Global Messaging Protocol

Version: 1.0, 2017-12-28

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

- The message is encoded by public key of the receiver and decoded by his private key.

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
- Example:
```
{"type":"<other type>",
"field1":value1,
"field2":value2,
...
}
```

## Messaging Blockchains
- Implemeting GMP on currency blockchains like BTC or BCH, or smart contract blockchains like ETH, would add to the already overloaded capacity and storage of them. The best way is to use a specialized blockchain for messaging only. An Example of that is the Hush blockchain.

## Hushlist comparison
- While [Hushlist](https://github.com/leto/hushlist) is a good start. However, it is different from GMP and it has 2 issues:
1. The idea of having mail lists meta data outside the blockchain, will make it centralized and under censorship. 
2. The list capacity is only 54 which is too low. Also increasing it in following versions will include sending the message through multiple transactions which means more transaction fees.

- However in GMP, anyone can send a global message tied to a location (optional), and has a topic, which will enable anyone to filter the messages by location, topic, or name. No need to maintain/worry about subscription lists.
