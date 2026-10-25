# netforge-survivors-like-template: guide for coding agents

This game is built with Net Forge, the open-source multiplayer browser game engine built for AI coding agents (`@netforge` on npm, https://github.com/netnetdev/net-forge and https://forge.netnet.capital/docs).

It runs on the `survivor` template (EMBERWATCH, survivor-like co-op, up to 4 players). The template package `@netforge/template-survivor` holds the client, the simulation and the bots. This project holds only the rules.

## Where things live

- `src/game.ts` is the whole game and the only code file you change. It is declarations against the typed API in `@netforge/engine`; it does not render, network or simulate anything itself.
- `src/game.ts` may import only `@netforge/engine`. Any other import (a relative file, `node:*`, another package) fails the build. Put helpers and constants in `src/game.ts` itself.
- The declarations this template uses: `achievement`, `director`, `earn`, `game`, `keeper`, `mission`, `prefab`, `store`, `system`, `title`, `upgrade`, `weapon`. Keep the `game({ template: "survivor", ... })` call and its `template` value.
- `llms.txt` is the template's guide. Read it before changing anything: its "Rules the build enforces" section lists the limits the build checks, and "Common changes" has a recipe for each usual change.
- Where `llms.txt` names monorepo paths, read the installed copies instead: `packages/engine/...` is `node_modules/@netforge/engine/...`, and `client/...` is `node_modules/@netforge/template-survivor/client/...`. Where it says `pnpm --filter @netforge/template-survivor test`, run `npm run test`.
- The typed API: `node_modules/@netforge/engine/dist/index.d.ts` (exports) and `node_modules/@netforge/engine/dist/api/schemas.d.ts` (every field each declaration accepts). The design rules the playtests score are in `node_modules/@netforge/engine/GAME-DESIGN.md`, and `node_modules/@netforge/engine/llms.txt` covers look and feel.
- `src/names.json` (optional) sets the title and every label players read: `{ "title": string, "weapons": { <id>: { "name", "line" } }, "upgrades": { <id>: { "name", "line" } }, "enemies": { <boss prefab id>: string }, "rules": { <earn rule id>: string }, "skus": { <sku id>: string } }`. Labels are 1 to 40 characters and ids are lower_snake_case. When the file exists it replaces the template's labels, so label every weapon, upgrade, earn rule and store item you declare.
- `netnet.json` names the template. Do not change its `template` value: it must match the installed template package.
- Do not edit `node_modules/` or `.netnet/` (build output). Do not add a client, a server or a renderer: the template provides them.

## Run and test

```sh
npm install                     # once
npm run test                     # compile the rules and run the playtest bots; exits non-zero on a failure
npm run check                    # the same, with the full report as JSON on stdout
npm run typecheck                # the TypeScript check of src/game.ts
npm run play -- --no-open       # build and serve at http://127.0.0.1:4663 until stopped
```

- After every change, run `npm run test` first. It compiles `src/game.ts` exactly as the play build does (the same validation errors) and then plays it with bots, so it catches broken rules without a browser. `npm run test -- --json` prints the full report, and `npm run test -- --minutes 4 --bots 2` plays longer.
- The play command does not exit on its own: it serves the game until Ctrl+C. In an agent session, start it in the background or skip it and rely on the test command. It rebuilds on each start, so stop it and start it again after a change.
- Never invent asset ids. Model ids look like `style.lowpoly.<name>`; reuse ids that already appear in `src/game.ts` or `llms.txt`.

## Common errors

| Message | What to do |
|---|---|
| `src/game.ts may import only "@netforge/engine"` | Remove the other import and inline what it gave you. |
| A declaration and a field, for example `prefab("shade"): - health: Too small: expected number to be >=1` | The value is outside what the schema accepts. Check the field in `schemas.d.ts` and the limits in `llms.txt`. |
| `game module: - every game needs ...` | A required declaration is missing (for example `earn.rule` or `earn.stake`). Put it back. |
| `@netforge/template-survivor is not installed` | Run `npm install` in this folder. |
| `netnet.json names an unknown template` | Restore the `template` value in `netnet.json`. |
| `src/names.json ...` | Fix the file as the message says, or delete it to use the template's labels. |
| `EADDRINUSE` or the port is taken | Another game is running. Stop it, or pass `--port 4700`. |
| A playtest check failed, or `softlock` | The bots could not make progress: an enemy too strong, a wave that never ends, a goal that cannot be reached. Undo the last change or tune it, then test again. |

## The Net Forge MCP server

`.mcp.json`, `.cursor/mcp.json` and `.vscode/mcp.json` register the Net Forge MCP server (`npx -y @netforge/mcp`). When your client has it, `netforge_validate_game` and `netforge_run_playtest` do what the test command does, and `netforge_search_docs` and `netforge_explain_api` look up the API before you guess. Docs: https://forge.netnet.capital/docs

## Money

Local play is play money only. Earn rules (`earn.rule`) and the store (`store.sku`) still fire so you can see them, and pay nothing. Keep the earn rules, the `earn.stake` line and the store SKUs the template declares: the template's client is wired to them and the build requires them. Real payouts are decided only by Net Forge's hosted servers, never by this code.
