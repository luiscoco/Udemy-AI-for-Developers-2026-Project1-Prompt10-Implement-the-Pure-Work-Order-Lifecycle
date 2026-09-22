# Prompt 10 Implement the Pure Work-Order Lifecycle

This README walks through the steps taken to complete the latest prompt, so students can
follow the reasoning and reproduce it themselves.

## The prompt

> Create `apps/backend/src/domain/workOrderLifecycle.ts`. Import `WorkOrderState` and
> `WorkOrderAction` from `@equipment-hub/contract`. Implement `transition(state, action)` and
> `allowedActions(state)`, following the work-order lifecycle rules, as a pure module with no
> Fastify, file access, or thrown exceptions.

## Steps followed

1. **Located the shared contract types.** Before writing any code, searched the codebase for
   `WorkOrderState` and `WorkOrderAction` to confirm where they are defined and exported from,
   rather than guessing their shape:
   - `packages/contract/openapi.yaml` — the source-of-truth OpenAPI schema.
   - `packages/contract/src/types.gen.ts` — generated TypeScript types from that schema.
   - `packages/contract/src/index.ts` — re-exports `WorkOrderState` and `WorkOrderAction`, plus
     `STATES` and `ACTIONS` const arrays used for runtime checks.

   This confirmed the exact states (`reported`, `triaged`, `scheduled`, `in_progress`,
   `completed`, `cancelled`) and actions (`triage`, `schedule`, `start`, `complete`, `cancel`).

2. **Modeled the lifecycle as a transition table.** Instead of a long `if/else` or `switch`
   chain, the allowed transitions were encoded as a lookup object:
   `Record<WorkOrderState, Partial<Record<WorkOrderAction, WorkOrderState>>>`. Each state maps
   only to the actions that are legal from it, and each of those maps to the resulting state.
   `completed` and `cancelled` map to empty objects, which encodes that they are final states
   with no legal actions.

3. **Implemented `transition(state, action)` as a pure function.** It looks up the next state in
   the table. If found, it returns `{ ok: true, state: nextState }`. If not found (illegal
   action for that state), it returns `{ ok: false, reason: "..." }` instead of throwing — this
   keeps the module free of exceptions, as required.

4. **Implemented `allowedActions(state)`.** Returns the list of actions available from a given
   state by reading the keys of that state's entry in the transition table
   (`Object.keys(TRANSITIONS[state])`).

5. **Kept the module pure.** No imports of Fastify, no file system access, no network calls, and
   no thrown exceptions anywhere in the file — only plain data and functions, so the lifecycle
   logic can be unit-tested in isolation and reused by any HTTP layer later.

## Running the app (Windows Terminal / PowerShell)

This repo is an npm workspaces monorepo (`apps/backend`, `apps/frontend`, `packages/contract`).
As of this prompt, there is **no working Fastify server or Vite dev server wired up yet** —
`apps/backend/package.json` and `apps/frontend/package.json` don't define `dev`/`start` scripts,
and the root `dev` script is only a placeholder. So there is nothing to actually "run" as an app
right now; the commands below are what is currently functional.

1. Install dependencies (once, from the repo root):

   ```powershell
   npm install
   ```

2. Run the contract package's tests (verifies the OpenAPI-generated types):

   ```powershell
   npm run test --workspace=@equipment-hub/contract
   ```

3. Run the root placeholder scripts:

   ```powershell
   npm run dev
   npm run test
   npm run lint
   ```

   `npm run dev` currently just prints `no dev server configured yet` — once a real backend
   (Fastify) and frontend (Vite) are implemented in a later prompt, this section should be
   updated with the real `npm run dev` command that starts them.

## Result

The lifecycle rules from the prompt are enforced by data (the `TRANSITIONS` table) rather than
scattered conditionals, which makes the rules easy to read at a glance and easy to extend if a
new state or action is added later.

See [`apps/backend/src/domain/workOrderLifecycle.ts`](apps/backend/src/domain/workOrderLifecycle.ts)
for the full implementation.
