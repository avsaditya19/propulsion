# Propulsion [![Build Status](https://dev.azure.com/jet-opensource/opensource/_apis/build/status/jet.Propulsion)](https://dev.azure.com/jet-opensource/opensource/_build/latest?definitionId=16) [![release](https://img.shields.io/github/release/jet/propulsion.svg)](https://github.com/jet/propulsion/releases) [![NuGet](https://img.shields.io/nuget/v/Propulsion.svg?logo=nuget)](https://www.nuget.org/packages/Propulsion/) [![license](https://img.shields.io/github/license/jet/propulsion.svg)](LICENSE) ![code size](https://img.shields.io/github/languages/code-size/jet/propulsion.svg) [<img src="https://img.shields.io/badge/slack-DDD--CQRS--ES%20%23equinox-yellow.svg?logo=slack">](https://j.mp/ddd-es-cqrs) [![docs status](https://img.shields.io/badge/DOCUMENTATION-WIP-important.svg?style=popout)](DOCUMENTATION.md) 

While the bulk of this code is in production across various Walmart systems, the [documentation](DOCUMENTATION.md) is very much a work in progress (ideally there'd be a nice [summary of various projection patterns](https://github.com/jet/propulsion/issues/21), but also much [broader information discussing the tradeoffs implied in an event-centric system as a whole](https://github.com/ylorph/The-Inevitable-Event-Centric-Book/issues)

If you're looking for a good discussion forum on these kinds of topics, look no further than the [DDD-CQRS-ES Slack](https://github.com/ddd-cqrs-es/slack-community)'s [#equinox channel](https://ddd-cqrs-es.slack.com/messages/CF5J67H6Z) ([invite link](https://j.mp/ddd-es-cqrs)).

## Components

The components within this repository are delivered as a multi-targeted Nuget package targeting `net461` (F# 3.1+) and `netstandard2.0` (F# 4.5+) profiles

- `Propulsion` [![NuGet](https://img.shields.io/nuget/v/Propulsion.svg)](https://www.nuget.org/packages/Propulsion/) Implements core functionality in a channel-independent fashion including `ParallelProjector`, `StreamsProjector`. [Depends](https://www.fuget.org/packages/Propulsion) on `MathNet.Numerics`, `Serilog`
- `Propulsion.Cosmos` [![NuGet](https://img.shields.io/nuget/v/Propulsion.Cosmos.svg)](https://www.nuget.org/packages/Propulsion.Cosmos/) Provides bindings to Azure CosmosDb a) writing to `Equinox.Cosmos` :- `CosmosSink` b) reading from CosmosDb's changefeed by wrapping the [`dotnet-changefeedprocessor` library](https://github.com/Azure/azure-documentdb-changefeedprocessor-dotnet) :- `CosmosSource`. [Depends](https://www.fuget.org/packages/Propulsion.Cosmos) on `Equinox.Cosmos`, `Microsoft.Azure.DocumentDB.ChangeFeedProcessor`, `Serilog`
- `Propulsion.EventStore` [![NuGet](https://img.shields.io/nuget/v/Propulsion.EventStore.svg)](https://www.nuget.org/packages/Propulsion.EventStore/). Provides bindings to [EventStore](https://www.eventstore.org), writing via `Propulsion.EventStore.EventStoreSink` [Depends](https://www.fuget.org/packages/Propulsion.EventStore) on `Equinox.EventStore`, `Serilog`
- `Propulsion.Kafka` [![NuGet](https://img.shields.io/nuget/v/Propulsion.Kafka.svg)](https://www.nuget.org/packages/Propulsion.Kafka/) Provides bindings for producing and consuming both streamwise and in parallel. Includes a standard codec for use with streamwise projection and consumption, `Propulsion.Kafka.Codec.NewtonsoftJson.RenderedSpan`. Implements a `KafkaMonitor` that can log status information based on [Burrow](https://github.com/linkedin/Burrow). [Depends](https://www.fuget.org/packages/Propulsion.Kafka) on `FsKafka` v ` = 1.3.0`, `Serilog`
- `Propulsion.Kafka0` [![NuGet](https://img.shields.io/nuget/v/Propulsion.Kafka0.svg)](https://www.nuget.org/packages/Propulsion.Kafka0/). Same functionality/purpose as `Propulsion.Kafka` but targets older `Confluent.Kafka`/`librdkafka` version for for interoperability with systems that have a hard dependency on that. [Depends](https://www.fuget.org/packages/Propulsion.Kafka0) on `Confluent.Kafka [0.11.3]`, `librdkafka.redist [0.11.4]`, `Serilog`

The ubiquitous `Serilog` dependency is solely on the core module, not any sinks, i.e. you configure to emit to `NLog` etc.

### `dotnet tool` provisioning / projections test tool

- `Propulsion.Tool` [![Tool NuGet](https://img.shields.io/nuget/v/Propulsion.Tool.svg)](https://www.nuget.org/packages/Propulsion.Tool/): Tool used to initialize a Change Feed Processor `aux` container for `Propulsion.Cosmos` and demonstrate basic projection, including to Kafka. (Install via: `dotnet tool install Propulsion.Tool -g`)

## Related repos

- See [the Equinox QuickStart](https://github.com/jet/equinox#quickstart) for examples of using this library to project to Kafka from `Equinox.Cosmos` and/or `Equinox.EventStore`.

- See [the `dotnet new` templates repo](https://github.com/jet/dotnet-templates) for examples using the packages herein:

    - [Propulsion-specific templates](https://github.com/jet/dotnet-templates#propulsion-related):
      - `proProjector` template for `CosmosSource`+`StreamsProjector` logic consuming from a CosmosDb `ChangeFeedProcessor`.
      - `proProjector` template (in `--kafka` mode) for producer logic using `StreamsProducerSink` or `ParallelProducerSink`.
      - `proConsumer` template for example consumer logic using `ParallelConsumer` and `StreamsConsumer` etc.

    - [Propulsion+Equinox templates](https://github.com/jet/dotnet-templates#producerreactor-templates-combining-usage-of-equinox-and-propulsion):
      - `proReactor` template, which includes multiple sources and multiple processing modes
      - `summaryConsumer` template, consumes from the output of a `proReactor --kafka`, saving them in an `Equinox.Cosmos` store
      - `trackingConsumer`template, which consumes from Kafka, feeding into example Ingester logic
      - `proSync` template is a fully fledged store <-> store synchronization tool syncing from a `CosmosSource` or `EventStoreSource` to a `CosmosSink` or `EventStoreSink`

- See [the `FsKafka` repo](https://github.com/jet/FsKafka) for `BatchedProducer` and `BatchedConsumer` implementations (together with the `KafkaConsumerConfig` and `KafkaProducerConfig` used in the Parallel and Streams wrappers in `Propulsion.Kafka`)

# Overview

## The Equinox Perspective

Propulsion and Equinox have a [Yin and yang](https://en.wikipedia.org/wiki/Yin_and_yang) relationship; the use cases for both naturally interlock and overlap.

It can be relevant to peruse [the Equinox Documentation's Overview Diagrams](https://github.com/jet/equinox/blob/master/DOCUMENTATION.md#overview) for the perspective from the other side (TL;DR its largely the same topology, with elements that are de-emphasized here central over there, and vice versa)

## [C4](https://c4model.com) Context diagram

While Equinox focuses on the **Consistent Processing** element of building an event-sourced system, offering tailored components that interact with a specific **Consistent Event Store**, Propulsion supports the building of complementary facilities as part of an overall Application:

- **Ingesters**: reading stuff from outside the Bounded Context of the System. This kind of service covers aspects such as feed reference data into **Read Models**, ingesting changes into a consistent model via **Consistent Processing**. _These services are not acting in reaction to events emanating from the **Consistent Event Store**_
- **Publishers**: reacting to events as they are arrive from the **Consistent Event Store** by either rendering and producing it out to a feed. _While these services may in some cases do synchronous queries via **Consistent Processing**, it's never transacting or driving follow-on work._
- **Reactors**: driven by either external feeds, or events observed in the **Consistent Event Store**. _These services handle anything beyond the remit of **Ingesters** or **Publishers**, and will often drive follow-on processing via Process Managers and/or transacting via **Consistent Processing**_

The overall territory is laid out here in this [C4](https://c4model.com) System Context Diagram:

![Propulsion c4model.com Context Diagram](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.github.com/jet/propulsion/diag/diagrams/context.puml&fmt=svg)

## [C4](https://c4model.com) Container diagram

The relevant pieces of the above break down as follows, when we emphasize the [Containers](https://c4model.com) aspects relevant to Propulsion:

![Propulsion c4model.com Container Diagram](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.github.com/jet/propulsion/diag/diagrams/container.puml&fmt=svg)

**[See Overview section in `DOCUMENTATION`.md for further drill down](https://github.com/jet/propulsion/blob/diag/DOCUMENTATION.md#overview)**

## QuickStart

### 1. Use `propulsion` tool to run a CosmosDb ChangeFeedProcessor

```powershell
dotnet tool uninstall Propulsion.Tool -g
dotnet tool install Propulsion.Tool -g

propulsion init -ru 400 cosmos # generates a -aux container for the ChangeFeedProcessor to maintain consumer group progress within
# -V for verbose ChangeFeedProcessor logging
# `-g projector1` represents the consumer group - >=1 are allowed, allowing multiple independent projections to run concurrently
# stats specifies one only wants stats regarding items (other options include `kafka` to project to Kafka)
# cosmos specifies source overrides (using defaults in step 1 in this instance)
propulsion -V project -g projector1 stats cosmos
```

### 2. Use `propulsion` tool to Run a CosmosDb ChangeFeedProcessor, emitting to a Kafka topic

```powershell
$env:PROPULSION_KAFKA_BROKER="instance.kafka.mysite.com:9092" # or use -b

# `-V` for verbose logging
# `-g projector3` represents the consumer group; >=1 are allowed, allowing multiple independent projections to run concurrently
# `-l 5` to report ChangeFeed lags every 5 minutes
# `kafka` specifies one wants to emit to Kafka
# `temp-topic` is the topic to emit to
# `cosmos` specifies source overrides (using defaults in step 1 in this instance)
propulsion -V project -g projector3 -l 5 kafka temp-topic cosmos
```

## CONTRIBUTING

See [CONTRIBUTING.md](CONTRIBUTING.md)

## TEMPLATES

The best place to start, sample-wise is with the [QuickStart](#quickstart), which walks you through sample code, tuned for approachability, from `dotnet new` templates stored [in a dedicated repo](https://github.com/jet/dotnet-templates).

## BUILDING

Please note the [QuickStart](#quickstart) is probably the best way to gain an overview, and the templates are the best way to see how to consume it; these instructions are intended mainly for people looking to make changes. 

NB The `Propulsion.Kafka.Integration` tests are reliant on a `TEST_KAFKA_BROKER` environment variable pointing to a Broker that has been configured to auto-create ephemeral Kafka Topics as required by the tests (each test run blindly writes to a guid-named topic and trusts the broker will accept the write without any initialization step)

### build, including tests on net461 and netcoreapp3.1

```powershell
dotnet build build.proj -v n
```

## FAQ

<a name="why-not-cfp-all-the-things"></a>
### why do you employ Kafka as an additional layer, when downstream processes could simply subscribe directly and individually to the relevant Cosmos db change feed(s)? Is it to accommodate other messages besides those emitted from events and snapshot updates? :pray: [@Roland Andrag](https://github.com/randrag)

Well, Kafka is definitely not a critical component or a panacea.

You're correct that the bulk of things that can be achieved using Kafka can be accomplished via usage of the changefeed. One thing to point out is that in the context of enterprise systems, having a well maintained Kafka cluster does have less incremental cost that it might do if you're building a smaller system from nothing.

Some of the negatives of consuming from the CF direct:

- each CFP reader imposes RU charges (its a set of continuous queries against each and every physical range of which the cosmos store is composed)
- you can't apply a server-side filter, so you pay for everything you see
- you're very prone to falling into coupling issues
- (as you alluded to), if there's some logic or work involved in the production of events you'd emit to Kafka, each consumer would need to duplicate that

While many of these concerns can be alleviated to varying degrees by splitting the storage up into multiple containers in order that each consumer will intrinsically be interested in a large proportion of the data it will observe (potentially using database level RU allocations), the write amplification effects of having multiple consumers will always be more significant when reading directly than when using Kafka,  the design of which is well suited to running lots of concurrent readers.

Splitting event categories into containers solely to optimize these effects can also make the management of the transactional workload more complex; the ideal for any given container is to balance the concerns of:

- ensuring that datasets for which you want to ringfence availability / RU allocations don't share with containers/databases for which running hot (potentially significant levels of rate limiting but overall high throughput in aggregate as a result of using a high percentage of the allocated capacity)
- avoiding prematurely splitting data prior to it being required by the constraints of CosmosDB (i.e. you want to let splitting primarily be driven by reaching the [10GB] physical partition range)
- not having logical partition hotspots that lead to a small number of physical partitions having significantly above average RU consumption
- having relatively consistent document sizes
- economies of scale - if each container (or database if you provision at that level) needs to individually managed (with a degree of headroom to ensure availability for load spikes etc), you'll tend to require higher aggregate RU assignment for a given overall workload based on a topology that has more containers

### What's the deal with the history of this repo?

This repo is derived from [`FsKafka`](https://github.com/jet/FsKafka); the history has been edited to focus only on edits to the `Propulsion` libraries.

### Your question here

- Please feel free to log question-issues; they'll get answered here

# FURTHER READING

See [`DOCUMENTATION.md`](DOCUMENTATION.md) and [Equinox's `DOCUMENTATION.md`](https://github.com/jet/equinox/blob/master/DOCUMENTATION.md)