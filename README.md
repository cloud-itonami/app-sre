# app-sre

**`sre.etzhayyim.com` —— etzhayyim の各 App を定期的に叩いて死活と smoke を測り、
問題を Matrix の issue room に起票する SRE 監視系の、設計文書とクライアント側
スカフォールドの保管物である。** 名前は Site Reliability Engineering の略で、
それ以外の手掛かりを名前は持たない。この節が名乗りである。

**保管されている 38 ファイル / 66,613 バイトは出所とバイト単位で同一で、そのことは
暗号学的に検査できる**（§5）。出所は `etzhayyim/root@1316716c:60-apps/etzhayyim-project-sre`。

読む前に知っておくべきことが 2 つある。

1. **`CLAUDE.md` と `PROJECT.jsonld` が説明しているサーバ側は、この repo に 1 行も無い。**
   監視ロジック（`health_check_all` 等 9 method）も Go の MCP backend も
   ランディングページも、ここには入っていない（§3）。
2. **クライアント側は、在るが**、`vite build` **が通らない。** 保管対象の中に
   独立した破れが 2 つある —— 解決できない import 1 件と、**本体が無い関数** 1 件。
   どちらも実測済みで、直せば通ることも実測した（§4）。

---

## 1. 在庫

`git ls-files` は 43 ファイル（2026-08-18 実測、この文書と `docs/` の 2 本を含む）。
3 層に分かれる。

| 層 | 数 | バイト | 何か |
|---|---|---|---|
| **保管対象**（出所からそのまま） | **38** | **66,613** | 下表のすべて |
| 抽出時の生成レコード | 2 | 可変 | `README.edn` / `migration.edn` |
| 後から足した文書（保管対象ではない） | 3 | 可変 | この `README.md` と `docs/` の 2 本 |

**意味のある数は第 1 層の 38 / 66,613 だけ**である。第 2 層と第 3 層はこの文書を
書き換えるたびに動く —— repo 全体のバイト総数をここに書かないのはそのためで、書いた
瞬間にこの文書自身がその数に含まれ、自己言及で必ず陳腐化する。

38 / 66,613 は `migration.edn` の `:source` に記録された値と一致する。**この一致は
検査できる**（§5）。

保管対象 38 ファイルの内訳:

| 区画 | 数 | バイト | 何か |
|---|---|---|---|
| `runner/` | 9 | 32,049 | Playwright smoke ランナー（+ 死んだデバッグ script 2 本 = 14,185 B） |
| 根の文書 | 4 | 13,379 | `CLAUDE.md` `DESIGN_SRE_TOOLBAR_MCP_INTEGRATION.md` `NOTICE` `PROJECT.jsonld` |
| `extension/` | 20 | 11,163 | Chrome MV3 拡張（Svelte 5 + Vite） |
| `shared/sre-toolbar-ui/` | 4 | 8,775 | 拡張が `@sre-shared` として取り込む共有 UI + MCP client |
| `appview/` | 1 | 1,247 | App 移行の配置メモ（README のみ） |

最大のファイルは `runner/test-game-mode.mjs`（13,568 B）で、これは SRE とは無関係の
一回きりのデバッグ script である（§3）。

---

## 2. 何が在るか

### `extension/` —— Chrome MV3 の SRE ツールバー

`etzhayyim.com` 配下のページに overlay を差し込み、フィードバックを MCP 経由で
送るためのブラウザ拡張。Vite が 4 つの entry（popup / background / toolbar / inject）を
束ね、`shared/sre-toolbar-ui/src` を `@sre-shared` alias で取り込む。

### `runner/` —— Playwright smoke ランナー

`run.ts` が `sre.etzhayyim.com` の SpinApp レジストリを引き、`playwrightEnabled` な
App ごとに `e2e/smoke.spec.ts` の 5 検査（200 応答 / `h1` の可視 / `/health` 200 /
meta description / JS console error 0 件）を回して結果を戻す。Docker 前提
（`mcr.microsoft.com/playwright:v1.50.0-jammy`）。

### `shared/sre-toolbar-ui/` —— 共有 UI と MCP client

7,743 B の Svelte コンポーネントと、`sreToolbar.submitFeedback` を呼ぶ薄い層。

---

## 3. 何が無いか（文書が在ると言っているもの）

**これらは「未実装」ではなく「この repo には入っていない」である。**出所の
`etzhayyim/root` 側に在るのかどうかは、ここからは分からない。

| 文書 | 在ると言っているもの | 実際 |
|---|---|---|
| `CLAUDE.md` | `wasm/etzhayyim-wasm-sre-job-srej0b1x/` — 9 method の SRE App 本体 | ディレクトリごと無い |
| `PROJECT.jsonld` | `app/` — Go の MCP backend（port 8080） | 無い |
| `PROJECT.jsonld` | `ui/` — `sre.etzhayyim.com` のランディングページ（SvelteKit SSG） | 無い |
| `NOTICE` | `CHARTER-RIDER.md` | 無い |
| `extension/buf.gen.yaml` | `src/lib/gen` へ protobuf/Connect を生成 | 生成物も `.proto` も `buf.yaml` も無い |

したがって **`CLAUDE.md` の Architecture 図・method 表・Arrow テーブル表は、この repo に
対する説明ではない。** 出所プロジェクト全体に対する説明である。

**参照先ホストは解決しない**（2026-08-18 実測、`dig`）:

| ホスト | A レコード |
|---|---|
| `sre.etzhayyim.com` | NXDOMAIN |
| `matrix.etzhayyim.com` | NXDOMAIN |
| `news.etzhayyim.com` | NXDOMAIN |
| `color-by-number.etzhayyim.com` | NXDOMAIN |
| `ex66satl.etzhayyim.com` | NXDOMAIN |
| `etzhayyim.com` | 172.67.179.128 / 104.21.51.111（Cloudflare） |

`runner/run.ts` は起動して 1 行印字したのち `ENOTFOUND sre.etzhayyim.com` で exit 1 になる
（§4 で実行した）。**レジストリが引けないので、このランナーは現状どの App も測れない。**

`runner/` の 2 本は SRE とも無関係な、名指しの App に対する一回きりのデバッグ script:

- `test-game-mode.mjs`（13,568 B）—— `color-by-number` と snake を叩く。`package.json` の
  どの script からも呼ばれない
- `news-debug.spec.ts`（617 B）—— `news.etzhayyim.com` のスクリーンショットを撮る。
  `playwright.config.ts` の `testDir` は `./e2e` なので `npx playwright test` は**これを拾わない**

合わせて 14,185 B、`runner/` の 44% が到達不能なコードである。

---

## 4. 動くもの・動かないもの（実測）

すべて 2026-08-18 に、clean な展開物に対して実行した。手順と全出力は
[`docs/operator-quickstart.md`](docs/operator-quickstart.md)。

| 何を | 結果 |
|---|---|
| `extension` の `npm install` | **exit 1** — `ERESOLVE`。`--legacy-peer-deps` を付ければ exit 0 |
| `extension` の `npm test`（vitest） | exit 0 — 1 passed。ただし中身は `expect(true).toBe(true)` |
| `extension` の `npm run build`（vite） | **exit 1** — 下記 ① |
| ① を直して再ビルド | **exit 1** — 下記 ② |
| ①② を直して再ビルド | **exit 0** — 141 modules、6 ファイル出力 |
| `runner` の `npm ci --ignore-scripts` | exit 0 — 24 packages |
| `runner` の `npx ts-node run.ts` | **exit 1** — `ENOTFOUND sre.etzhayyim.com` |
| `runner` の `npx playwright test`（`TARGET_URL=https://etzhayyim.com`） | exit 1 — 5 中 4 が「browser が無い」で落ち、`/health` の 1 件だけ通る |

### ① 解決できない import

`shared/sre-toolbar-ui/src/index.ts:1` は
`./components/AietzhayyimProjectSreToolbar.svelte` を読む。
ディスク上の名前は **`EtzhayyimProjectSreToolbar.svelte`**（先頭に `Ai` が無い）。

```
Could not resolve "./components/AietzhayyimProjectSreToolbar.svelte"
  from "../shared/sre-toolbar-ui/src/index.ts"
```

`extension/src/lib/index.ts:2` も同じ綴りの alias を export しており、機械的な
リネームの取りこぼしに見える。

### ② 本体の無い関数

`shared/sre-toolbar-ui/src/mcp/client.ts` は 273 バイト・12 行で、
`callMcpTool` の**シグネチャを宣言したところでファイルが終わっている**。

```ts
export async function callMcpTool<T = unknown>(params: {
  endpoint: string;
  toolName: string;
  arguments: Record<string, unknown>;
  authToken?: string;
}): Promise<T> {        // ← ここで EOF
```

```
[vite:esbuild] Transform failed with 1 error:
  .../shared/sre-toolbar-ui/src/mcp/client.ts:13:0: ERROR: Unexpected end of file
```

**この 1 関数がツールバーの MCP 経路の全部である** —— `mcp/toolbar.ts` の
`submitFeedbackViaMcp` はこれを呼ぶだけで、`SreToolbar.svelte` はそれを呼ぶだけ。
つまり**フィードバック送信は、実装が存在しない。**

### なぜ直さないのか

**①② はどちらも保管対象のファイルの中に在る。**直せば §5 の保管検査が落ちる
（それが検査の目的である）。この repo の役目は出所を bit 単位で保つことなので、
ここでは**可視化に留める**。直すのは出所 `etzhayyim/root` 側の仕事で、その後に
新しい `migration.edn` で取り込み直すのが筋。

同じ理由で、以下も見つけたが直していない:

- `extension/manifest.json` のキーが camelCase（`manifestVersion` `contentScripts`
  `runAt` `serviceWorker` `defaultPopup` `webAccessibleResources`）。Chrome MV3 は
  snake_case を要求するので、**このままでは拡張として読み込めない**
- 同ファイルが `content/toolbar.css` を宣言しているが、ビルドは CSS を 1 つも出さない
- `extension/icons/` の 16 / 48 / 128 は**同一ファイル**（sha256 `07c94aef…`、68 B の
  1×1 グレースケール PNG）
- `runner/package.json` は `@playwright/test@^1.50.0` を指すが lockfile は 1.58.2 を
  固定する。`Dockerfile` は `v1.50.0-jammy` を基底にしつつ `package-lock.json` を
  COPY しないので、イメージ内では毎回別解決になる

---

## 5. 保管を検査する

```bash
nbb docs/verify-custody.cljs            # ローカルのみ（network 不要）
nbb docs/verify-custody.cljs --origin   # 出所 GitHub の実 tree とも突き合わせる
```

`migration.edn` の `:identity :allowed-additions` に挙がった名前を除いた残りから
git tree を再構成し、`:source :tree` のハッシュと突き合わせる。ファイル数とバイト数も
見る。**バイト数だけでは足し引きが相殺する改変を通してしまう**ので、錨はハッシュ。

exit は 0 = PASS / 1 = FAIL / **3 = 判定できなかった**。3 が独立しているのは、
git が無い・repo でない・記録が読めない・対象が 0 件、といった「測れなかった」場合を
**PASS と同じ値で返さない**ためである。

---

## 6. この repo に何をしてよいか

- **保管対象 38 ファイルを編集しない。**編集した瞬間、この repo は出所の証拠では
  なくなる。§4 の欠陥は出所側で直す
- 足すのは文書と検査だけ。足したら `migration.edn` の `:allowed-additions` にも
  足す（さもないと保管検査が落ちる）。`:source` は 1 バイトも触らない
- ここを起点に SRE を再建するなら、まず出所 `etzhayyim/root` に
  `wasm/etzhayyim-wasm-sre-job-srej0b1x/` が実在するかを確かめる。**この repo に
  在るのはクライアント側だけで、監視そのものは入っていない**
