# Design and Solution Patterns

A documentation library containing reusable design- and solution patterns, supporting [Data Engine Thinking](https://dataenginethinking.com/en/).

## Getting started

Please have a look at [the introduction documentation](./docs/index.md) to get started!

## Implementation

This repository is intended to be cloned and modified for organization-specific scenarios. All files are text-based (MarkDown format, by default) for convenient editing and collaboration using Git. A DocFX file is also provided to generate static HTML from the repository's contents.

To generate the content as a website (on localhost port 8081), please run the following from the 'docs' directory of the repository:

```azurepowershell
docfx docfx.json --serve -p 8081
```
