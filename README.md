# TAP Introduction
TAP is an extendible OrdFi-enabling protocol that includes - but is not limited to - the TAP token standard.

TAP works entirely without the use of L2 chains or other overly complex mechanics, but utilizes tapping - a simple mechanism to verify transactions within the protocol.

The goal of TAP is not to replace or compete with BRC-20 but instead embraces it and allows for new features that will be consistently added to the protocol by community driven governance.

The BRC-20 ticker lengths 1,2 and 4 are reserved for future interordinal bridging between BRC-20 and TAP Tokens and cannot be deployed on TAP.

# Structure

TAP consists of two parts, external and internal. The external part acts in the exact same way like BRC-20 does. 

External: Marketplaces clone their existing BRC-20 infrastructure to get connected to TAP. From there on out, TAP tokens can be traded like regular BRC-20 tokens. Nothing more needs to be done.

Internal: Users benefit from features such as token staking and swaps. Mass-sending of tokens are already available. The community will decide through the use of $TRAC (BRC-20) which features will be added or updated.

Alongside these specs, there is already TAP protocol tracking available on https://trac.network. 

The description for the public endpoint will be published in the coming days on Github. This allows developers to embrace the new standard. After the official release of Trac, developers may self-host TAP tracking and create their own endpoints.

# Specs
#### External
As mentioned above, TAP tokens work in the exact same way as BRC-20 tokens. There are however a couple of minor modifications required for indexers:

|| TAP | BRC-20 |
|-------------| ------------- | ------------- |
| Allowed ticker lengths | 3 and 5 to 32 (UTF16)  | 4 letters |
| Protocol | tap  | brc-20  |
| Deploy op | token-deploy  | deploy  |
| Mint op | token-mint  | mint  |
| Transfer op | token-transfer  | transfer  |

#### Examples

```javascript
{ 
  "p": "tap",
  "op": "token-deploy",
  "tick": "tap",
  "max": "21000000",
  "lim": "1000"
}
```

```javascript
{ 
  "p": "tap",
  "op": "token-mint",
  "tick": "tap",
  "amt": "1000"
}
```

```javascript
{ 
  "p": "tap",
  "op": "token-transfer",
  "tick": "tap",
  "amt": "100"
}
```

#### Internal

Internally, there exists a new function called "token-send", which enables mass-transfer of different tokens to many recipients.

There will be more internal functions, so it may be useful to check back here once in a while.

The specs for "token-send" are defined as follows:

- Users inscribe a "token-send" inscription to their Bitcoin address.
- After the transaction confirmed, the inscription has to be resent to the same address for confirmation (tapping).
- Upon tapping, the Bitcoin address must own the the amounts of tokens that are given with the inscription.
- Only if the "token-send" inscription is tapped, sending tokens will be performed.
- The receiver addresses must be carefully validated: must be valid Bitcoin addresses, trimming is allowed, addresses starting with bc1 have to be lowercased.
- "token-send" is atomic upon inscribing (before tapping): all amounts, tickers and addresses must be valid.
- Upon tapping, invalid token sends may be skipped (e.g. insufficient funds).
- Each successful token send must credit the given amounts to recipient and be removed from the sender's balance.
- Since neither BRC20 nor TAP tokens are "cursed-aware", indexers need to check for negative inscription numbers to separate cursed tokens from non-cursed ones.
- After discontinued ord wallet support for _new_ cursed Ordinals, the token-transfer function becomes disfunctional and cursed tokens can only be transferred using "token-send" internally.

#### Example

The below example will send 1000 tap tokens to each address. Tokens and amounts can be mixed.
There is no restriction on the amount of items (receivers).

```javascript
{
  "p" : "tap",
  "op" : "token-send",
  "items" : [
     {
      "tick": "tap",
      "amt": "10000",
      "address" : "bc1p9lpne8pnzq87dpygtqdd9vd3w28fknwwgv362xff9zv4ewxg6was504w20"
     },
     {
      "tick": "tap",
      "amt": "10000",
      "address" : "bc1p063utyzvjuhkn0g06l5xq6e9nv6p4kjh5yxrwsr94de5zfhrj7csns0aj4"
     }
  ]
}
```

#### Outlook

This document and the tracking for the TAP protocol will be continuously worked on and updated. Feel free to join the Discord if you have questions: https://discord.gg/sPyYDa5q6P
