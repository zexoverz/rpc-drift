# rpc-drift

Check that several RPC endpoints agree before trusting any of them.

Apps put two or three RPC URLs behind a fallback transport and assume they are the same chain at the
same height. They often are not: one node lags twenty blocks, one returns a different hash for the
same block during a reorg, one is on the wrong chain id after a config mistake, one reports a stale
pending nonce and a transaction goes out with a reused nonce. Nothing on npm measures this.

## Planned API

```ts
import { checkRpcs } from "rpc-drift";

const report = await checkRpcs(["https://a.example", "https://b.example"], { chainId: 84532 });
// report.head:      { number: 12345678n, hash, agreedBy: 2 }
// report.endpoints: [{ url, chainId, blockNumber, lagBlocks, latencyMs, ok }]
// report.problems:  [{ code: "wrong-chain" | "lagging" | "hash-mismatch" | "unreachable" | "nonce-disagreement", url, detail }]
```

- Queries `eth_chainId`, the latest block and one shared block hash on every endpoint in parallel.
- Optional nonce check for an address (`latest` and `pending`) across endpoints.
- `pickRpc(report)` returns the healthiest endpoint for a viem `fallback` transport.
- CLI: `npx rpc-drift <url> <url> ...`, nonzero exit when endpoints disagree.

Built at ETHGlobal Tokyo 2026 as a working project of the End Credits demo: the library is written in
a Claude Code session, and End Credits pays the open-source packages that session used.

## License

MIT
