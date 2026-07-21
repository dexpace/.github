<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/dexpace/morphic/main/docs/assets/dexpace-wordmark-dark.svg">
    <img alt="dexpace" src="https://raw.githubusercontent.com/dexpace/morphic/main/docs/assets/dexpace-wordmark-light.svg" width="320">
  </picture>
</p>

<p align="center">Standards-faithful, typed tooling for building SDKs and API clients.</p>

dexpace is an open engineering effort building the plumbing developers need to ship SDKs and
API clients: immutable request/response models, staged policy pipelines, and a spec-to-SDK
compiler that treats no target language as an afterthought. Everything here favors correctness
and explicitness over convenience shortcuts — see [styleguide](https://github.com/dexpace/styleguide)
for the rules we hold ourselves to.

## Flagship: morphic

[**morphic**](https://github.com/dexpace/morphic) is a spec-to-SDK compiler. It lowers an API
specification — OpenAPI today, with Smithy, TypeSpec, GraphQL, and more planned — into one
spec-agnostic intermediate representation, then generates idiomatic SDKs and docs from that IR.
One IR, many targets, by design.

**Status: early development.** The IR package and the OpenAPI 3.x compiler are implemented and
exercised end-to-end by the `morphic compile` CLI. Emitters (the IR → SDK side) aren't built
yet, and there's no released version — the IR schema is still unstable.

## SDK toolkits, one architecture per language

Independent of morphic, dexpace also hand-builds the lower-level HTTP-client toolkit that a
generated or hand-written SDK sits on top of — immutable request/response models, a staged
policy pipeline, pluggable transports, and an auth layer — ported consistently across five
languages:

| Repo | Language | Status |
|---|---|---|
| [java-sdk](https://github.com/dexpace/java-sdk) | Kotlin (JVM) | Alpha (`0.0.1-alpha.1`) — public API stabilizing |
| [python-sdk](https://github.com/dexpace/python-sdk) | Python 3.12+ | Typed end-to-end under `mypy --strict` |
| [go-sdk](https://github.com/dexpace/go-sdk) | Go | Built directly on `net/http` |
| [dotnet-sdk](https://github.com/dexpace/dotnet-sdk) | C#/.NET | Alpha — foundation slice: core models plus one reference transport |
| [ruby-sdk](https://github.com/dexpace/ruby-sdk) | Ruby | Just started |

## Available today: kuri

[**kuri**](https://github.com/dexpace/kuri) is a standards-faithful URI and URL library for
Kotlin Multiplatform and Java — RFC 3986, the WHATWG URL Standard, and UTS #46 for
internationalized hosts. Published to Maven Central and ready to use now.

[![Maven Central](https://img.shields.io/maven-central/v/org.dexpace/kuri?logo=apachemaven&logoColor=white&label=Maven%20Central)](https://central.sonatype.com/artifact/org.dexpace/kuri)

## Conventions

Everything above follows one shared [styleguide](https://github.com/dexpace/styleguide) —
cross-cutting engineering rules plus per-language guides, distributed as an installable
Claude Code skills plugin.

---

[Join us on Discord](https://discord.gg/hPXjhXKbF)
