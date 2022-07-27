# Storage

The protocol is designed to be storage system agnostic. For the quick-start and proof-of-concept implementations, the data storge system used is [swarm](https://ethswarm.org). Swarm meets the design requirements of *decentralised* and *data ownership*. 

## Generic Payloads

### Service Provider Info / Items / Terms

The initial design is for service provider data to be stored monolithically, excluding images. The payloads are generic in nature, with payloads designed to be extended by industry-specific implementations.

Each payload is placed in a `SignedPayloadWrapper`, with an authorised API signer having signed a hash of the payload which is subsequently placed in `SignedPayloadWrapper.signature`. This provides 

```protobuf
message ServiceItemData {
  // primitive item.id
  bytes item = 1;
  // industry-specific payload describing item
  bytes payload = 2;
}

message ServiceTermData {
  // primitive term.id
  bytes term = 1;
  // industry-specific payload describing term
  bytes payload = 2;
  // smart contract address that implements ITerm interface
  string implementation = 3;
}

message ServiceProviderData {
  // primitive serviceProvider.id
  bytes serviceProvider = 1;
  // services (items) provided by this service provider
  repeated ServiceItemData items = 2;
  // terms that may be applicable to services provided
  repeated ServiceTermData terms = 3;
  // industry-specific payload describing service provider
  bytes payload = 4;
  // signed hash by ServiceProvider `api` signer
  bytes signature = 5;
}
```

## movi Payloads

### Driver Profile

```protobuf
// ServiceProviderData.payload
message Driver {
  // name of driver
  // vehicle description
  // phone number
  // profile picture
  // connectivity in vehicle
}
```

### Rider Profile

```protobuf
// BuyerData.payload
message Rider {
  // name of rider
  // phone number
  // profile picture
}
```