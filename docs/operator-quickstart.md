# operator quickstart

この repo は **descriptor + gate** であって実行系ではない（[README](../README.md) /
[ADR-0001](adr/0001-descriptor-snapshot-not-an-executor.md)）。だから operator の仕事は
「起動する」ことではなく、**名乗っていることが今も本当かを確かめる**ことである。
所要 1〜2 分（`--network` を含めて 3 分）。

必要なもの: `nbb`（ClojureScript on Node）と `curl`。**この repo に依存パッケージは
無い** —— `deps.edn` も `package.json` も持たないので、install する物は何も無い。

## 1. 取得する

```bash
west update --fetch smart marine-insurance
cd orgs/cloud-itonami/marine-insurance
```

west を使わないなら `git clone git@github.com:cloud-itonami/marine-insurance` でよい。
なお **remote 名は `origin` ではなく `cloud-itonami`**（west が付ける名前）なので、
west 経由の checkout で `git fetch origin` は通らない。

## 2. 構造・gate・固定値・文書を検査する（network 不要）

```bash
nbb --classpath src:test run_tests.cljs
```

最後の 3 行がこうなれば緑:

```
0 failures, 0 errors.

mode: offline
marine-insurance actor: all green
```

見ているもの:

- **gate が緩む方向**（安全側）— 無 attest で `:ready` にならない / 7 本の gate が
  AND である / `false` を attest とみなさない / blocked な plan が `:records` を
  持ち歩かない / effect が他 actor に帰属しない / 宣言外の collection に書かない /
  未知の cell が黙って no-op にならない
- **gate がきつくなる方向**（生存側）— attestation の 4 形（keyword の set / keyword
  map / string map / string の set）を全部受ける。1 本落ちると**その形で attest して
  いる呼び出し側だけが黙って全 blocked** になり、安全側なので事故に見えない
- **descriptor 本体** — step が宣言外の capability を呼んでいないこと（およびその逆＝
  使われない過剰付与が `agent.invoke` ちょうど 1 個であること）、cron が 5 field で
  あること、`did.json` の `service[].id` が自分の DID の fragment であること
- **固定値** — pipeline 10 / cell 18 / gate 7 などの census が実体と一致。count は
  **両方向に**落ちる（増えても減っても赤）
- **文書そのもの** — README の数・2 つの DID・相対リンク・quickstart が名指しする
  ファイルの実在。**踏めない手順を書けないようにするため**
- **`docs/identity-claims.edn` の URL が DID から導けること** —— 手書きの URL は
  「存在しない URL を測って 404 だと報告する」ので、`didweb` の規則から導出して
  突き合わせる

## 3. 名乗りを実際に解決しに行く

```bash
nbb --classpath src:test run_tests.cljs --network
```

`mode: offline + network` になる。curl で各 DID / 配信面を引き、
`docs/identity-claims.edn` の `:measured` と突き合わせる。

⚠ **`0 failures` は「全部健全」という意味ではない。** この repo の identity は
割れており（README 参照）、**割れていること自体が固定値として記録されている**。
緑なのは「実測が固定値と一致した」という意味でしかない。

## 4. 検査が本当に噛むかを確かめる（任意・数分）

緑を見ているだけでは、テストが静かに噛まなくなったことに気づけない。
superproject 側の mutation runner が、**壊して赤くなること**を確かめる:

```bash
cd <superproject root>
nbb scripts/maturity-loop/run.cljs --only marine-insurance
```

使い捨て worktree を west の pin から切って壊すので、**共有 checkout には触れない**。
`噛まない` が 1 つでも出たら、それは「そのテストはその不変条件を守っていない」という
具体的な TODO である。

## 5. gate を手で撃ってみる（任意・5 秒）

```bash
nbb --classpath src -e '(require (quote [marine_insurance.murakumo :as m]))
  (let [blocked (m/cell-plan :getvesselcoverage {:attestations {}})
        ready   (m/cell-plan :getvesselcoverage {:attestations (into #{} m/common-gates) :request-id "req-1"})]
    (println "blocked:" (:status blocked) "effects" (count (:effects blocked)) "missing" (count (:missing-gates blocked)))
    (println "ready:  " (:status ready)   "effects" (count (:effects ready))   "->" (:collection (first (:effects ready)))))'
```

```
blocked: :blocked effects 0 missing 7
ready:   :ready effects 1 -> com.etzhayyim.marine-insurance.getvesselcoverage
```

**`:ready` は「書いた」ではない。** 返るのは `{:op :mst/put-record ...}` という
data であって、それを実行する者はこの repo に居ない。

## 赤くなったら

**壊れたとは限らない。直ったのかもしれない。** この repo の test の多くは
「測って、直していない」現状（`docs/identity-claims.edn` の `:gaps`）を固定している
ので、穴を塞ぐと赤くなる。どちらの場合もやることは同じ:

1. 失敗行を読む（固定値と実測値の両方が出る）
2. `docs/identity-claims.edn` の `:measured` / `:census` を実測に合わせ、
   `:measured-at` を更新
3. **`README.md` の該当記述も直す** —— これが本体。EDN だけ直すと README が嘘のまま残る
4. 何がどちらへ動いたかを commit message に書く

## やらないこと

- **`actor-manifest.test.ts` を走らせようとしない。** `package.json` も vitest も
  無いので走らない。走らせるために依存を足すと、この repo は「install する物が無い」
  という性質を失う。数の drift は `run_tests.cljs` 側が見ている。
- **`.well-known/did.json` を編集して「直った」としない。** live DID document の
  source ではない。ここを変えても配信は変わらない。
- **`MIGRATION-TODO.md` のチェックを、作業せずに埋めない。**
