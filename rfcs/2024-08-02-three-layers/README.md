# Three Software Layers

This document proposes three conceptual layers in the software system that constitutes a Common Runtime. Each of these layers constitutes a (lower-case) runtime context and distinctive relationship to users of a Common Runtime.

### Goals

- Define three relevant software layers within the Common Runtime
- Disambiguate some users and describe their relationships to the different layers

#### User stories

_As a Common employee, when I author stories about Common Runtime users, I want to reference one or more specific layers of the programmable software stack in a Common Runtime so that my audience has a shared mental model of the workflow I'm describing and which conceptual users may be relevant to consider_

### Non-goals

- Define all possible layers in the Common Runtime stack
- Define the full scope of user archetypes with adjacency to Common software

## Background

This proposal builds on ideas proposed in prior RFCs:

- [On-demand Isolated Modules]
- [Runtime Library Registration]
- [Three Software Authors]

## The Module Layer

We have provisionally adopted the term "Module" to refer to a unit of code that is able to run securely within our [Runtime]'s sandbox: the Module Layer.

As of today, all code running at the Module Layer runs in a Web Assembly sandbox. The sandbox ensures that:

- I/O happens over a tightly-constrained, Common Tools-provided API
- A Module exports a prescribed target API that gives the Runtime direct control over the Module's lifecycle

Theoretically speaking, Modules may be authored in any language that can express a [Wasm Component]. In practice, all Modules of substance have been authored in JavaScript.

A Module's expected inputs and intended outputs are known before it is instantiated and invoked. The only guarantee that a module has about its inputs and outputs is that the configured inputs may be read and the configured outputs may be written at runtime. There is no guarantee that the input values are meaningful, nor that output values will be persisted.

### Users of The Module Layer

Some of the users of the Module Layer are described in [Three Software Authors]:

- [The Standard Library Author]
- [The Framework Author]
- [The Module Author]

## The Recipe Layer

If a Module is a unit of code, a Recipe may be thought of as an assemblage of many Modules into a composite program. Recipes do not have a formal specification (yet). Nevertheless, we tend to agree that recipes are graphs whose nodes include Modules, and whose edges describe the flow of data across those nodes.

The Recipe Layer is therefor responsible for:

- Managing the lifetime of all Recipes and their constituent Modules that are scheduled to the local Common Runtime
- Participating in the scheduling regime that spans all Common Runtimes in the user's trust domain
- Co-ordinating the ingress and egress of user data across a Recipe graph

At the top-level of its description, a Recipe is modeled as a special case of a Module. It follows that a Recipe may compose other Recipes as nodes in their interior graph.

The basic serialized form of a Recipe is hierarchical, structured data. More specifically: all candidate data structures that we have called Recipe are representable as JSON. We may statically analyze all of the inputs and outputs of all Modules that a Recipe composes. And, although this is not true today, Recipes are expected to have well-defined Runtime semantics.

### Users of The Recipe Layer

#### The Recipe Author

The Recipe Author assembles one or more Modules into a Recipe. The method of assembly may vary, but the final product that The Recipe Author delivers to The Recipe Layer is a directly-serializable, hierarchical data structure that constitutes a Recipe.

The Recipe Author may be human, but also may be an agent of a human such as a trusted LLM or other trusted, automated process.

#### The Recipe Runtime Author

The Recipe Runtime Author is a creator and/or maintainer of the Common Runtime, which is the software system that interprets Modules (and transitively: Recipes), enforces privacy policies as needed and circumstantially schedules the Modules to be instantiated by The Module Layer.

The work product of the Recipe Runtime Author is an API that accepts a Recipe, manages its lifecycle and facilitates access to capabilities in the Content Layer.

## The Content Layer

The Content Layer manages all capabilities that may allow a Module (and transitively: a Recipe) to manipulate the on-screen experience of the [Content Audience].

Modules (and transitively: Recipes) may be thought of as running in a sandbox, and have no direct means to influence the UX of the Content Audience. However, We intend to develop the capability for Modules to include static descriptions of UI hierarchies among their outputs, and to circumstantially render corresponding UI for the Content Audience. We have even speculated about Modules that have advanced capabilities such as access to a local device's GPU.

Whenever a Module produces an output that is to be displayed to the Content Audience, it is rendered as content by the Content Layer. Whenever a Module requires a special, device-specific capability, that capability is provided by the Content Layer (or else it is considered unavailable). The Content Layer is responsible for making the lifecycle of a given Module legible to the Content Audience (even in cases where the Module does not access capabilities that would cause other content to be rendered).

To the extent that there is a top-level interface (chrome) for the Common Runtime, it interleaves the Content Layer's rendered content as appropriate to reflect the state of all active Modules within the Common Runtime (including those that may be scheduled to other devices in the same trust domain).

### Users of The Content Layer

#### The Content Audience

The Content Audience is who we may think of as the end-users of the Common Runtime (although this is an imperfect analogy and is not universally applicable). They are the users who view and interact with the rendered content produced by the [Content Layer]. When a Module is said to access user data, the user in question is typically the Content Audience.

The Content Audience does not directly operate on the [Recipe Layer] or the [Module Layer]. Instead, they interact with Common Runtime chrome or rendered content at the Content Layer. Content Audience interactions with the chrome or Content Layer imply various actions such as the assembly or instantiation of Recipes and other Modules. The Content Audience may occassionally write code directly, but when they do it is part of an auxilliary UX flow.

#### The Content Capability Author

The Content Capability Author is a human who is responsibile for implementing and/or maintaining various capabilities that are to be supported by the Common Runtime. Although the set of capabilities supported today are limited to those that enable UI manipulation, they may also include such things as peripheral access (e.g., GPU, camera), location information and network APIs (to name a few speculative examples).

#### The Chrome Author

The Chrome Author is a human who is responsible for producing a UX that enables the [Content Audience] to create, assemble, instantiate and manage the lifecycle of Modules (and transitively: Recipes). The Chrome Author is firstly responsible for Common Runtime chrome, but is also responsible for interleaving rendered content from The Content Layer as appropriate.

[Module Layer]: #the-module-layer
[Recipe Layer]: #the-recipe-layer
[Content Layer]: #the-content-layer
[Content Audience]: #the-content-audience
[Wasm Component]: https://component-model.bytecodealliance.org/
[Runtime]: ../2024-05-23-runtime-library-registration/README.md#runtime
[On-demand Isolated Modules]: https://github.com/commontoolsinc/labs/blob/main/rfcs/2024-05-19-on-demand-isolated-modules.md
[Runtime Library Registration]: https://github.com/commontoolsinc/labs/blob/main/rfcs/2024-05-23-runtime-library-registration.md
[Three Software Authors]: https://github.com/commontoolsinc/labs/blob/main/rfcs/2024-05-31-three-authors.md
[The Standard Library Author]: https://github.com/commontoolsinc/labs/blob/main/rfcs/2024-05-31-three-authors.md#the-standard-library-author
[The Framework Author]: https://github.com/commontoolsinc/labs/blob/main/rfcs/2024-05-31-three-authors.md#the-framework-author
[The Module Author]: https://github.com/commontoolsinc/labs/blob/main/rfcs/2024-05-31-three-authors.md#the-module-author