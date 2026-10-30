# OCH-02

the octra circle-hosted living nft standard. an OCH-02 piece is a circle-hosted nft that keeps changing with the chain after mint: the renderer draws from its seed plus its on-chain state.

OCH-02 covers the renderer and how wallets, marketplaces and explorers read these nfts. how a collection mints, reveals, evolves or prices is up to the deployer. every OCH-02 contract is a full OCH-01 ledger, so OCH-01 integrations keep working.

## files
- `OCH-02_Octra_Circle-Hosted_Living_NFT_Standard.md`: the spec. core interface (OCH-01 + `state_of`, `animation_url_of`), the living renderer rules, metadata shapes, provenance, how clients handle tokens that change over time, and adopter integration.
- `conformance.mjs`: checks a live contract + its circles against the spec.
- `LICENSE`: MIT.

## check a collection
```
node conformance.mjs <contract_address> [rpc_url]
```
rpc defaults to `https://octra.network/rpc`. it reads every core getter, checks the renderer rules, and checks one live token: `token_uri` resolves, the `animation_url` seed equals `seed_of`, and any number in the name is the token id. exit 0 conformant, 1 not. the public rpc rate-limits hard, so a full run may need patience.

## related
- OCH-01, the static circle-hosted nft standard: github.com/euintlabs/och01

## status
v1.1.
