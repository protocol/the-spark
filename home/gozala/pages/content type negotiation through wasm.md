---
title: Content type negotiation through WASM
---

## We would like to store arbitrary [[IPLD]] blocks, but consume those in a very specific representation.could view [[dag-cbor]] as JSON file
### View [[dag-cbor]] as a JSON file
### View markdown as rendered HTML
### View JS AST as JS source file.
### View encrypted file in plain text.
## Unlike arbitrary DAG encode / decode problem this is smaller in scope.
### WASM program could be give bytes corresponding to [[CAR]] file for the give [[DAG]].
### It's job will be to spit out bytes for [[CAR]] file which correspond to transcoded data.
