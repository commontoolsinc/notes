# Information Flow Control

*Exploring our IFC implementation and requirements. Meant to succinctly describe the constraints an IFC system should uphold.*

In the system, a module is an execution context that may contain any number of input and output ports for data to be read or written to. A program may contain many modules, creating a directed execution graph with inputs mapped to outputs. To enable the safe processing of classified data by untrusted code in modules, information flow control technique are employed by the system.

## Overview

All data in the system is tagged with both a **Confidentiality** label and an **Integrity** label. Both label sets are finite, representing linear degrees of confidentiality and integrity. An example of a minimal set of labels, in decreasing degrees, shown:

```
C = [
  Classified,
  Unclassified,
]
I = [
  HighIntegrity,
  LowIntegrity,
]
```

A **Policy** codifies a user's permissions, specifying what modules can access labeled data. In a **Policy**, each label degree is mapped to a **ModuleContext** representing requirements that must be met in order for data with that label to be accessible to that module.

For example, a **Policy** that enforces `Classified` data to only be accessible to a sufficiently trusted **ModuleContext** may look like:

```
Hi = HighTrustContext
Lo = LowTrustContext
Cp = (
  (C::Classified, Hi),
  (C::Unclassified, Lo),
)
Ip = (
  (I::HighIntegrity, Lo),
  (I::LowIntegrity, Lo),
)
P(Cp, Ip)
```

Each module at validation time has a **ModuleContext**, containing details about the module, speculatively containing:

* **Environment**: Where the module will be evaluated; locally, on a remote trusted/untrusted server, in a confidential compute environment, etc.
* **Origin**: Details about the origin of the module, from a trusted/untrusted source or author.
* **Capabilities**: A list of features required to execute this module.

When validating a module's **ModuleContext** against a **Policy**'s **ModuleContext** requirements, each property's validation semantics are independent. For example, an **Environment** requirement may be satisfied with any **Environment** "more secure" than the requirement, and **Capabilities** requirements are permissions, where the requested **Capabilities** must all be satisfied by the requirements.

## Enforcement

During validation, a graph is created of all modules and their contexts, and the edges between their input and output ports. Safe propagation of data through the system is enforced through the following principles.

### Sandboxing

To support the flow of classified data into potentially untrusted modules, each module is executed within a sandboxed environment. The module's input and output ports are the only means for data to enter or exit the environment.

The only exception to this are some **Capabilities** (like networking) which may introduce means for data egress, which must be explicitly allowed via **Policy**.

### Connection Validity

Each input to a module must be validated to ensure that the module is allowed to receive the data. The **ModuleContext** of the module must be satisfied by the **Policy**'s **ModuleContext** requirements for both the **Confidentiality** and **Integrity** labels of the data.

```
inputs = [
    ("supersecret", Classified, HighIntegrity),
    (1, Unclassified, HighIntegrity),
]

for input in inputs:
    (data, confidentiality, integrity) = input
    for label in [confidentiality, integrity]:
        reqs = policy.get_reqs(label)
        if !validate(reqs, module_ctx):
    	    throw
```

### Label Normalization

By default, the labels of data egressing from module outputs are dependent on input labels. When a module receives both `Unclassified` and `Classified` data, all outputs must also be `Classified`, as any data can be derived from the `Classified` data. Similarly, modules receiving both `HighIntegrity` and `LowIntegrity` data must be considered `LowIntegrity`.

* The **Confidentiality** of an output is the *most* confidential of all input data.
* The **Integrity** of an output is the *least* integral of all input data.

```
max_conf = bottom(Confidentiality)
min_int = top(Integrity)
for (conf, int) in inputs:
    max_conf = max(max_conf, conf)
    min_int  = min(min_int, int)
```

## Questions

> Wouldn't all outputs be lowered to the lowest integrity, always, except if explicitly requested in a ~trusted module?

> For recursive graphs, where a cycle between modules is introduced, can we statically verify that there's no escalation of access? Can imagine a graph walker that would produce different requirements per graph resolution

> This doesn't address multi-user data. How would something like that work?

> What dimensions cannot be represented by this approach? 
