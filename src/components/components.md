# Components

Each of the SEP components rely on other protocols and systems, yet it is designed to be protocol and system agnostic. That is by design, yet in practice there aren't many protocols and systems to pick that fulfil the design requirements, as most of them have only come into existance in the past few years. 

The current three components and the systems used for each are:
- [Messaging](./messaging.md) is using [waku](https://waku.org)
- [Storage](./storage.md) is using [swarm](htpps://ethswarm.org)
- [Execution](./execution.md) is using [gnosis chain](https://gnosis.io)

>It is important to understand that the only on-chain component is the Execution, the rest is off-chain.



