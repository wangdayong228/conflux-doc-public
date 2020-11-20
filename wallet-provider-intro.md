[toc]
# Wallet provider standard
Conflux wallet need injects a global API into websites visited by its users at window.conflux. This API allows websites to request users'  accounts, read data from blockchains the user is connected to, and suggest that the user sign messages and transactions. The presence of the provider object indicates an Conflux user.

The Conflux JavaScript provider API is specified by EIP-1193, **and the difference of conflux is replcing the rpc name prefix from "eth_" to "cfx_"**
## EIP-1193

## Summary

A JavaScript Ethereum Provider API for consistency across clients and applications.

## Rationale

The purpose of a Provider is to _provide_ a consumer with access to Ethereum.
In general, a Provider must enable an Ethereum web application to do two things:

- Make Ethereum RPC requests
- Respond to state changes in the Provider's Ethereum chain, Client, and Wallet

The Provider API specification consists of a single method and five events.
The `request` method and the `message` event alone, are sufficient to implement a complete Provider.
They are designed to make arbitrary RPC requests and communicate arbitrary messages, respectively.

The remaining four events can be separated into two categories:

- Changes to the Provider's ability to make RPC requests
  - `connect`
  - `disconnect`
- Common Client and/or Wallet state changes that any non-trivial application must handle
  - `chainChanged`
  - `accountsChanged`

These events are included due to the widespread production usage of related patterns, at the time of writing.
## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC-2119](https://www.ietf.org/rfc/rfc2119.txt).

> Comments like this are non-normative.

### Definitions

_This section is non-normative._

- Provider
  - A JavaScript object made available to a consumer, that provides access to Ethereum by means of a Client.
- Client
  - An endpoint that receives Remote Procedure Call (RPC) requests from the Provider, and returns their results.
- Wallet
  - An end-user application that manages private keys, performs signing operations, and acts as a middleware between the Provider and the Client.
- Remote Procedure Call (RPC)
  - A Remote Procedure Call (RPC), is any request submitted to a Provider for some procedure that is to be processed by a Provider, its Wallet, or its Client.

### Connectivity

The Provider is said to be "connected" when it can service RPC requests to at least one chain.

The Provider is said to be "disconnected" when it cannot service RPC requests to any chain at all.

> To service an RPC request, the Provider must successfully submit the request to the remote location, and receive a response.
> In other words, if the Provider is unable to communicate with its Client, for example due to network issues, the Provider is disconnected.

### API

> The Provider API is specified using TypeScript.
> The authors encourage implementers to declare their own types and interfaces, using the ones in this section as a basis.
>
> For consumer-facing API documentation, see [Appendix I](#appendix-i-consumer-facing-api-documentation)

The Provider **MUST** implement and expose the API defined in this section.
All API entities **MUST** adhere to the types and interfaces defined in this section.

#### request

> The `request` method is intended as a transport- and protocol-agnostic wrapper function for Remote Procedure Calls (RPCs).

```typescript
interface RequestArguments {
  readonly method: string;
  readonly params?: readonly unknown[] | object;
}

Provider.request(args: RequestArguments): Promise<unknown>;
```

The Provider **MUST** identify the requested RPC method by the value of `RequestArguments.method`.

If the requested RPC method takes any parameters, the Provider **MUST** accept them as the value of `RequestArguments.params`.

RPC requests **MUST** be handled such that the returned Promise either resolves with a value per the requested RPC method's specification, or rejects with an error.

If resolved, the Promise **MUST** resolve with a result per the RPC method's specification. The Promise **MUST NOT** resolve with any RPC protocol-specific response objects, unless the RPC method's return type is so defined.

If the returned Promise rejects, it **MUST** reject with a `ProviderRpcError` as specified in the [RPC Errors](#rpc-errors) section below.

The returned Promise **MUST** reject if any of the following conditions are met:

- An error is returned for the RPC request.
  - If the returned error is compatible with the `ProviderRpcError` interface, the Promise **MAY** reject with that error directly.
- The Provider encounters an error or fails to process the request for any reason.

> If the Provider implements any kind of authorization logic, the authors recommend rejecting with a `4100` error in case of authorization failures.

The returned Promise **SHOULD** reject if any of the following conditions are met:

- The Provider is disconnected.
  - If rejecting for this reason, the Promise rejection error `code` **MUST** be `4900`.
- The RPC request is directed at a specific chain, and the Provider is not connected to that chain, but is connected to at least one other chain.
  - If rejecting for this reason, the Promise rejection error `code` **MUST** be `4901`.

See the section [Connectivity](#connectivity) for the definitions of "connected" and "disconnected".

### Supported RPC Methods

A "supported RPC method" is any RPC method that may be called via the Provider.

All supported RPC methods **MUST** be identified by unique strings.

Providers **MAY** support whatever RPC methods required to fulfill their purpose, standardized or otherwise.

If an RPC method defined in a finalized EIP is not supported, it **SHOULD** be rejected with a `4200` error per the [Provider Errors](#provider-errors) section below, or an appropriate error per the RPC method's specification.

#### RPC Errors

```typescript
interface ProviderRpcError extends Error {
  code: number;
  data?: unknown;
}
```

- `message`
  - **MUST** be a human-readable string
  - **SHOULD** adhere to the specifications in the [Error Standards](#error-standards) section below
- `code`
  - **MUST** be an integer number
  - **SHOULD** adhere to the specifications in the [Error Standards](#error-standards) section below
- `data`
  - **SHOULD** contain any other useful information about the error

##### Error Standards

`ProviderRpcError` codes and messages **SHOULD** follow these conventions, in order of priority:

1. The errors in the [Provider Errors](#provider-errors) section below

2. Any errors mandated by the erroring RPC method's specification

3. The [`CloseEvent` status codes](https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent#Status_codes)

#### Provider Errors

| Status code | Name                  | Description                                                              |
| ----------- | --------------------- | ------------------------------------------------------------------------ |
| 4001        | User Rejected Request | The user rejected the request.                                           |
| 4100        | Unauthorized          | The requested method and/or account has not been authorized by the user. |
| 4200        | Unsupported Method    | The Provider does not support the requested method.                      |
| 4900        | Disconnected          | The Provider is disconnected from all chains.                            |
| 4901        | Chain Disconnected    | The Provider is not connected to the requested chain.                    |

> `4900` is intended to indicate that the Provider is disconnected from all chains, while `4901` is intended to indicate that the Provider is disconnected from a specific chain only.
> In other words, `4901` implies that the Provider is connected to other chains, just not the requested one.

### Events

The Provider **MUST** implement the following event handling methods:

- `on`
- `removeListener`

These methods **MUST** be implemented per the Node.js [`EventEmitter` API](https://nodejs.org/api/events.html).

> To satisfy these requirements, Provider implementers should consider simply extending the Node.js `EventEmitter` class and bundling it for the target environment.

#### message

> The `message` event is intended for arbitrary notifications not covered by other events.

When emitted, the `message` event **MUST** be emitted with an object argument of the following form:

```typescript
interface ProviderMessage {
  readonly type: string;
  readonly data: unknown;
}
```

##### Subscriptions

If the Provider supports Ethereum RPC subscriptions, e.g. [`eth_subscribe`](https://geth.ethereum.org/docs/rpc/pubsub), the Provider **MUST** emit the `message` event when it receives a subscription notification.

If the Provider receives a subscription message from e.g. an `eth_subscribe` subscription, the Provider **MUST** emit a `message` event with a `ProviderMessage` object of the following form:

```typescript
interface EthSubscription extends ProviderMessage {
  readonly type: 'eth_subscription';
  readonly data: {
    readonly subscription: string;
    readonly result: unknown;
  };
}
```

#### connect

See the section [Connectivity](#connectivity) for the definition of "connected".

If the Provider becomes connected, the Provider **MUST** emit the event named `connect`.

This includes when:

- The Provider first connects to a chain after initialization.
- The Provider connects to a chain after the `disconnect` event was emitted.

This event **MUST** be emitted with an object of the following form:

```typescript
interface ProviderConnectInfo {
  readonly chainId: string;
}
```

`chainId` **MUST** specify the integer ID of the connected chain as a hexadecimal string, per the [`eth_chainId`](./eip-695.md) Ethereum RPC method.

#### disconnect

See the section [Connectivity](#connectivity) for the definition of "disconnected".

If the Provider becomes disconnected from all chains, the Provider **MUST** emit the event named `disconnect` with value `error: ProviderRpcError`, per the interfaced defined in the [RPC Errors](#rpc-errors) section. The value of the error's `code` property **MUST** follow the [status codes for `CloseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent#Status_codes).

#### chainChanged

If the chain the Provider is connected to changes, the Provider **MUST** emit the event named `chainChanged` with value `chainId: string`, specifying the integer ID of the new chain as a hexadecimal string, per the [`eth_chainId`](./eip-695.md) Ethereum RPC method.

#### accountsChanged

If the accounts available to the Provider change, the Provider **MUST** emit the event named `accountsChanged` with value `accounts: string[]`, containing the account addresses per the `eth_accounts` Ethereum RPC method.

The "accounts available to the Provider" change when the return value of `eth_accounts` changes.

## EIP 1102

### Simple summary

This proposal describes a communication protocol between dapps and Conflux-enabled DOM environments that allows the Conflux-enabled DOM environment to choose what information to supply the dapp with and when.

provider should support follow rpc methods，relay on EIP-1102, and **the difference of Conflux is replace window.ethereum to window.conflux and replace rpc prefix eth_ to cfx_**

- cfx_requestAccounts

## CIP-23 (Suggest compatible with EIP-712)
### Simple Summary
Signing data is a solved problem if all we care about are bytestrings. Unfortunately in the real world we care about complex meaningful messages. Hashing structured data is non-trivial and errors result in loss of the security properties of the system. Conflux should provide a method for signing typed data like Ethereum EIP-712.


provider should support follow rpc methods，relay on [CIP-23](https://github.com/Conflux-Chain/CIPs/blob/master/CIPs/cip-23.md)，and suggest to be compatible with [EIP-712](https://eips.ethereum.org/EIPS/eip-712)

- cfx_sign
- cfx_signTypedData
- cfx_signTypedData_v1
- cfx_signTypedData_v3
- cfx_signTypedData_v4

## EIP-2255 (Optional)

### Simple Summary
A proposed standard interface for restricting and permitting access to security-sensitive methods within a restricted web3 context like a website or “dapp”.

provider should support follow rpc methods，relay on [EIP-2255](https://eips.ethereum.org/EIPS/eip-2255)
- wallet_requestPermissions
- wallet_getPermissions

*it is optional*

## Others
Due to some dapp is released and interact with protal, so suggest wallets to compatible with portal provider API.
### Portal provider supported methods and properties

#### Methods relay on EIP-1193 old version
Portal currently relay on old EIP-1193 version for rpc request, suggest to update follow new EIP-1193
- send(options, callback) (To Be Replaced)
- sendAsync(options, callback)
- on(eventName, callback)

#### Shortcut for EIP-1102
portal currently relay on old EIP-1102 version for account requests, and it is a quick access method for rpc request_accounts
- enable()

#### Shortcut for send
- call(method, ...params)

#### Deprecated methods
- autoRefreshOnNetworkChange (To Be Removed)

#### Propertis
portal supported properties, some of them are deprecated and will remove in later version.

- isConfluxPortal
- networkVersion (Deprecated)
- selectedAddress (Deprecated)
- requestId
- isConnected

## Refrences
- [EIP-1193](https://eips.ethereum.org/EIPS/eip-1193)
- [EIP-1102](https://eips.ethereum.org/EIPS/eip-1102)
- [EIP-712](https://eips.ethereum.org/EIPS/eip-712)
- [CIP-23](https://github.com/Conflux-Chain/CIPs/blob/master/CIPs/cip-23.md)
- [EIP-2255](https://eips.ethereum.org/EIPS/eip-2255)
- [EIP-695](https://eips.ethereum.org/EIPS/eip-2255)