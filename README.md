# TAP Introduction
TAP is an extendible OrdFi-enabling protocol that includes - but is not limited to - the TAP token standard.

TAP works entirely without the use of L2 chains or other overly complex mechanics, but utilizes tapping - a simple mechanism to verify transactions within the protocol.

The goal of TAP is not to replace or compete with BRC-20 but instead embraces it and allows for new features that will be consistently added to the protocol by community driven governance.

The BRC-20 ticker lengths 1,2 and 4 are reserved for future interordinal bridging between BRC-20 and TAP Tokens and cannot be deployed on TAP.

# Structure

TAP consists of two parts, external and internal. The external part acts in the exact same way like BRC-20 does. 

**External:**

To get connected to TAP, marketplaces/wallets clone their existing BRC-20 infrastructure. Then either TAP's internal functions (see below) have to be implemented or endpoints that implement those have to be used to verify all balances (balance, available, transferable). From there on out, TAP tokens can be traded like regular BRC-20 tokens on their platforms. [updated Aug. 13th, 2023]

**Internal:**

Users benefit from features such as token staking and swaps. Mass-sending of tokens are already available. The community will decide through the use of $TRAC (BRC-20) which features will be added or updated.

Since neither BRC20 nor TAP tokens are "cursed-aware", indexers with cursed support need to check for negative inscription numbers to separate cursed tokens from non-cursed ones. [added Aug. 8th, 2023]

Alongside these specs, there is already TAP protocol tracking available on https://trac.network. 

Trac's public endpoint (https://github.com/BennyTheDev/trac-tap-public-endpoint) allows developers to embrace the new standard. After the official release of Trac, developers may self-host TAP tracking and create their own endpoints.

Protocol updates will be announced in advance - with a reasonable grace-period - to allow indexers to follow. [added Aug. 13th, 2023]

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

Leading dashes in tickers are not allowed for any function's "tick" attribute in its json root. (internal or external). This restriction is valid until the support for new cursed for Ordinals ended. From the first block after ended cursed support, the leading dash limitation must be lifted. From this moment on, already existing cursed tokens must be addressed with dash prefixes and _new_ tokens may have leading dashes. [added Aug. 22nd, 2023]

If the indexer does _not_ support cursed tokens, a check must be added that excludes tokens derived from cursed Ordinals, for all of the external functions above and below. (inscription number is < 0). [added Aug. 8th, 2023]

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

# Internal

#### token-send

Internally, there exists a new function called "token-send", which enables mass-transfer of different tokens to many recipients.

There will be more internal functions, so it may be useful to check back here once in a while.

The specs for "token-send" are defined as follows:

- Users inscribe a "token-send" inscription to their Bitcoin address.
- After the transaction confirmed, the inscription has to be resent to the same address for confirmation (tapping).
- Upon tapping, the Bitcoin address must own the the amounts of tokens that are given with the inscription.
- Only if the "token-send" inscription is tapped, sending tokens will be performed.
- The receiver addresses must be carefully validated: must be valid Bitcoin addresses, must be trimmed, addresses starting with bc1 have to be lowercased.
- "token-send" is atomic upon inscribing (before tapping): all amounts, tickers and addresses must be syntactically valid.
- Upon tapping, invalid and semantically incorrect token sends must be skipped (e.g. insufficient funds, invalid amounts or invalid data types).
- Each successful token send must credit the given amounts to recipient and be removed from the sender's balance.
- Each send item must exclusively operate on available balances, not overall balances (available = balance - transferable). [added Aug. 8th, 2023]
- There is _no_ difference between cursed and non-cursed tokens in the "items" attribute. Cursed tokens have to be addressed using a leading dash (unlike functions' "tick" attribute in the json root). This is to enable token-send to mix cursed and non-cursed tickers in the same transaction. [added Aug. 22nd, 2023]

#### Example

The below example will send 1000 tap tokens to each address. Tokens and amounts can be mixed.
There is no restriction on the amount of items (receivers).

```javascript
{
  "p" : "tap",
  "op" : "token-send",
  "items" : [
     {
      "tick": "-tap",
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

#### token-trade

The internal function set "token-trade" consists of 3 functions to allow text-inscription based trading. By inscribing a trade inscription as seller, a trade is being added to the Ordinals collection and can be picked up by any buyer for the duration of a valid period in blocks. 

Upon filling a trade by the buyer, an optional fee receiver address may be added to receive fixed 0.3% fees in tokens per filled trade. Fees are applied on top of the purchased token and is being paid by the buyer. Fees give inscription services and marketplaces a way to benefit from internal trading, while buyers benefit from the convenience and guidance of their services.

To inscribe a trade, a seller has to inscribe a text inscription in the following format:

#### Creating a Trade

```javascript
{
  "p" : "tap",
  "op" : "token-trade",
  "side" : "0",
  "tick" : "tap",
  "amt" : "1",
  "accept" : [
    {
      "tick" : "buidl",
      "amt" : "0.1"
    },
    {
      "tick" : "based",
      "amt" : "0.2"
    }
  ],
  "valid" : "900000"
}
```

As soon as the inscription above is inscribed, the trade is open and ready to be filled under the following conditions:

- All attributes are mandatory as of the example above.
- Side = 0 specifies that this is a trade function by a seller.
- The seller must own the offered token amount in the moment of filling the trade by a buyer.
- The offered token amount must be deducted from the available balances, not overall balances (available = balance - transferable).
- The tokens defined in the "accept" attribute represent accepted tokens for the trade.
- If any of the accepted tokens are being used by the buyer, the trade concludes. 
- The tokens defined in the "accept" attribute must be deployed in the moment of filling the trade by a buyer.
- The block specified in the "valid" attribute must be a future block (inclusive).
- The trade is invalid and not being indexed if the current block is larger than "valid".
- If the offered tokens is cursed, then the inscription has to be inscribed as a cursed one.
- Ticks within the "accept" attribute must be unique. Only the first must be indexed if there is more than one of the same.
- Cursed tokens defined in the "accept" attribute have to be prefixed with a dash (this is to allow mixed cursed/non-cursed trading).
- As soon as cursed support ends for new inscriptions (1st block), the offered token ticker has to be prefixed with a dash, too if it originally has been a cursed token.
- After inscribing the function, sellers have to send the inscription to themselves to approve (tapping).

#### Cancel a Trade

In order to cancel an existing, non-expired trade. The following function has to be inscribed and tapped:

```javascript
{
  "p" : "tap",
  "op" : "token-trade",
  "side" : "0",
  "trade" : "<inscription id>"
}
```

- All attributes are mandatory as of the example above.
- Side = 0 specifies that this is a trade function by a seller.
- The "trade" attribute has to be populated with the inscription id (_not_ number) of the referenced trade inscription above.
- The trade will only cancel if the referenced trade inscription is owned by wallet address that intends to cancel.
- After a trade has been cancelled, no trades for it can occur.
- After inscribing the function, sellers have to send the inscription to themselves to approve (tapping).

#### Fill a Trade

Valid trades are fillable by inscribing a token-trade inscription that specifies which offered token to purchase, with a reference to the original trade inscription. Additionally, an optional fee receiver may be specified. The fee receiver will earn fixed 0.3% of the token that the buyer is purchasing. The fee is applied on top of the purchase amount.

```javascript
{
  "p" : "tap",
  "op" : "token-trade",
  "side" : "1",
  "trade" : "<inscription id>",
  "tick" : "based",
  "amt" : "0.2",
  "fee_rcv" : "<fee receiver address>"
}
```

- Except "fee_rcv", all attributes are mandatory as of the example above.
- Side = 1 specifies that this is a trade function by a buyer.
- The "trade" attribute must match the original trade inscription id (_not_ number).
- The "tick" and "amt" attributes must exactly match one of the offered tokens from the referenced trade inscription.
- The buyer must _not_ fill all accepted tokens as of the referenced trade but exactly one.
- The ticker must be deployed in order to get the above indexed.
- The buyer must own the specified token and it's amount + 0.3% trading fees if the "fee_rcv" attribute is set.
- If the "fee_rcv" attribute is set, it must be a valid Bitcoin address.
- Upon indexing, the Bitcoin address in "fee_rcv" must be trimmed and lowercased if the address starts with "bc1".
- If the token in the "tick" attribute is a cursed one, the inscription has to be inscribed as cursed.
- As soon as cursed support ends for new inscriptions (1st block), the "tick" attribute must reference cursed tokens with a leading dash.
- After inscribing the function, buyers have to send the inscription to themselves to approve (tapping).
- If the current block upon tapping is larger than the block in the "valid" attribute as of the referenced trade, filling will fail.
- If all conditions are met, the token and its exact amount is being sent to the seller and the offered token is being sent to the buyer. If fees apply, the fee will be sent to the fee receiver.
- Fees are only applicable on amounts and decimals that allow 0.3% to be applied. This means there may be zero fees if not applicable.
- A trade has been filled successfully from an indexer point of view, if all conditions are met and all balances are credited correctly.

#### Outlook

This document and the tracking for the TAP protocol will be continuously worked on and updated. Feel free to join the Discord if you have questions: https://discord.gg/sPyYDa5q6P
