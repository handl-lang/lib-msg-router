# lib-msg-router

Pure-handl **message/event-envelope dispatcher** over
[`lib-route`](https://github.com/handl-lang/lib-route) (P-008 / T-142).
Where `http-router` adapts lib-route to HTTP requests, `msg-router`
adapts it to message envelopes — chat events, webhook deliveries,
queue messages, actor mailboxes — anything shaped
`{ type: "message.created", ...payload }`.

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | the dispatcher |
| `src/index.test.hnl` | the .hnl test suite |
| `pkg.hnl`, `ci.sh`, `scripts/hnl-gate.sh` | manifest (dep `route`) / bupha entry / gate |

## Contract

- `routes` is a flat record `pattern -> handler | { handler, doc?,
  name? }`; `router(routes, opts)` returns a dispatch arrow.
- `opts.type_key`/`opts.sep` control envelope type extraction
  (default `"type"`/`"/"`); `ctx.req` is the WHOLE envelope,
  `ctx.params` carries `:param`/`*splat` bindings; `"*"` is the
  catch-all, `on_miss` the fallback; `{out,state}` middleware
  pipeline plus `describe`/`usage` helpers.
- Pure `.hnl`; no Rust crate.

## Verification

```bash
./ci.sh   # bupha entry → scripts/hnl-gate.sh
```

## License

MIT.

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
