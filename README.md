<div align="center">
<img src="assets/social-preview.png" alt="output-gate" width="860">
</div>

<h1 align="center">output-gate</h1>

<p align="center"><b>Before AI-written text ships, run the four checks it always fails.</b><br><sub>11 files (a small engine + rulesets) · zero dependencies · stdlib only</sub></p>

<p align="center"><a href="https://github.com/chenhz01/output-gate/actions/workflows/gate.yml"><img src="https://github.com/chenhz01/output-gate/actions/workflows/gate.yml/badge.svg" alt="tests"></a> <img src="https://img.shields.io/badge/python-3.9%2B-3776AB?style=flat-square" alt="python"> <img src="https://img.shields.io/badge/dependencies-0-2ea44f?style=flat-square" alt="deps"> <img src="https://img.shields.io/badge/license-Apache--2.0%20%2B%20CC%20BY%204.0-blue?style=flat-square" alt="license"></p>

---

## The failure this catches

- The delivery doc still contains '[TBD]' and an 'about 30%' that nobody verified.
- Page 3 says the budget is 80k; page 11 says 120k.
- Placeholders, fake citations, unit drift — a human has to read every page to catch them.

One command, on a file you already have:

```bash
python gate.py run ai-output examples/draft.md --format text
```

<details>
<summary><b>Real output</b> — produced by the bundled <code>examples/</code>, not a mock-up</summary>

````text
==========================================================================
output-gate · 规则集 ai-output v1.1.0
目标：客户方案 v3
场景：AI 产出通用体检
==========================================================================
✗ [PLACEHOLDER](project_custom) 存在未替换的占位符（TBD/待补/XXX/{{}}）—— 发出去就是半成品
      → 2 处占位符未替换：第10行 待补、第11行 TBD
✓ [CODEBLOCK-UNCLOSED](project_custom) 代码块未闭合（``` 数量为奇数）
      → 0 处代码块均闭合
– [BRACKET-UNBALANCED](project_custom) 代码括号不平衡（() [] {} 未配对）—— 直接导致无法运行
      → 非代码文件，不适用
✓ [SECRET-LEAK](project_custom) 疑似密钥/令牌泄漏（sk-…、api_key、token=、private key）
      → 未检出疑似凭据
✓ [PII-LEAK](project_custom) 疑似个人信息（手机号/身份证/银行卡）—— 需人判断是否为公开信息
      → 未检出个人信息特征
⚠ [CLAIM-NO-SOURCE](project_custom) 含统计数字的断言没有出处标记 —— 读者无法验证
      → 2/7 条含数字无出处（29% ≥ 15%）。例：第4行「整体预算约 30 万，其中开发占 60%」；第14行「历史版本预算为 8 万，本版调整为 12 万」
✓ [CLAIM-ABSOLUTE](project_custom) 不可验证的绝对化表述（一定/绝对/100%/永久/零风险）
      → 无绝对化表述
✓ [STRUCTURE-SHALLOW](project_custom) 标题层级过浅（无 h2 及以下）—— 多半是平铺直叙，难读
      → 5 个标题，4 个二级及以下
⚠ [SIZE-ANOMALY](project_custom) 篇幅异常（过短疑似敷衍 / 过长疑似灌水）
      → 仅 16 行，低于 20 行 —— 疑似敷衍
✓ [COMPLETION-NO-EXIT](project_custom) 自报完成（已完成/跑通/全部通过…）但全文无任何退路声明（未通过怎么办/回滚/降级/视为未完成）——按 ZEG 降级为「未验证的自报」而非完成
      → 无自报完成表述（ZEG 不适用）
--------------------------------------------------------------------------
合计 10 条：BLOCK 1 ｜ WARN 2 ｜ NEEDS_VALIDATION 0 ｜ INFO 0 ｜ PASS 6 ｜ SKIP 1
  ⛔ 本规则集做【形式与可追溯性】检查，不判断内容对错
  ⛔ WARN 项需人判断 —— 不要把 WARN 当成「有问题」的证据
  ⛔ 「无源数字」不等于「数据是假的」，只等于「读者无法验证」
结论：不通过（存在 BLOCK 项），先修再发布。
````

</details>

## What it does

- Rule-driven checks: placeholders, unsourced numbers, numeric conflicts, unit drift.
- Emits text / json / sarif / junit — SARIF lands straight in code scanning.
- Keeps a coverage ledger across runs: what newly failed, what you actually fixed.

## What it does **not** do

- It does not verify facts. It only checks that numbers declare a source.
- Rulesets are opinionated and domain-bound (ai-output, design, ecommerce, music, shortvideo).
- It will not catch a plausible lie written fluently.

Being explicit here is cheaper than a wrong expectation later.

## Rulesets

52 rules across five rulesets — one engine, one directory per vertical.
`python gate.py list` prints this list live, with each ruleset's own version.

| Ruleset | What it checks | Rules | Version | Run it (exits 1) |
|---|---|---|---|---|
| `ai-output` | AI 产出通用体检 | 10 | v1.1.0 | `python gate.py run ai-output examples/draft.md` |
| `design` | 设计输出体检 | 5 | v0.1.0 | `python gate.py run design rulesets/design/fixtures/design_fail.html` |
| `ecommerce` | 跨境电商 listing 合规 | 10 | v2026-10-11.0 | `python gate.py run ecommerce rulesets/ecommerce/fixtures/listing_fail.json` |
| `music` | 音乐发行前合规体检 | 20 | v2026-10-04.1 | `python gate.py run music examples/track_fail.json` |
| `shortvideo` | 短视频 / 口播脚本合规 | 7 | v1.0.0 | `python gate.py run shortvideo rulesets/shortvideo/fixtures/script_fail.json` |

The bundled `examples/` and `rulesets/*/fixtures/` are committed, so every
command above reproduces on a fresh clone. The clean fixture of each ruleset
exits `0` and the failing one exits `1`; CI asserts both directions, so a green
badge means the ruleset fired when it should **and** stayed quiet when it should.

Adding a vertical means dropping in a directory — the engine does not change.

## Exit codes

`0` = clean (no BLOCK, no WARN). `1` = at least one BLOCK rule fired. `2` = WARN-level findings only — nothing blocks the build. `3` = you asked for something this gate could not do: a malformed command line, an unknown ruleset, an unreadable file, or an engine failure (it prints `✗ …` first). **`2` used to double as the usage-error code; since 2026-10-20 a mistyped command exits 3, so a non-zero badge always means a real finding.**

## Install

```bash
git clone https://github.com/chenhz01/output-gate
cd output-gate
python gate.py --help
```

Python 3.9+. Nothing to `pip install` — stdlib only.

## Use it in CI

```yaml
- name: output-gate
  run: python gate.py run ai-output examples/draft.md --format text
```

## Use it as a GitHub Action

`action.yml` is included, so you can call this repo directly as a composite action.

```yaml
# .github/workflows/gates.yml
- uses: actions/checkout@v4

- uses: chenhz01/output-gate@v0.1.3
  with:
    # IMPORTANT: this path is inside YOUR repository.
    # The examples/ in this repo are NOT in your workspace — pass your own files.
    args: "run ai-output path/to/draft.md --format text"
```

Pin the tag (`@v0.1.3`) rather than `@main` — `main` moves.
Check [Releases](https://github.com/chenhz01/output-gate/releases) for the newest tag; `v0.1.0` predates the
fix that makes a mistyped path fail instead of silently passing.

If you omit `args`, the action only prints help and exits 0. It raises a warning
annotation — quoting this gate's own correct usage — so a green check is never
mistaken for "it inspected my files".


## 中文速览

**产出体检门** — AI 写的东西发出去之前，先跑一遍它最常犯的四类错。

- 交付文档里留着「[待补充]」和一个没人核过的「约 30%」。
- 同一份方案里，第 3 页说预算 8 万，第 11 页说 12 万。
- 占位符、伪引用、单位错乱——每一份都要人肉读一遍才发现。

一行命令：`python gate.py run ai-output examples/draft.md --format text`

边界：It does not verify facts. It only checks that numbers declare a source.

### 互推：只装一个的话，装哪个

十五个链接是迷宫，一个症状是门。按上表挑一个，其余九个在下面 [The gates](#the-gates)。
每个仓都能单独活：复制 `gate.py`，一条命令跑完，不共享运行时、不装包、不加配置。
有星就给这一个，没有也不影响任何仓的可用性——没有任何门是「先给星才能用」。

### 热门前置：保证结构，不保证排名

本页承诺的是 **15 项可机器复核的结构特征**，当前 **15/15**（2026-10-20 复核）。
GitHub 从未公开 Trending 的排序算法，所以任何「保证上热门」的说法都在描述一件它不可能知道的事；
本仓不这么说。跑 `python tools/hotness.py` 可自行复算每一条。

## Start here (if you install only one)

Fifteen links is a maze; one symptom is a door. Find your symptom, install that gate,
ignore the rest until it hurts.

| what you are seeing | install this |
|---|---|
| the bill is 10x normal and the trace is a wall of the same step | **[loop-gate](https://github.com/chenhz01/loop-gate)** |
| the agent can delete prod and nobody signed off on that | **[permission-gate](https://github.com/chenhz01/permission-gate)** |
| answers get worse after truncation and nobody knows what got dropped | **[context-gate](https://github.com/chenhz01/context-gate)** |
| two rules in the prompt contradict each other; the model obeys neither | **[instruction-gate](https://github.com/chenhz01/instruction-gate)** |
| AI text ships with fabricated links and empty phrases | **output-gate** ← you are here |
| half the repo might be dead and nobody can prove which half | **[deadcode-gate](https://github.com/chenhz01/deadcode-gate)** |

A gate you did not need is a gate you uninstall, so nothing here says "install all
fifteen". Each repo stands alone: copy `gate.py`, run one command, no shared runtime,
no package manager, no config file to grow.

**On stars.** If this one saved you an incident, star it — that is the only signal I get
about which gate to build next. There is no exchange: starring one repo does not owe me
a star on another, and no gate here is unlocked by a star. Star trading breaks GitHub's
rules and would poison the single honest metric this project has.

## Hot-ready guarantee

What is guaranteed is **structure**, not ranking. Fifteen properties, each one
machine-checked against this repo — current score **15/15** (2026-10-20).

Plain version: instead of asking you to trust a badge, I list what I actually checked,
name the script that checks it, and let you re-run it. If one of these lines is false,
that is a bug in my repo, not a matter of taste.

- Falsifiable side: `python tools/hotness.py` re-runs every check and prints the evidence line
  for each one. You do not have to trust this paragraph.
- Non-falsifiable side: GitHub has never published how Trending sorts repositories, so
  any repo promising you a guaranteed spot is telling you something it cannot know.
  This one does not make that promise.
- If a claim on this page stops being true, that is a bug: open an issue and it gets
  fixed in the same week, not in the next launch.

What actually carries this repo:

- the command in this README is byte-for-byte the command CI runs <sub>(53-char argv, exact match)</sub>
- the output block is a real run of `examples/`, not a mock-up <sub>(Real output section)</sub>
- CI asserts the exit code, so the badge can turn red — and it has <sub>(asserts exit=1)</sub>
- every fixture is committed, so a fresh clone reproduces the output <sub>(1 file under examples/ tracked in git)</sub>
- `* -text` in `.gitattributes`: same bytes on Windows and Linux <sub>(.gitattributes * -text)</sub>

## The gates

| # | gate | what it stops |
|---|---|---|
| 02 | **[loop-gate](https://github.com/chenhz01/loop-gate)** | Find where your agent looped, before it burns another run |
| 03 | **[permission-gate](https://github.com/chenhz01/permission-gate)** | Score how much more your agent can do than it should |
| 04 | **[context-gate](https://github.com/chenhz01/context-gate)** | Measure what truncation deletes, before the model starts guessing |
| 05 | **[instruction-gate](https://github.com/chenhz01/instruction-gate)** | Two of your instructions contradict each other. This finds them |
| 06 | **output-gate** ← you are here | Before AI-written text ships, run the four checks it always fails |
| 07 | **[memory-expiry-gate](https://github.com/chenhz01/memory-expiry-gate)** | Give every memory a TTL, then prove the expiry semantics actually hold |
| 08 | **[drift-gate](https://github.com/chenhz01/drift-gate)** | When embeddings drift, retrieval quietly rots. Fingerprint it |
| 09 | **[price-gate](https://github.com/chenhz01/price-gate)** | Your cost model is only as good as the price table under it |
| 10 | **[complexity-ceiling](https://github.com/chenhz01/complexity-ceiling)** | Per-file complexity budgets that fail the build instead of the reviewer |
| 11 | **[deadcode-gate](https://github.com/chenhz01/deadcode-gate)** | Quantify how much of the repo is dead, instead of guessing |
| 12 | **[docrot-gate](https://github.com/chenhz01/docrot-gate)** | Docs don't die at once. They rot one stale command at a time |
| 13 | **[leak-gate](https://github.com/chenhz01/leak-gate)** | Scan for internal detail before the doc leaves the building |
| 14 | **[backup-gate](https://github.com/chenhz01/backup-gate)** | A backup you have never verified is a rumor |
| 15 | **[trail-gate](https://github.com/chenhz01/trail-gate)** | Hash-chained agent audit trail — and it verifies its own chain |

## License

- Code: **Apache-2.0** — see [LICENSE](LICENSE)
- Docs and example data: **CC BY 4.0** — see [LICENSE-SPEC](LICENSE-SPEC)

Attribution required when you redistribute the documentation; code can be used commercially
without a separate agreement. See [NOTICE](NOTICE).
