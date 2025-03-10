---
id: app-id
title: What are Applications IDs?
sidebar_label: App IDs
description: Learn about Avail's data availability chain
keywords:
  - docs
  - avail
  - availability
  - scale
  - rollup
image: https://docs.availproject.org/img/avail/AvailDocs.png
---

As a general purpose base layer, Avail is designed to support many
modular chains at the same time, providing consensus and data
availability to all of them simultaneously.

How does this work? Avail headers contain an index that allows a given
modular chain (or "application" in Avail terminology) to determine and
download _only_ the sections of a block that have data for that
particular application.

This has very important benefits, including:

- Modular applications are largely unaffected by other uses of the
  base layer at the same time.
- Block sizes can increase without requiring applications to fetch
  more data, because they don't fetch the whole block, only what's
  relevant to them.

Data availability sampling is still done on the entire block,
however--this is the process where clients sample very small parts of
the block at random to verify availability.

If you'd like to learn how your idea could
benefit from Avail, please [join our
Discord](https://discord.gg/S2XQJjHsZt). We'd love to chat.
