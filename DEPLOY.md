# DEPLOY.md — `simplememofast.github.io` を公開する手順

GitHub Pages の **User Pages** として `https://simplememofast.github.io/` に公開する手順を最短経路でまとめます。
所要時間は GUI ルートで約 10〜15 分、CLI ルートなら 5 分程度です。

---

## 前提

- GitHub アカウント `simplememofast` にログインできること
- このフォルダ（`simplememofast.github.io/` 配下の全ファイル）が手元にあること
- カスタムドメインは **使いません**（理由は最後のセクション参照）

---

## ルート A — Web UI だけで完結（推奨・最も簡単）

### 1. 空のリポジトリを作る

1. <https://github.com/new> を開く
2. Owner: `simplememofast`
3. Repository name: **`simplememofast.github.io`**（アカウント名 + `.github.io` 完全一致が必須）
4. Public を選択
5. "Add a README file" など他のチェックは **すべて OFF**
6. "Create repository" をクリック

### 2. ファイルをアップロード

1. 作ったリポジトリのトップで "uploading an existing file" をクリック
2. このフォルダ内のファイルを **すべてドラッグ&ドロップ**
   ```
   index.html
   about.html
   features.html
   captio-migration.html
   changelog.html
   press.html
   404.html
   sitemap.xml
   robots.txt
   README.md
   DEPLOY.md
   assets/css/style.css
   ```
   ※ `assets/` フォルダごとドラッグ&ドロップで OK。ブラウザがディレクトリ構造を維持してくれます。
3. コミットメッセージ: `Initial commit: simplememofast developer hub`
4. "Commit changes" をクリック

### 3. GitHub Pages を有効化

1. リポジトリの **Settings** タブ → 左メニューの **Pages**
2. "Build and deployment" セクションで:
   - Source: **Deploy from a branch**
   - Branch: **`main`** / **`/ (root)`**
3. "Save" をクリック
4. 1〜2 分待つと、ページ上部に **"Your site is live at https://simplememofast.github.io/"** と表示される

### 4. 動作確認

ブラウザで以下を順にアクセスしてください:

- <https://simplememofast.github.io/> → トップ
- <https://simplememofast.github.io/about.html>
- <https://simplememofast.github.io/features.html>
- <https://simplememofast.github.io/captio-migration.html>
- <https://simplememofast.github.io/changelog.html>
- <https://simplememofast.github.io/press.html>
- <https://simplememofast.github.io/sitemap.xml>
- <https://simplememofast.github.io/robots.txt>
- <https://simplememofast.github.io/존在しないパス> → 404 ページが出れば OK

---

## ルート B — CLI で push（運用更新も楽）

```bash
# 1. このフォルダで初期化
cd simplememofast.github.io
git init -b main

# 2. リモートを設定（事前に空リポジトリを作っておくのは A と同じ）
git remote add origin https://github.com/simplememofast/simplememofast.github.io.git

# 3. コミット
git add .
git commit -m "Initial commit: simplememofast developer hub"

# 4. プッシュ
git push -u origin main

# 5. Pages の有効化は GitHub UI の Settings → Pages から
#    （ルート A の 3 と同じ。CLI から有効化したい場合は gh コマンドを使う）
gh repo edit simplememofast/simplememofast.github.io --enable-issues
gh api -X POST repos/simplememofast/simplememofast.github.io/pages \
  -f source.branch=main -f source.path=/
```

> 以降の更新は `git add . && git commit -m "..." && git push` だけで自動再デプロイされます。

---

## デプロイ後にやること（SEO 配線）

### 1. Google Search Console に登録

1. <https://search.google.com/search-console> を開く
2. プロパティを追加 → URL プレフィックス → `https://simplememofast.github.io/`
3. 所有権確認 → "HTML タグ" 方式を選択
4. 表示された `<meta name="google-site-verification" content="...">` タグを `index.html` の `<head>` 内に追加 → コミット → push
5. Search Console で「確認」
6. Sitemaps メニューから `sitemap.xml` を送信

### 2. Bing Webmaster Tools に登録

1. <https://www.bing.com/webmasters>
2. Search Console と同じ手順で `https://simplememofast.github.io/` を追加
3. サイトマップを送信

### 3. 本体サイトから相互リンク（任意・推奨）

`simplememofast.com` の footer / about ページに

```html
<a rel="me" href="https://simplememofast.github.io/">Developer Hub</a>
```

を 1 本だけ追加すると、`rel="me"` 本人確認チェーンが完成します（IndieAuth 仕様）。
about.html 側からは既に `rel="me"` で `simplememofast.com/about/` を指しているため、相互参照になります。

### 4. Ahrefs / 各種 SEO ツールに登録

- Ahrefs Site Explorer に `simplememofast.github.io` を登録すると、被リンクと内部リンクの両方を追跡できます。
- 反映には数日〜2 週間かかります。焦らないでください。

---

## なぜカスタムドメイン（CNAME）を使わないのか

GitHub Pages にはカスタムドメインを当てる機能があります。たとえば `dev.simplememofast.com` を割り当てることも可能です。
しかし今回は **意図的に `simplememofast.github.io` のまま運用** します。

理由は SEO 戦略上、**`simplememofast.com` とは別ドメインからの被リンク（referring domain）を一つ増やすこと** が目的だからです。
カスタムドメインで `*.simplememofast.com` に紐付けてしまうと、ホスト名は違っても **同一ルートドメイン** として扱われ、
referring domain 数の純増に寄与しません。

`github.io` 自体は DR が極めて高いドメインで、サブドメインからの被リンクでも一定の評価が期待できます。
（ただし `*.github.io` は厳密にはサブドメイン単位で評価される側面もあり、効果は他の独立ドメインからのリンクほど強力ではありません — 後段の「不確実な点」参照）

---

## 不確実な点 / 注意事項

- **`github.io` サブドメインの SEO 評価**: Google は `github.io` を **publicly available suffix** として扱い、各サブドメインを独立サイトとして評価する傾向があります。これは追い風ですが、効果の大きさは検証推奨。
- **inデックス反映**: クロールに 1〜4 週間、Ahrefs の被リンクテーブル反映にも同程度かかります。
- **アンカーテキスト**: 全 6 ページで `simplememofast.com` への dofollow リンクのアンカーを意図的に分散しています（ブランド名 / 機能名 / Captio 関連 / 汎用 / 裸 URL）。後で記事を増やす際も、同じ分散を意識してください。
- **重複コンテンツ**: 本体サイト `simplememofast.com` と同じコピーをそのまま転載しないこと。要約・別切り口・開発者視点でのリライトを徹底しています。

---

## ファイル一覧（最終確認用）

```
simplememofast.github.io/
├── index.html
├── about.html
├── features.html
├── captio-migration.html
├── changelog.html
├── press.html
├── 404.html
├── sitemap.xml
├── robots.txt
├── README.md
├── DEPLOY.md
└── assets/
    └── css/
        └── style.css
```

以上です。詰まったら このファイルを再度共有してください。
