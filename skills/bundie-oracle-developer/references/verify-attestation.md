# Verify: how do I trust the read?

`read_price` returns a `signed_attestation` field — an ed25519 signature produced by Bundie's backend over the response. `verify_attestation` checks that signature locally so you know the price wasn't tampered with in transit. Without this step, you're trusting the MCP transport and any proxy in between; with it, you're trusting Bundie's signing key.

## Tool sequence

```
read_price { event_id }                  # returns signed_attestation
verify_attestation { ...read_price... }  # local ed25519 check
                                         # public key is fetched + cached
                                         # automatically
```

## Example call

```jsonc
// verify_attestation {
//   event_id: "aws_us_east_1_incident_30min_30d",
//   as_of: "2026-05-14T11:42:08Z",
//   price: 0.087,
//   signed_attestation: "4tG7...base64...==",
//   payload: { ...the full read_price response... }
// }
{
  "valid": true,
  "signer": "BundieAttestKey11111111111111111111111111111",
  "algorithm": "ed25519",
  "signed_over": "full_response"
}
```

## What gets signed

Two canonical forms are supported:

- **`full_response`** (preferred) — the entire `read_price` response with the `signed_attestation` field stripped, serialised as JSON. This is what Bundie's backend signs. Pass the response back as `payload`.
- **`minimal_triple`** — `{ event_id, as_of, price }` only. Used when you've kept just those three fields and dropped the rest. Omit `payload` to trigger this path.

If you control the agent end-to-end, always use `full_response`. The minimal form is a fallback for clients that have already discarded the response.

## When verification fails

A failed verification means one of:

1. The payload was tampered with between Bundie and your agent (proxy, MITM, buggy serialiser).
2. The cached public key is stale because Bundie rotated keys mid-deploy.
3. You signed over the wrong canonical form (e.g., you passed only `price` but the response was signed over the full body).

The fix in order:

1. Re-call `read_price` and try again. Transient transport corruption clears.
2. Bounce the MCP client process so the cached attestation key is refetched.
3. If verification still fails after a fresh key, the response is not authentic — do not act on the price.

## Local verification without the tool

The tool is for convenience. If you want to verify in your own code:

```ts
import nacl from "tweetnacl";
import bs58 from "bs58";

// 1. Fetch the public key once: GET ${BACKEND_URL}/v1/attestation-key
const { public_key } = await fetchAttestationKey();
const pubkey = bs58.decode(public_key); // 32 bytes

// 2. Strip signed_attestation and serialise canonically.
const { signed_attestation, ...canonical } = priceResponse;
const message = new TextEncoder().encode(JSON.stringify(canonical));

// 3. Decode signature (base64 or hex) and verify.
const sig = Buffer.from(signed_attestation, "base64");
const valid = nacl.sign.detached.verify(message, sig, pubkey);
```

The MCP `verify_attestation` tool does exactly this internally.

## Common pitfalls

- **Don't re-serialise with a different JSON canonicaliser.** `JSON.stringify` on the same object can produce different byte strings if you've mutated key order. Pass `payload` straight from `read_price` without modification.
- **Don't cache the public key across deploys you don't control.** Bundie's signing key is stable within a deploy but the team may rotate it on releases. `verify_attestation` refetches on key mismatch; if you build your own verifier, treat a verify failure as a signal to refetch the key.
- **Don't skip verification because it "always passes."** The day it fails is the day someone is feeding your agent fake prices. The cost of the call is zero; run it every time.
