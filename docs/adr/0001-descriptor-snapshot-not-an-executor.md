# ADR-0001 — この repo は descriptor + gate であって実行系ではない

- **status**: accepted
- **date**: 2026-08-09
- **参照**: superproject ADR-2608052000（成熟度の測り方）/ ADR-2608080000（1 反復 1 軸）
- **先例**: `cloud-itonami/m365-ingest` ADR-0001（同型の descriptor snapshot）

## 文脈

`marine-insurance` は 2026-05-21 に etzhayyim monorepo の
`20-actors/marine-insurance` から descriptor だけを写した snapshot である。
2026-07-18 の rescue commit（PR #1）で `src/marine_insurance/murakumo.cljk` が
入ったが、**走るテストは 1 本も無く、README も無かった。**

名前（`marine-insurance`）は主題を言うが、この repo が「海上保険を処理する実装」
なのか「宣言だけ」なのかを言わない。実際には後者である。読み手がそれを知る手段が
無い状態で、`actor-manifest.jsonld` は `runtime: k8s-langserver` /
`edge: sveltekit-proxy` / 10 本の pipeline を宣言していた —— **ここに無いものを、
在るかのように読める。**

## 測ってわかったこと（2026-08-09）

1. **`actor-manifest.test.ts` は嘘をついていた。** `expect(m.pipelines).toHaveLength(8)`
   と主張していたが実体は 10。`package.json` が無く vitest も入っていないので
   `npx vitest` は `missing packages` で止まる —— **一度も走ったことがない。**
   走らないテストは、テストが無いより悪い（在ることが検査済みに見える）。
2. **名乗りが 2 つに割れている。** `actor-manifest.jsonld` の `@id` は
   `did:web:marine-insurance.etzhayyim.com` で、そのホストは **DNS に無い**。
   `.well-known/did.json` の `id` は `did:web:etzhayyim.com:actor:marine-insurance` で、
   こちらは 200 で解決する。**substrate（`murakumo.cljc` の `actor-did`）は
   解決しない方を名乗っている** ので、gate が出す effect は全部その DID に帰属する。
3. **手元の `.well-known/did.json` は配信文書ではない。** 同じ DID の配信文書とは
   `@context` の suite（ed25519-2020 / jws-2020）・`alsoKnownAs`（手元 4 件 /
   配信 0 件）・PDS endpoint（`pds.etzhayyim.com` / `pds.aozora.app`）・service の
   顔ぶれ・`_meta` の有無が違う。**ここを編集しても配信は変わらない。**
4. **west の pin が rescue commit の 2 つ手前で止まっていた。** 成熟度 scan が測る
   tree には `src/` が存在せず、`axis-substrate=0` は「コードが無い」ではなく
   **「pin が古くて見えていない」** の意味だった。m365-ingest / jp-ashiba と同じ
   pin 遅れで、**これで 3 例目**。scan の測定値そのものを疑う理由になる。
5. **`agent.invoke` は宣言だけ。** capability 5 個のうち、pipeline step が実際に
   `fn` として使うのは 4 個。

## 決定

1. **この repo を「descriptor + gate」と名乗る。** README の冒頭で、何が在って何が
   無いかを表で示す。`runtime` / `edge` の宣言はここに実体が無いと明記する。
2. **走る検査を持つ。** runner は `run_tests.cljk`（nbb + `cljs.test`）。workspace の
   規則で script host は nbb に一本化されており、新規の `.ts` / `.mjs` / `.sh` は
   禁止なので、**新しい harness を TypeScript では書かない。**
3. **`actor-manifest.test.ts` は削除せず、数だけ実体に合わせる。** そのうえで
   `run_tests.cljk` が「`.ts` が主張する pipeline 数 == 実体」を検査する ——
   **走らないファイルを、走るファイルから縛る。** 二重実装（mirror）にしないため、
   `.ts` の他の assertion は nbb 側に写さない。
4. **測ったが直していないことは `docs/identity-claims.edn` の `:gaps` に固定する。**
   塞ぐと test が赤くなり「測り直せ」と言う。**穴を穴として記録し、記録が飾りに
   ならないようにする。**
5. **文書自身を検査対象にする。** README の数・両 DID・相対リンク・quickstart が
   名指しするファイルの実在を `docs_test.cljs` が突き合わせる。加えて
   **「実行できない手順（`npm test` / `npx vitest` 等）を文書に書けない」** ことを
   test で禁じる。散文は黙って腐るが、腐ったことが赤で出る。

## 却下した案

- **`package.json` + vitest を足して `.ts` を走らせる。** 走るようにはなるが、
  「install する物が何も無い」という性質を失い、node_modules と lockfile を
  抱え込む。workspace の nbb 一本化にも反する。
- **`.ts` を削除する。** 履歴上の意図（どの不変条件を守りたかったか）が消える。
  数を直して縛る方が情報量が多い。
- **`.ts` の assertion を全部 nbb に写す。** 同じ判断の 2 実装は片方だけ直る
  （superproject CLAUDE.md「mirror を作らない」）。nbb 側は `.ts` に無い観点
  （gate の緩み / 文書の腐り / 実測との突き合わせ）を持つ。
- **`.well-known/did.json` を配信文書に合わせて書き換える。** 割れが消えたように
  見えるだけで配信は変わらない。**割れの記録の方が価値がある。**
- **manifest の `@id` を解決する DID に揃える。** 一見「直す」だが、substrate・
  manifest・`.ts` の 3 つが同じ DID を名乗っている現状を片側だけ動かすことになり、
  どこが権威かの判断（`_meta.primaryLexicon` は substrate 側）を先にやる必要がある。
  この反復の scope（成熟度 1 軸 = docs）を超えるので**やらない**。

## 帰結

- operator は `docs/operator-quickstart.md` の 5 手順で、この repo の主張が今も
  本当かを 1〜2 分で確かめられる。
- **緑は「健全」を意味しない。** identity は割れたままで、割れていること自体が
  固定値である。緑は「実測が固定値と一致した」としか言っていない ——
  quickstart にその断りを明記した。
- codemod（`MIGRATION-TODO.md` の 6 項目）は依然として未着手。この ADR はそれを
  進めていない。**進めていないことを、進んだように見せないことがこの ADR の仕事。**
