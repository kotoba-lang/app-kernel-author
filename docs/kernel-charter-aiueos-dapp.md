# app-kernel-author（charter 判定）— itonami（dapp）上の分散型設計

状態: 第 1 スライス実装（2026-10-20）。正本はこの文書。
追跡: Linear JUN-128（親 JUN-116）。

## 位置づけ

app-kernel-author は **Kernel 判断核の authoring charter** —— 「`.kotoba` で
書くべき部分と、書いてはならない部分の境界を所有する」。**この repo は
charter（契約書）であって実装ではない**（実際の author は charter に従う
agent）。

## このスライス: charter 判定コア

| 層 | 実体 | 状態 |
|---|---|---|
| 判定コア guest | `aiueos/kernel_charter.cljk` — mechanism-judgment（**C は decision-free のみ、admission/validation 経路を足さない**）/ acceptance-ok（**`--jvm-free` のみ**、JVM は diagnostic であって acceptance ではない）/ parity-gate（**CID / exports / effects / bounds が一致するまで cutover 不可**）/ cutover-ok（**skip と pass を同じ顔にしない**） | **実装済み・実測** |
| 記述子 | `aiueos/kernel-charter.edn` | **実装済み** |
| parity | `test/kernel/charter_parity_test.cljk` — 正本（README.md の charter）の境界・受入・parity 規約と一致 | **実装済み・実測** |

実測: `amu test aiueos/kernel_charter.cljk` → **15/15**（jvm-kir / js / wasm）。
parity → 2 tests / 11 assertions / 0 failures。

## 正本の正確な規則（guest はこれに追従 —— charter の明文）

- **層の境界**（ADR-2607241100 D6 / ADR-2608261100）:
  - 判断 = `.kotoba` / `.cljk`（whole-component Q9 migration。
    **「predicate だけの shadow は移行進捗にならない」**）
  - 機構 = C/asm（decision-free: PCI/MMIO/GDT/IDT/paging。
    **「新しい admission/validation 経路を C に足さない」**）
  - build/acceptance = **`--jvm-free` のみ**（java/javac/clojure/clj を
    deny/trace、**fallback で JVM を使わない — block する**）。
- **parity ゲート**: payload CID・definition CID・exports/imports・effects・
  resource bounds が一致するまで consumer cutover しない。
  **「skip と pass を同じ顔にしない」**。
- 手順 6 段: 分類（:disposition 言語化）→ whole-component → build（両方）→
  oracle（JVM は diagnostic）→ parity → propose（worktree branch、main 直
  push しない）。
- **実測済みの罠 4 件**（fuel は policy バジェット / RW 境界 0x115000（N2）/
  pinned build は worktree / unlisted export 拒否の空 error）は host 側の
  手順知識として docs に反映済み。

## 未実装境界

- Q9 migration の機械的検査（whole-component の transitive closure 判定）。
- `amu check --jvm-free` の deny/trace 実行（host = amu）。
- mon（DNA）化、aiueos session 載せ（fleet 共通）。
