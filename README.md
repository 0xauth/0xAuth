# 0xAuth
A minimalistic protocol for decentralized authentication

## Introduction

A cryptocurrency wallet can be used to sign arbitrary strings using the private key of the active account. This feature can be used to generate an authentication protocol which eliminate any need for usernames and passwords.

Websites already use OAuth and similar protocols to allow an external identity provider, like Facebook or Google, to authenticate their users. There are many advantages in using a wallet for the authentication. The most important is that a website autenticates a user interacting with the user itself, without passing for an identity provider. This implies that nobody can stop a website from using their API, except the user itself. It also implies that there is no need for subscribing with an identity provider, generate API key and secret, etc.

All that is needed is a signature verification.

### The sign up flow
* The user loads a website which requires authorization, exposing its default account address.
* The website's frontend recognizes address and platform and passes the data to the backend.
* The website's backend generates an authorization token.
* The website's frontend generates a request to the wallet for signing the authorization token.
* The wallet asks the uses to approve the signature and signs the authorization token.
* The frontend receives the signatures and send it to the backend.
* The backend verifies the validity of the signature and generates a token to be used for the future requests.

### The sign in flow

It is very similar to the signup flow. In fact, the website checks if an authentication session is active and if not replicate the points 2, 3, 4, 5, 6 and 7 of the sign up process. The difference is that in a real scenario, during the signup the user is probably saved in a database, while during the sign in the backend will just check if the user has been previously saved.

##The authorization token
The authorization token contains the data necessary for the authentication.

Here an example of a valid authorization token:
```
0xAuth:1;com.example.Auth;1556997887:1559000000;fb7c
```

Splitting the string by semicolon and colon, we have the following array:
```
[
  [ '0xAuth', '1' ],              // protocol and version
  [ 'com.example.Auth' ],           // reverse domain name notation
  [ '1556997887', '1559000000' ], // Unix timestamp at creation (required) and expiration (optional)
  [ 'fb7c' ]                      // extra field, starting with a random string
]
```
If any of the token elements cointans a reserved character (colon and semicolon), the character can be encoded using a backslash. For example, if the 4th element contains ['aed4', '0:0', 'a;b'] the resulting field would be:
```
aed4:0\:0:a\;b
```

The backslash itself is always encoded as `\\`.

0xAuth supports any type of blockchain and address (and any public/private key schema).

The first implementation (Node/JS) will support initally Tron, after Ethereum, and later Tezos and many other chains.

Examples of accepted address fields are:
```
eth:0x4811a2cd0255ebf0533e373e48faec692c45b193
trx:TGYGnEiyHZrR8XjitLjkrHiGmPysYXCUCm
tez:tz1NUbGVwYkam4cVe7SZoweu75HnZL5D98hW
```

## The signed token
When the JSON authorization token is signed, a new field with the signature is added to the authorization token. The new field contains info about the signature and the signature itself. For example, if the authorization token has been signed by Metamask using v_2, the signature string is
```
0xb646ff642a60680cf6f5d7ce650e2fd2df26c175ec7990f1e2a65ad8fdfdb105786a36763fb6bf9f30bdd5175c748723330e5fe0e843bbbb034948b2cf23f2e21c,web3,2
```
and the entire signed token is something like:
```
0xAuth:1;com.example.Auth;1556997887;fb7c;eth:0x4811a2cd0255ebf0533e373e48faec692c45b193;0xb646ff642a60680cf6f5d7ce650e2fd2df26c175ec7990f1e2a65ad8fdfdb105786a36763fb6bf9f30bdd5175c748723330e5fe0e843bbbb034948b2cf23f2e21c,web3,2
```
Splitting it following the approach used before, this becomes
```
[
  [ '0xAuth', '1' ],
  [ 'com.example.Auth' ],
  [ '1556997887' ],
  [ 'fb7c' ]
  [ 'eth', '0x4811a2cd0255ebf0533e373e48faec692c45b193' ],
  [ '0xb646ff642a60680cf6f5d7ce650e2fd2df26c175ec7990f1e2a65ad8fdfdb105786a36763fb6bf9f30bdd5175c748723330e5fe0e843bbbb034948b2cf23f2e21c', 'web3', '2' ]
]
```

The two added elements compared with the authorization token specify blockchain and address of the signer, and the signature itself.
This way, the signed token contains anything is needed to verify it.

For Tron, the signed token would be something like:
```
0xAuth:1;com.example.Auth;1556997887;fb7c;trx:TXtMUJpGugXqoCRdvzEGPXqRZU7vbf2SnF;0x95d1bc003c5648cf410b2067294a5ede28bcd76ff56b8c4db83377307599c8e15b52c62b211be715be9601cf195c42463aaf80196598f972ccb5e04457ea171f1b:tronweb:1
```

## The typical use case
The process describes above is the base.

In a real scenario the user's address can be saved in a database and associated to a profile.

Inside companies, an employee's address can be pre-approved waiting for confirmation at the first login.

Also, an authorization token can be released to the user, via cookies or jwt, to avoid to repeat the verification.
Identity

0xAuth focuses on the authorization process. But a user's wallet can be associated with social accounts, national IDs, etc.
Identity providers like uPort or Origin Protocol base their approach on standards like the ERC725 and ERC735, acting as guaranters of the identity.

Self-claiming identity systems like [Tweedentity](https://tweedentity.com), instead, use an approach similar to 0xAuth for verification.

In the first version, Tweedentity was generating a token like this
```
tweedentity(0x93a5,twitter/946957110411005953,0x3dcbf1b6aa2b2f772b4109a60df1bd425a1485bb6dc163b3365612d5fbc0dbe17f3f6463bfed44c34e13493016438bf9ce8778334873ee73140d4418944a3c311b,3,web3;1)
```
Where
`tweedentity` is an arbitrary identifier
`0x93a5` is the beginning of the ETH address
`946957110411005953` is the Twitter ID of the self-claiming user
`0x3dcbf1b6aa2b2f772b4109a60df1bd425a1485bb6dc163b3365612d5fbc0dbe17f3f6463bfed44c34e13493016438bf9ce8778334873ee73140d4418944a3c311b` is the signature
`3` is the version of the signature algorithm
`web3` is the library providing the signature
`1` is tweedentity version

The version 2 of Tweedentity will extend the 0xAuth protocol. It will add extra values, like the twitter user id, in the x field (nounce, Tweedentity version, identity provider and ID) and a signed token will be something like this:
```
0xAuth:1;com.tweedentity;1556997887;98fa,1,t,946957110411005953;eth:0x4811a2cd0255ebf0533e373e48faec692c45b193;0xa1c056f46db4a4c6d69166a5f0e534f4e10f3b7e8e7c45f9d9b1b9c8dbbc326456ee488bc69dc2b232be0d88004e6a0ad40344560b6fc0a35ca48c08eb2bc32b1b,web3,3
```
## Implementations

* [0xauth-js](https://github.com/0xauth/0xauth-js) — Javascript implementation supporting Solidity addresses for Tron and Ethereum (coming soon), available on the npm registry as `0xauth`.

## License
MIT

## Author
[Francesco Sullo](https://francesco.sullo.co), <francesco@sullo.co>

## Version
draft-0.0.3, May 3rd, 2019
