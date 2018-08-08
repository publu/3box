#A Proposal for User-Managed Data

This proposal is to introduce a user-controlled space for dapp-related data storage. Right now dapps that store non-blockchain related data about the user will use a server to do this operated by the dapp.

We propose a server-side service and a client library that will associate an encrypted key-value store to an Ethereum address. The Ethereum address will authorize a dapp to access its data store by signing a message. The current version of the libraries require no changes to wallet software (MetaMask, Cipher Browser, etc) used to access the dapp.

You can explore our code at https://github.com/uport-project/userspace and community at https://chat.uport.me/#/room/#3box:chat.uport.me. If you’re interested in our project, we would love to collaborate. 

##Motivation

Dapps that want to associate data to their users will eventually need to store data off-chain and link it to the Ethereum address of the user. This kind of data can be for instance a user-chosen username, an email address or settings or preferences that shape the experience of the dapp.

The way that this data is stored right now is most commonly done by using a server operated by the dapp developers, making this data siloed and only useable to that particular dapp. Additionally, the overhead of managing this user data in a secure, compliant way can become costly.

If the user instead had their own private data store that the dapp could hook into the dapp could forgo running their own server and rely on the user themselves to handle this data. This model opens the possibility of sharing data between dapps to improve user experience (such as username, emails, contact lists etc).

You might consider implementing a user-managed data architecture if you…

* are a dapp developer looking to decentralize your dapp’s data infrastructure
* are a dapp developer looking to share data with other dapps
* are a dapp developer looking to onboard rich user accounts with one click
* are a dapp developer looking to monetize data generated by your dapp [future]
* are a dapp developer looking to buy data generated by other dapps [future]

* are a wallet/signer dev looking to facilitate a seamless web3 user experience

##Architecture
The current architecture is very simple and require no changes to existing wallets or dApp browsers.

The library in the dApp front end will first ask the user to sign a message “Open UserSpace” with the private key corresponding to their current address. This will create a signature S which is returned to the dApp front end. The signature is used as a source of entropy to generate one elliptic curve key pair (k, K) used for authentication and one symmetric key e used for data encryption. The benefit of this method is that it works with Ethereum wallets today without modification. The downside is security - if a malicious dapp asks you to sign this message then this app can steal the private keys needed to download and decrypt your data. In the future it’s possible that Ethereum wallets will have native support for encryption keys in which case the design can be improved.

The server-side component will associate the authentication key K with an encrypted key-value store. When the user first creates a data entry D in the dApp front end they will use the encryption key e to encrypt the data record and post the encrypted data to the server. The data record will also be signed by the authentication key k to prove to the server that the data comes from the right user.

When retrieving the data from the server the front end downloads the whole database from the server and then decrypts the data in the front end. When querying the database in the dapp only the local database is queried.

###Potential Use Cases
A shared-access, user-managed database can facilitate a cohesive web3 experience between different dapps, devices, browsers, and wallets that choose to collaborate. Here are some ways dapps can collaborate using this technology.

With this technology developers can:

* Build a universal contact list / web3 address book that users can use across every wallet and social dapp
* Keep token watch lists, fiat conversion currencies, and other default wallet settings synced across every platform and wallet
* Provide accurate recommendations to users based on their history of on-chain and off-chain interaction and behavioral patterns across other platforms
* Collaborate on a shared reputation system
* Monetize user behavioral data in a fair and responsible way, by allowing users to own consent and share in the benefit
* Build richer cross-dapp partnerships and user experiences
* And more...

##Implementation and Interface

We have a github repository here with our first implementation:

https://github.com/uport-project/userspace

The front end library exposes a `UserSpace` object that will contain the decrypted key-value store with the following exposed functions:

get(key) - returns the value corresponding to the key
getAll() - returns all key-value pairs as a JavaScript object
set(key, value) - sets {key: value} as a new data entry and pushes it to the server
remove(key) - removes the data entry corresponding to the key


##Demo
We also have the following demo that you can play with using a dapp browser like MetaMask:

https://developer.uport.me/userspace/example/

##Future Improvements

Some unresolved questions with the current version:

* Security: If a malicious app tricks a user into signing the “Open UserSpace” message this app has access to the users authentication keys and encryption keys, allowing them permanent access to read and write the users data. This leads to the possibility of phishing attacks and other dangers.
* Namespacing: How to make sure that dapps chose names of the keys for data such that they don’t clash with the names chosen by other dapps
* Self Sovereignty: How can we allow the user to run their own instance? DappNode?
* Storage Options: How can we allow the user to select the service where their encrypted data is actually stored?
* Improved data access control and permissioning
* Payment Protocols: How can we allow partners to make money
* Data Schemas: Data is now very simple, do we need common formats for data that is shared between dapps?