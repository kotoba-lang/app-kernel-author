# app-kernel-author

**Kernel 判断核の authoring charter — `.kotoba` で書くべき部分と、書いてはならない
部分の境界を所有する。** この repo は charter（契約書）であって実装ではない。実際の
author はこの charter に従う agent。

## 境界（ADR-2607241100 D6 / ADR-2608261100）

| 層 | 所有 | 書き方 |
|---|---|---|
| **判断 (decision)** | `.kotoba` / `.cljk` — aiueos の kernel objects | whole-component Q9 migration。namespace の全 public surface + transitive closure を移す。predicate だけの shadow は移行進捗にならない |
| **機構 (mechanism)** | C/asm — `aiueos.kernel` | decision-free のみ: PCI/MMIO/GDT/IDT/paging。新しい admission/validation 経路を C に足さない |
| **build/acceptance** | `--jvm-free` のみ | `amu check --jvm-free` / `amu compile --jvm-free`。java/javac/clojure/clj を deny/trace。fallback で JVM を使わない — block する |

## Author の手順（1 tranche = 1 判断核単位）

1. **分類**: 書けない理由を `:disposition` で言語化（恒久の安全 / backend 未達 /
   スライスの切り方）。分類なしの「書けない」を残さない。
2. **whole-component で移す**: public surface 全体。caller が前計算した scalar
   shadow は移行の証拠にならない（Q9 決定）。
3. **build**: verified native `kotoba check` / `kotoba compile` /
   `kotoba rad build` と `amu --jvm-free` の両方。
4. **oracle**: `.cljc` oracle は nbb/CLJS/native/Wasm か golden vector で照合。
   JVM oracle は diagnostic であって acceptance ではない。
5. **parity**: payload CID・definition CID・exports/imports・effects・resource
   bounds が一致するまで consumer cutover しない。skip と pass を同じ顔にしない。
6. **propose**: worktree branch → server-side merge。main 直 push しない。

## 落ちている罠（実測済み — 繰り返さない）

- CLI project route (`--source-path`) は `--fuel` を黙って捨てる →
  `policy {:budgets {:fuel N}}` を使う
- RW 境界 0x110000 は旧 RX 前提 — N2 では 0x115000
- amu pinned build は worktree `/tmp/amu-<sha>` で
- unlisted `aiueos-*` export 拒否が空 internal-error になる（amu#782 解消済みだが
  引き合いとして）

## 正本

- `lang/q9-migration.edn`（kotoba-lang）— machine authority
- `docs/adr/ADR-q9-whole-component-build-migration.md`
- aiueos の ADR 番号体系（ADR-0142/0175/0195 等は fuel 天井の経緯）
