# operator quickstart — app-sre

**この手順は 2026-08-18 に上から順に実走した。**掲載している数字・エラー本文・
exit code は、すべてその実行が実際に印字したものである。予想を書いていない。

環境: macOS 26.3 / arm64、node v26.3.0、npm 11.16.0、nbb 1.4.210、gh 認証済み。

所要は §1–§3 と §6 で 2 分ほど。§4 は依存の取得を含むので 2〜4 分。

---

## 0. 何をする repo か 30 秒で

`README.md` の §1 と §3 を読む。要点だけ先に:

- **これは保管物である。**出所は `etzhayyim/root@1316716c:60-apps/etzhayyim-project-sre`
- **サーバ側（監視ロジック・Go backend・ランディング）はこの repo に入っていない**
- **クライアント側は入っているが、ビルドが通らない**（§4）
- 参照先の `sre.etzhayyim.com` は **NXDOMAIN**（§5）

---

## 1. 取得

```bash
git clone https://github.com/cloud-itonami/app-sre
cd app-sre
```

## 2. 在庫を見る

```bash
git ls-files | wc -l
```

```
      43
```

43 = 保管対象 38 + 抽出レコード 2（`README.edn` `migration.edn`）+ 後から足した文書 3
（`README.md` と `docs/` の 2 本）。

保管対象だけを数え直す:

```bash
git ls-files | grep -v -E '^(README\.edn|migration\.edn|README\.md|docs/)' | wc -l
git ls-files | grep -v -E '^(README\.edn|migration\.edn|README\.md|docs/)' \
  | while read f; do wc -c < "$f"; done | awk '{s+=$1} END{print s}'
```

```
      38
66613
```

`migration.edn` の `:source {:tracked-files 38 :bytes 66613}` と一致する。

## 3. 保管を検査する（network 不要、数秒）

```bash
nbb docs/verify-custody.cljs
```

```
SCANNED	38 保管ファイル / 3 検査
  ok   出所 tree（再構成 vs 記録）
         got  eb81e706783d12ada7937f8c73466b72c200813c
  ok   保管ファイル数
         got  38
  ok   保管バイト数
         got  66613
PASS — 保管対象 38 ファイルは出所と同一（--origin を付けると出所 GitHub とも突き合わせる）
```

出所 GitHub の実 tree とも突き合わせる（`gh` の認証が要る）:

```bash
nbb docs/verify-custody.cljs --origin
```

```
SCANNED	38 保管ファイル / 4 検査
  ok   出所 tree（再構成 vs 記録）
         got  eb81e706783d12ada7937f8c73466b72c200813c
  ok   保管ファイル数
         got  38
  ok   保管バイト数
         got  66613
  ok   出所 GitHub の実 tree（etzhayyim/root@1316716c:60-apps/etzhayyim-project-sre）
         got  eb81e706783d12ada7937f8c73466b72c200813c
PASS — 保管対象 38 ファイルは出所と同一
```

exit 0 = PASS / 1 = FAIL / **3 = 判定できなかった**（git が無い・repo でない・
記録が読めない・対象 0 件）。**3 を PASS と混同しないこと。**

## 4. 拡張をビルドしてみる（ここで壊れる）

```bash
cd extension
npm install --no-audit --no-fund
```

**exit 1。**

```
npm error code ERESOLVE
npm error Found: vite@6.4.3
npm error Could not resolve dependency:
npm error peer vite@"^5.0.0" from @sveltejs/vite-plugin-svelte@4.0.4
```

`package.json` が `vite@^6.0.0` を要求する一方、`@sveltejs/vite-plugin-svelte@^4.0.0`
（4.0.4 に解決）は `vite@^5.0.0` を peer に持つ。回避:

```bash
npm install --no-audit --no-fund --legacy-peer-deps    # exit 0
```

テストは通る:

```bash
npm test
```

```
 Test Files  1 passed (1)
      Tests  1 passed (1)
```

**ただしこの 1 件は `expect(true).toBe(true)` である**（`extension/test/sre.test.ts`、
154 B）。緑だが何も主張していない。

ビルド:

```bash
npm run build
```

> com-junkawasaki のワークスペース内から回す場合は、並行セッションと競合するので
> 共有 build ロックを経由すること —— `node <superproject>/scripts/resource-guard.mjs
> run build -- npm run build`。単独の clone なら素の `npm run build` でよい
> （下の出力はロック経由で取ったが、内容は同じ）。

**exit 1。**

```
✗ Build failed
error during build:
Could not resolve "./components/AietzhayyimProjectSreToolbar.svelte"
  from "../shared/sre-toolbar-ui/src/index.ts"
```

ディスク上の名前は `EtzhayyimProjectSreToolbar.svelte`（先頭の `Ai` が無い）。

**この先を見たい場合は、clone の外に展開したコピーで試すこと**（repo 内のファイルを
編集すると §3 の保管検査が落ちる。それが検査の目的である）:

```bash
git archive HEAD | tar -x -C /tmp/sre-scratch
```

`/tmp/sre-scratch` 側で import 名を直して再ビルドすると、**別の場所で落ちる**:

```
[vite:esbuild] Transform failed with 1 error:
  .../shared/sre-toolbar-ui/src/mcp/client.ts:13:0: ERROR: Unexpected end of file
```

`client.ts` は `callMcpTool` のシグネチャを宣言したところで終わっている（273 B / 12 行）。
本体が無い。これがツールバーの MCP 経路の全部である。

`callMcpTool` に仮の本体を足すと、そこで初めて通る:

```
dist/popup.html
dist/background.js
dist/content/inject.js
dist/popup.js
dist/content/toolbar.js
dist/chunks/custom-element-<hash>.js
```

exit 0。**つまり保管対象への 2 箇所の編集が、ビルドが通る状態との差の全部である。**

> ここでバイト数と module 数を引用していないのは、**再現しないからである。**
> `extension/` に lockfile が無いので、数分あけた 2 回のインストールで module 数
> （130 / 131）と `dist/content/toolbar.js` のサイズ（13.68 / 13.62 kB）が動いた。
> 出力される**ファイルの集合**は 2 回とも同じだった。

`dist/` に **CSS は 1 つも無い**が、`manifest.json` は `content/toolbar.css` を
宣言している。加えて `manifest.json` のキーは camelCase なので、この `dist/` を
Chrome に読み込ませることはできない（MV3 は snake_case を要求する）。

**終わったら scratch を捨てる。**`/tmp/sre-scratch` の編集を clone 側へ持ち帰らない。

## 5. ランナーを動かしてみる（レジストリが無い）

```bash
cd runner
npm ci --ignore-scripts --no-audit --no-fund     # exit 0, 24 packages
npx ts-node run.ts
```

**exit 1。**

```
SRE Playwright Runner — fetching SpinApp list from https://sre.etzhayyim.com
[TypeError: fetch failed] {
  [cause]: Error: getaddrinfo ENOTFOUND sre.etzhayyim.com
```

```bash
dig +short sre.etzhayyim.com A     # 出力なし（NXDOMAIN）
```

レジストリが引けないので、**このランナーは現状どの App も測れない。**

smoke suite 自体は任意の URL に対して直接回せる:

```bash
TARGET_URL=https://etzhayyim.com npx playwright test
```

```
  ✘  1 › page loads and returns 200
  ✘  3 › h1 is visible
  ✓  5 › /health endpoint returns 200
  ✘  6 › meta description is present
  ✘  8 › no JS console errors on load
  4 failed
  1 passed
```

落ちた 4 件はすべて同じ原因で、対象サイトの問題ではない:

```
Error: browserType.launch: Executable doesn't exist at
  .../ms-playwright/chromium_headless_shell-1208/...
```

`--ignore-scripts` で入れたのでブラウザが無い。通った 1 件は `request` を使う
（ブラウザ不要な）検査で、`https://etzhayyim.com/health` は実際に 200 を返す。

ブラウザまで入れるなら:

```bash
npx playwright install chromium        # 約 150 MB のダウンロード
```

## 6. 検査が本当に落ちることを見る（1 分）

**落ちない検査は劇場である。**この検査器が実際に差を捕まえること、そして
「測れなかった」を PASS と同じ値で返さないことを、一度自分で見ておく。
以下 4 つはすべて実走した。

### 保管対象に 1 バイト足す

```bash
printf '\n' >> NOTICE
nbb docs/verify-custody.cljs; echo "exit=$?"
```

```
  ok   出所 tree（再構成 vs 記録）
  ok   保管ファイル数
  FAIL 保管バイト数
         got  66614
         want 66613
FAIL — 保管対象が出所と一致しない
exit=1
```

tree が `ok` のままなのは仕様である —— **tree の検査は HEAD を、バイト数の検査は
working tree を見る。**未 commit の改変はバイト数側で捕まる。

```bash
git checkout -- NOTICE      # 戻すと exit=0
```

### 保管対象を 1 件消して commit する

```bash
git rm -q appview/README.md && git commit -q -m TEMP
nbb docs/verify-custody.cljs; echo "exit=$?"
```

今度は 3 つとも落ちる（ハッシュが動く）:

```
  FAIL 出所 tree（再構成 vs 記録）
         got  844a68596b67ddd5f89ab5cbd86087ebbdc6a47a
         want eb81e706783d12ada7937f8c73466b72c200813c
  FAIL 保管ファイル数
         got  37
         want 38
  FAIL 保管バイト数
         got  65366
         want 66613
exit=1
```

```bash
git reset -q --hard HEAD~1  # 戻すと exit=0
```

### 保管対象を `:allowed-additions` に紛れ込ませて迂回しようとする

`migration.edn` の `:allowed-additions` に `"shared"` を足す —— つまり
「`shared/` は追加物だから検査しなくてよい」と主張してみる。

```
SCANNED	34 保管ファイル / 3 検査
  FAIL 出所 tree（再構成 vs 記録）
         got  3d33e48d876846cc8280a08b60cd3073c611990b
         want eb81e706783d12ada7937f8c73466b72c200813c
  FAIL 保管ファイル数
         got  34
         want 38
exit=1
```

**除外した分だけ再構成 tree から消えるので、迂回はハッシュを合わなくする方向にしか
働かない。**検査対象を減らして通す、ができない。

### 判定できない場合（PASS でも FAIL でもない）

repo の外から呼ぶ:

```bash
cd /tmp && nbb /path/to/app-sre/docs/verify-custody.cljs; echo "exit=$?"
```

```
UNDETERMINED: migration.edn が無い。この repo のルートで実行すること
exit=3
```

**exit 3 は 0 でも 1 でもない。**git が無い・repo でない・記録が読めない・対象が
0 件、はすべてここに落ちる。CI で拾うときは `exit != 0` を fail とし、3 を
「合格」に丸めないこと。

---

## 次に読むもの

| 知りたいこと | 見る場所 |
|---|---|
| 在庫と、文書が在ると言っていて実際は無いもの | `README.md` §1・§3 |
| ビルドが壊れている 2 箇所の詳細 | `README.md` §4 |
| 出所プロジェクト全体の設計（この repo の説明ではない） | `CLAUDE.md` / `DESIGN_SRE_TOOLBAR_MCP_INTEGRATION.md` |
| ツールバーの MCP 設計 | `DESIGN_SRE_TOOLBAR_MCP_INTEGRATION.md` |
| Matrix issue room の運用と必須 env | `appview/README.md` |
| 抽出の出所・リビジョン・許可された追加物 | `migration.edn` |
