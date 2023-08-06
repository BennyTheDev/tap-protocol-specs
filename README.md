# TAP Introduction
TAP is an extendible OrdFi-enabling protocol that includes - but is not limited to - the TAP token standard.

TAP works entirely without the use of L2 chains or other overly complex mechanics, but utilizes tapping - a simple mechanism to verify transactions within the protocol.

The goal of TAP is not to replace or compete with BRC-20 but instead embraces it and allows for new features that will be consistently added to the protocol by community driven governance.

The BRC-20 ticker lengths 1,2 and 4 are reserved for future interordinal bridging between BRC-20 and TAP Tokens and cannot be deployed on TAP.

#### Structure

TAP consists of two parts, external and internal. The external part acts in the exact same way like BRC-20 does. 

External: Marketplaces clone their existing BRC-20 infrastructure to get connected to TAP. From there on out, TAP tokens can be traded like regular BRC-20 tokens.

Internal: Users benefit from features such as token staking and swaps. Mass-sending of tokens are already available. The community will decide through the use of $TRAC (BRC-20) which features will be added or updated.

Alongside these specs, there is already TAP protocol tracking available on https://trac.network. 

The endpoint description will be published in the coming days on Github.

#### Specs

As mentioned above, TAP tokens work in the exact same way as BRC-20 tokens. There are however a couple of minor modifications required for indexers:

| TAP | BRC-20 |
| ------------- | ------------- |
| Allowed ticker lengths 3 and 5 to 32 (UTF16)  | 4 letters |
| Protocol: tap  | brc-20  |
| Deploy op: token-deploy  | deploy  |
| Mint op: token-mint  | mint  |
| Transfer op: token-transfer  | transfer  |
