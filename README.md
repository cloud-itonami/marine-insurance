# marine-insurance

**海上保険 actor の *descriptor* と、その書き込みを止める *deny-by-default gate*。
保険の引受・査定・支払いそのものは、ここには無い。実装ではない。**

`did:web:marine-insurance.etzhayyim.com`（名乗り・**DNS に存在しない**）·
`did:web:etzhayyim.com:actor:marine-insurance`（解決する方・**この repo の substrate も
manifest も名乗っていない**）

`marine-insurance` という名前は主題（海上保険）を言うが、**この repo が何であるか**は
言わない。読み始める前に、この 2 つを分けて持ってほしい:

| | ここにあるか |
|---|---|
| actor が**何を名乗り、何を要求し、どの pipeline を持つと宣言しているか** | **ある**（`actor-manifest.jsonld` / `.well-known/did.json`） |
| **gate**（attestation が揃わなければ effect を 1 つも出さない判断） | **ある**（`src/marine_insurance/murakumo.cljk`、240 行） |
| Hull & Machinery / P&I / Cargo / War Risk / General Average の実処理、graph、XRPC の実行主体 | **無い** |

**ここには動くサービスは無い。** `cell-plan` が返すのは「書くとしたら何をどこに書くか」
という**計画**であって、書き込みそのものではない。`:effects` は
`{:op :mst/put-record ...}` という data であり、それを実行する者はこの repo に居ない。

## 2026-05-21 の snapshot であること

etzhayyim monorepo の `20-actors/marine-insurance` から descriptor だけを写した
snapshot で、codemod は未着手（`MIGRATION-TODO.md` の 6 項目は全部 `[ ]` のまま。
この数は test で固定してある）。`actor-manifest.jsonld` が `runtime: k8s-langserver` /
`edge: sveltekit-proxy` と宣言していても、**その runtime も edge もここには無い。**

経緯は [docs/adr/0001-descriptor-snapshot-not-an-executor.md](docs/adr/0001-descriptor-snapshot-not-an-executor.md)。

## 確かめる

散文ではなく実行で確かめられる。

```bash
nbb --classpath src:test run_tests.cljk             # 構造・gate・固定値（network 不要）
nbb --classpath src:test run_tests.cljk --network   # 上記 + 名乗りを実際に解決しに行く
```

最後に `marine-insurance actor: all green` が出れば緑。手順は
[docs/operator-quickstart.md](docs/operator-quickstart.md)。

**この README 自身も検査対象である。** 下に書いてある数（pipeline 10 / cell 18 /
gate 7）と 2 つの DID は `test/marine_insurance/docs_test.cljk` が実体と突き合わせるので、
実体が動けば README が赤くなる。quickstart が名指しする `.cljs` の実在も同じ場所で
守っている —— **踏めない手順を書けない**ようにするため。

## ここにあるもの

| ファイル | 役割 |
|---|---|
| `src/marine_insurance/murakumo.cljk` | **この repo で唯一 substrate と呼べるもの。** 18 cell × 7 gate の deny-by-default 判断 |
| `actor-manifest.jsonld` | actor 宣言。10 pipeline（cron 2 / subscribeRepos 1 / xrpc 7）、6 sub-actor、5 capability |
| `.well-known/did.json` | DID document。**配信されていない**（Pages 404）し、live 文書とも中身が違う |
| `docs/identity-claims.edn` | 下の表の**実測値を固定したもの**。test の期待値 |
| `test/` | gate（緩む方向 / きつくなる方向の両方）・descriptor 本体・文書・network 実測 |
| `run_tests.cljk` | 上記の runner。nbb + `cljs.test` |
| `actor-manifest.test.ts` | **走らない**（`package.json` も vitest も無い）。下記 |

## gate は何を止めるのか

`cell-plan` は 18 の cell それぞれについて、7 本の attestation が**全部**揃って
いなければ `:status :blocked` / `:effects []` を返す。

```
:council-charter-attestation      :no-platform-held-key-baseline
:no-probing-baseline              :murakumo-only-inference-baseline
:did-primary-baseline             :append-only-gate-baseline
:kotoba-only-substrate-baseline
```

7 本は AND であり、`false` と明示された attestation は「未 attest」より強い否定として
扱われる（キーが在るだけでは通らない）。attestation は 4 つの形（keyword の set /
keyword map / string map / string の set）すべてを受ける —— どれか 1 つが壊れると、
**その形で attest している呼び出し側だけが黙って全部 blocked** になる。安全側に
倒れるので事故に見えない。だから test は緩む方向ときつくなる方向の両方を押している。

## 測ったが直していないこと

`docs/identity-claims.edn` の `:gaps` が正本。**塞ぐと test が赤くなる** —— それは
故障ではなく「固定値を測り直せ」の合図である。

| gap | 実測（2026-08-09） |
|---|---|
| **名乗りが 2 つに割れている** | manifest の `@id` と `.well-known/did.json` の `id` が別物。前者は **DNS に無い**（`marine-insurance.etzhayyim.com` が引けない）。後者は 200 |
| **substrate も解決しない方を名乗る** | `murakumo.cljc` の `actor-did` は manifest と同じ「解決しない方」。effect は全部その DID に帰属する |
| **手元の `did.json` は配信文書ではない** | 配信側とは `@context` の suite・`alsoKnownAs`（手元 4 件 / 配信 0 件）・PDS endpoint・service の顔ぶれ・`_meta` の有無が違う。**ここを編集しても配信は変わらない** |
| **Pages の鏡は 404** | `alsoKnownAs` が名乗る `etzhayyim.github.io/com-etzhayyim-marine-insurance` は配信されていない |
| **`agent.invoke` は宣言だけ** | capability 5 個を宣言し、pipeline step が実際に使うのは 4 個。`agent.invoke` はどの step の `fn` にも現れない |
| **`actor-manifest.test.ts` は嘘をついていた** | `pipelines` を 8 と主張。実体は 10。`package.json` が無く vitest も入っていないので**一度も走ったことがない**。数だけ実体に合わせ、drift したら `run_tests.cljk` 側が赤くなるようにした |
| **lexicon が交差しない** | substrate は `com.etzhayyim.marine-insurance.*` に書き、manifest の xrpc は `com.etzhayyim.apps.marineInsurance.*`。交差 **0**。揃える先は **substrate 側** —— 配信されている DID document の `_meta.primaryLexicon` が `com.etzhayyim.marine-insurance` だからである |
| **codemod 未着手** | `MIGRATION-TODO.md` の 6 項目が全部 `[ ]` |

## やらないこと

- **`.well-known/did.json` を編集して「直った」としない。** live DID document の
  source ではない（配信文書と中身が食い違っていることを実測済み）。
- **substrate の collection prefix を manifest 側に合わせない。** 割れが解消したように
  見えて**権威から遠ざかる**（上表の最終行）。
- **`MIGRATION-TODO.md` のチェックを、作業せずに埋めない。** `[ ]` / `[x]` の数は
  固定してあり、埋めれば test が赤くなって「測り直せ」と言う。
