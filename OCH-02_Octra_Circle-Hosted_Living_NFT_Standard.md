# OCH-02: octra circle-hosted living nft standard (v1.1)

euint labs.

OCH-02 is how wallets, marketplaces and explorers read and render a **living** circle-hosted nft: a piece that keeps changing with the chain after mint. OCH-01 pieces render from their seed alone and never change. OCH-02 pieces render from their seed plus their on-chain state.

the standard covers two things only: the renderer, and how clients read these nfts. everything a collection builds on top (how it mints, reveals, evolves, prices or stores value) is up to the deployer and outside this standard. a plain living nft with no extras is fully OCH-02.

every OCH-02 contract is a full OCH-01 ledger. wallets and marketplaces that read OCH-01 keep working.

## 1. what changes from OCH-01

| | OCH-01 | OCH-02 |
|---|---|---|
| render input | seed | seed + state |
| renderer network | none | none required. optional read-only refresh from its own contract |
| `token_uri` | `oct://` file | `oct://` file, or inline `data:` json |
| seed | `sha256(SALT:id)` | defined by the collection, returned by `seed_of` |
| tokens | fixed set | may be burned or created after mint |

## 2. core contract interface

the full OCH-01 contract interface (OCH-01 §2.1 to §2.8) with the exact signatures, plus:
```
view fn state_of(token_id: u64): string
view fn animation_url_of(token_id: u64): string
```
- `state_of` returns `key=value` pairs joined by `;`, integer values only. keys are defined by the collection. clients ignore keys they don't know.
- `animation_url_of` returns the renderer url for the token with the current state baked in (§3).
- `seed_of(id)` returns the seed the renderer needs. how it is derived is the collection's choice and must be deterministic from on-chain data.
- a burned or never-minted id makes `owner_of`, `token_uri`, `state_of` and `animation_url_of` revert.

## 3. the renderer

one self-contained html file in the renderer circle. it must:
- render completely from `seed` and `st` with **zero network requests**. `st` is the url-encoded `state_of` value. wallets with no network see a correct, live piece.
- load no subresources: no external or sibling files, everything embedded.
- give an identical render for the same seed and the same state.
- respect `prefers-reduced-motion` and pause on `visibilitychange`.
- start sound only after a user gesture, if it has sound.

it may:
- refresh its state with read-only `contract_call` requests to the `rpc` url param, for the `contract` and `token` params only, and redraw when the state changes. no other network, no writes.
- answer to touch and clicks.

url:
```
oct://<renderer_circle>/index.html?seed=<seed>&st=<state>&contract=<contract>&token=<id>&rpc=<rpc>
```
`seed` is required. `st` is required whenever the token has state. `contract`, `token` and `rpc` are optional and only enable the refresh. `rpc` is a full, url-encoded rpc endpoint (for example `https%3A%2F%2Foctra.network%2Frpc`), never a short name.

## 4. metadata

`token_uri(id)` returns either:
- `oct://<metadata_circle>/<path>.json`, a json file in the metadata circle; or
- `data:application/json;utf8,{...}`, inline json from the contract. used for tokens created after deploy, or metadata that depends on state.

either way the json has OCH-01's token fields:
- `name`, `description`;
- `image` = `animation_url` = `external_url` = the §3 url;
- `attributes`.

`collection.json` and `traits.json` follow OCH-01, with `"render_type": "living"`.

names: when `name` carries a number, it must be the token id, never an art index or a position.

## 5. provenance

as OCH-01: `get_provenance` set and `lock_provenance` called before the first mint. the hash must commit to the renderer and to every seed or art input the collection fixed at deploy. how the hash is built is the collection's choice, published with the collection, and recomputable by anyone from the circles.

## 6. tokens that change over time

clients must handle these cases, however a collection produces them:
- **sealed tokens.** before a collection assigns art, `token_uri` may point to one shared sealed file and `seed_of` may revert. show the sealed metadata; read again later.
- **burned tokens.** `owner_of` reverts. treat the token as gone, not as an error.
- **tokens created after mint.** ids may exceed `get_max_supply`. their `token_uri` is usually inline `data:` json.
- **changing metadata.** `token_uri`, `name` and `attributes` may change after mint. never cache them forever.

## 7. adopter integration

wallets:
- render `animation_url` in an iframe that allows scripts and pointer events;
- **keep the full query string**: `seed`, `st`, `contract`, `token`, `rpc`;
- allow audio after a user gesture;
- may allow read-only network to the `rpc` param, or block all network (the piece still renders from `st`).

all clients:
- re-read `token_uri` instead of caching it forever.
- treat a reverting `owner_of` as "this token no longer exists".

marketplaces: as OCH-01. read `attributes` live; they may change.

explorers: as OCH-01, plus `state_of`.

## 8. conformance

- the core interface (§2) with exact signatures.
- the renderer rules (§3).
- metadata shapes (§4) and provenance set and locked (§5).

run `node conformance.mjs <contract> [rpc]`.
