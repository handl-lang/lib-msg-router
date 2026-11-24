# lib-msg-router

Pure-handl **message/event-envelope dispatcher** over
[`lib-route`](https://github.com/handl-lang/lib-route) (P-008 / T-142).

Where `http-router` adapts lib-route to HTTP requests, `msg-router`
adapts it to message envelopes — chat events, webhook deliveries,
queue messages, actor mailboxes — anything shaped
`{ type: "message.created", ...payload }`.

## Usage

```hnl
from "msg-router" import * as mr

dispatch := mr.router({
  "message/:channel":  (ctx) ~> "msg in " + ctx.params.channel,
  "reaction/:emoji":   { handler: on_reaction,
                         name: "on-reaction",
                         doc: { summary: "emoji reaction" } },
  "*":                 (ctx) ~> "unhandled"
}, { type_key: "type", sep: "/" })

dispatch({ type: "message/general", text: "hi" })
-- ctx.req is the WHOLE envelope; ctx.params.channel == "general"
```

## Route table

`routes` is a **flat** record `pattern -> handler | { handler, doc?, name? }`.
Keys are lib-route patterns (literals, `:param`, `*name`/`*` splats,
`(/optional)` groups) whose segments split on `opts.sep` (default `"/"`;
`"."` for `message.created`-style types). v1 has no nested subtrees —
deep types are just more segments.

Leaf values are either a handler arrow or a spec record
`{ handler, doc?, name? }`. `doc`/`name` survive compilation on
`ctx.hit.entry` — `doc` is the future **AsyncAPI** spec surface (same
role OpenAPI plays for http-router); `describe(routes)`/`usage(routes)`
render `[String]` lines for debugging and spec generation.

## Envelope contract

- `opts.type_key` — record key holding the type string
  (default `"type"`; e.g. `"kind"`, `"event"`). Missing/empty → miss
  path, never the `"*"` route. Non-string values are stringified.
- `opts.sep` — separator splitting the type into tokens (default `"/"`).
  Route keys are translated `sep -> "/"` before compiling; splat-bound
  param remainders are translated back (`*rest` on `evt.a.b.c` with
  `sep: "."` binds `"a.b.c"`).
- Handler ctx: `{ req: <whole envelope>, state, params, hit, type }`.
- Handler result: a record with `out` is the `{ out, state }` pair;
  anything else normalizes to `{ out: r, state: {} }`. The dispatcher
  returns the final `out`.

## Miss path

No candidate (including when `"*"` is absent) → `opts.on_miss` called
as `on_miss(envelope) ~> out` if provided; otherwise
`error.raise("msg-router: no route for type '...'")`.

## Middleware

`opts.middleware` — optional `[(ctx, next) ~> { out, state }]` list
folded via `rt.pipeline` (lib-route onion semantics, `state_get`/
`state_put`). Middleware wraps hits and misses alike.

## Development

```bash
HNL=~/.local/bin/hnl bash scripts/hnl-gate.sh   # typecheck + lint + test
~/.local/bin/hnl test --jobs 2 src/index.test.hnl
```
