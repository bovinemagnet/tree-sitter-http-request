# tree-sitter-http-request

A [Tree-sitter](https://tree-sitter.github.io/tree-sitter/) grammar for
IntelliJ-style `.http` / `.rest` request files, including the `GRAPHQL`
pseudo-method.

This repository is the **canonical source** of the grammar. The
[`zed-http-client`](https://github.com/bovinemagnet/zed-http-client) Zed
extension consumes it directly: its `extension.toml` pins this repository
and a commit SHA under `[grammars.http_request]`, and Zed clones the
grammar at that SHA when the extension is installed. `zed-http-client`
keeps no copy of the grammar — change it here.

## Scope

The grammar exists purely for **editor syntax highlighting and structural
queries** (Zed's `highlights.scm`, `runnables.scm`, and so on).

It is *not* used to execute requests. The `zed-http` CLI in
`zed-http-client` has its own hand-written Rust parser, which remains the
source of truth for execution semantics. Keeping the two separate is
deliberate: the grammar can stay deliberately lightweight.

## Grammar structure

A file is a sequence of comments, `@variable` declarations, and `###`
request blocks. The notable node types are:

| Node | Description |
|------|-------------|
| `comment` | `#`, `##`, or `//` line comment |
| `variable_declaration` | `@name = value`, with `name` and `value` fields |
| `request` | a `###` block: separator, optional `name`, `request_line`, `header`s, optional `body` |
| `request_line` | `method` + `url` |
| `method` | `GET` / `POST` / `PUT` / `PATCH` / `DELETE` / `HEAD` / `OPTIONS` / `GRAPHQL` |
| `interpolation` | a `{{variable}}` reference, valid inside `url` and `header` values |
| `header` | `header_name` + `header_value` |
| `body` | the request body, currently a single opaque token |

These node names are the grammar's public API — downstream query files
depend on them, so renaming one is a breaking change (see Versioning).

## Development

Requires the [Tree-sitter CLI](https://github.com/tree-sitter/tree-sitter).

```bash
npx tree-sitter generate     # regenerate src/parser.c from grammar.js
npx tree-sitter test         # run corpus tests
npx tree-sitter parse FILE   # parse a file and print its tree
```

`src/parser.c` is generated but committed, because Zed builds the grammar
from the checked-in source — always commit it alongside a `grammar.js`
change.

## Versioning

The grammar follows [Semantic Versioning](https://semver.org/) on its
**node-name API**: renaming or restructuring a node is a breaking change.

It is intentionally kept in the `0.x` range while the node structure is
still settling (for example, the opaque `body` node may later be split so
editors can inject JSON and GraphQL highlighting into request bodies).
A `1.0.0` release will follow once the node names are frozen.

## Releasing

1. Edit `grammar.js` and run `npx tree-sitter generate`.
2. Commit both `grammar.js` and the regenerated `src/parser.c`.
3. Bump the `version` in `package.json` and tag the commit
   (`git tag -a vX.Y.Z`).
4. In `zed-http-client`, copy the new commit SHA into the `rev` field
   under `[grammars.http_request]` in `extension.toml`.

## Licence

Apache-2.0. See [LICENSE](LICENSE).
