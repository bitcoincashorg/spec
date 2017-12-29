# Global Messaging Protocol

**Author:** Amr Aboelela; December 28, 2017

## Introduction

- Suggestion to add Global Messaging Protocol over the blockchain protocol, in which we can add things like direct messages, or global messaging.
- What we are trying to achieve here is to build a Global Messaging ability, like Twitter but without centralized authority. Also having to pay for those messages will keep them short in number, and increase the usability of crypto currencies in general.

## Direct Message

- Example:

```
{"t":"dm",
"tx":"<message encoded using public key of the receiver>",
"l":"<url encoded using public key of the receiver>"
}
```

- t: type (optional, by default it is gm (Global Message))
- tx: Text content
- l: url link
- The text and the url are encoded by public key of the receiver and decoded by his private key.
- The message is required to have either text or url fields or both

## Global Message
- Example:
```
{"t":"gm",
"lt":137.567,
"ln":43.7698,
"n":"Jack",
"tp":"Hello",
"tx":"Hello World"
}
```

- lt: latitude (optional)
- ln: longitude (optional)
- n: name (optional)
- tp: topic (required)

- Short form example:

```
{"tp":"Hello","l":"www.helloworld.com/hello.html"}
```

- The URL refers to any resource in the internet, e.g. html, txt, md, jpg, mpg, png, pdf, ...
- The reason we have latitude and longitude is if the user wants to associate this global message to a specific location in the world.
- In case of Global Message, there will be a regular transaction fee, but there is no need to specify sending money info from one address to another, and if this is required by the existing protocol, 
so we can just say something like transfer 0 coins from sender address to sender address.

## Other types
- We can add any other message types later as needed.
- Example:
```
{"t":"<other type>",
"field1":value1,
"field2":value2,
...
}
```

## Storage
- These messages will be stored in the blockchain, in the memo field. However, currently Bitcoin blockchains has memo field with size 80 bytes only, which is too small. To go around this limitation we can put in the content field a url to any online resource, text, image, or video.


## Retrieval
- Users can retrieve the messages they interested in based on location, name, or topic. The transactions which has global message could be identified by having a money transfer of amount 0.0 from one address to the same address.

## HushList comparison
- [HushList](https://github.com/leto/hushlist) is different from GMP and it has 2 issues:
1. The idea of having mail lists meta data outside the blockchain, will make it centralized and under censorship. 
2. The list capacity is only 54 which is too low. Also increasing it in following versions will include sending the message through multiple transactions which means more transaction fees. However in GMP, anyone can send a global message tied to a location (optional), and has a topic, which will enable anyone to filter the messages by location, topic, or name. No need to maintain/worry about subscription lists.
3. Hushlist emphasys the use of zaddr. In GMP, hiding the address is not really important, as the user can always change his address, they are almost free. Also we need to know user addresse so other users be able to block them if needed.

## References

1. David Mercer 2017 [HushList Protocol Specification](https://github.com/leto/hushlist/blob/master/whitepaper/protocol.pdf)
2. Radical App International 2017 [Mercury Protocol](https://www.mercuryprotocol.com/files/Mercury_Protocol_whitepaper.pdf)
