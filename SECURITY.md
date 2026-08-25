# Security Model

HCL is a toolkit for building structured, expressive configuration languages. The
evaluator is powerful by design: it resolves variables, calls functions, and processes
complex expressions on behalf of the calling application. The security properties of an
HCL integration are determined primarily by how the library is used, not by the library
itself.

This document describes the integration security model for Go developers embedding HCL
in their own applications. It is not exhaustive. Integrators are responsible for assessing
their own risk posture and applying controls appropriate to their deployment context.

> **HCL configuration from untrusted sources must not be evaluated unless appropriate
> security controls are in place.**

## What HCL Guarantees

The HCL library itself makes the following guarantees in isolation:

- The expression evaluator does not make network calls, read environment variables, or
  execute system commands.
- HCL reads files from disk during the **parsing** step when loading configuration
  source, but the evaluator itself performs no I/O.
- HCL does not dynamically load code or plugins at runtime.
- The type system is strict. Evaluated values are confined to the types explicitly
  exposed by the integrator; the evaluator cannot reach outside that boundary on its
  own.

These guarantees apply to the library in isolation. They say nothing about what
integrator-supplied functions and variables are capable of doing.

## Integrator Responsibilities

All capability available to an HCL expression is supplied by the integrator through the
evaluation context: the functions it can call, the variables it can read, and the data
it can access. HCL imposes no restrictions on what those functions do once registered.
The integrator is the primary security control for any HCL-based system.

### Principle of least privilege

Expose only the functions and variables needed for the configuration's intended purpose.
Do not expose general-purpose utilities that could be chained into unintended behavior.
The smaller the surface area of the evaluation context, the smaller the impact of a
malicious or malformed configuration.

### Input validation

Any value sourced from an untrusted origin that is placed into the evaluation context
must be validated and sanitized before use. The evaluator operates on the values it is
given without additional checks; ensuring those values are safe is the integrator's
responsibility.

### Privileged configuration and optional extensions

HCL includes optional extensions that significantly expand what a configuration can
express, including user-defined functions and dynamic block generation. These extensions
should only be enabled for **privileged configuration**: configuration that is fully
controlled by the integrator or a trusted operator. They should not be enabled for
end-user-supplied input unless strong, independent controls are in place that bound what
a user can do.
