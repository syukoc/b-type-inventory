# セットアップ手順 ― 菓子製造の在庫・製造管理アプリ

このアプリは、みんなの端末で同じデータを見られるようにするための設定が1回だけ必要です
設定が終わるまでは、アプリは「この端末だけの保存」で動きます（試しに触る分には問題ありません）

全体の流れは次の3つです

1. Supabase（データを置く場所・無料）を作る
2. config.js に接続情報と暗証番号を書く
3. GitHub Pages に置いて、共有アドレスで開けるようにする

---

## 1. Supabase プロジェクトを作る

この作業は、**アカウントを持つ本人がサインインする場面が1回あります**（プロジェクトの作成とキーの確認）
それ以外の手順（SQLの実行や設定値の記入）は、依頼を受けた側で進められます

1. https://supabase.com を開き、サインインして「New project」を押す
2. プロジェクト名（例：kashi-inventory）と、データベースのパスワードを決めて作成する
   - パスワードは控えておく（このアプリでは使わないが、管理用に必要）
   - 地域（Region）は Tokyo が近い
3. 作成が終わるまで数分待つ
4. 左メニューの「SQL Editor」を開き、下のSQLをそのまま貼り付けて「Run」を押す

```sql
create table if not exists public.kv_state (
  key text primary key,
  value jsonb not null,
  updated_at timestamptz not null default now()
);

alter table public.kv_state enable row level security;

create policy "kv_state anon full access"
  on public.kv_state
  for all
  to anon
  using (true)
  with check (true);
```

このアプリは、アプリ全体のデータを `kv_state` という表の `state-v1` という1行にまとめて保存します
上のポリシー（RLS）は「anon key を知っている人は読み書きできる」というゆるい設定です
- 共有アドレスと anon key は、実質的に事業所内で使う前提のものです
- 在庫データには個人情報を入れない運用にしてください（製品名・数・期限だけ）
- 本格的なログイン機能が必要になったら、ポリシーの見直しが必要です

5. 左メニューの「Project Settings」→「API」を開き、次の2つを控える
   - Project URL（`https://xxxxxxxxxxxx.supabase.co` の形）
   - anon public key（`eyJ...` で始まる長い文字列）

## 2. config.js を書き換える

`config.js` を開き、控えた2つと、暗証番号を書きます

```js
window.APP_CONFIG = {
  SUPABASE_URL: "https://xxxxxxxxxxxx.supabase.co",
  SUPABASE_ANON_KEY: "eyJ...",
  APP_PIN: "1234"
};
```

- `APP_PIN` は、事業所のみんなで共有する暗証番号です。開くときに一度入力すると、そのタブを閉じるまで覚えます
- PINは「知らない人がうっかり開けない」ための**簡易保護**です。パスワードの代わりにはなりません
- PINを使わない場合は `APP_PIN: ""` のままにします

書き換えたら、アプリを開き直して確認します
- 画面の上の「この端末だけの保存中（共有未設定）」の表示が消えていれば、共有が効いています
- 別の端末でも同じアドレスを開き、品目が同じに見えれば成功です

## 3. GitHub Pages に置く（共有アドレスで開けるようにする）

このアプリは、**専用の単独リポジトリ**に入れてください（個人サイトなどの別リポジトリと混ぜないこと）
リポジトリ名は **`b-type-inventory`** を推奨します
将来、法人アカウントへリポジトリごと移すときも、この形がいちばん簡単です（移行手順は `HANDOVER.md` の「アカウント移行の手順」）

1. GitHub にリポジトリ `b-type-inventory` を作る
2. このフォルダの4ファイル（`index.html`、`config.js`、`SETUP.md`、`HANDOVER.md`）をすべてリポジトリに入れる
   - 引き継ぐ人がリポジトリだけ見れば完結するよう、ドキュメントも必ず一緒に入れること
3. リポジトリの「Settings」→「Pages」を開き、公開元（Branch）を `main` / ルートに設定する
4. 数分待つと、`https://（アカウント名）.github.io/b-type-inventory/` のようなアドレスで開けるようになる
5. そのアドレスを、事業所のみんなの端末のホーム画面やブックマークに登録する

コードや設定の中に、特定のGitHubアカウント名は書かれていません
アカウントや公開URLが変わっても、`config.js` の書き換えと Pages 設定の確認だけで済みます

更新のときは、ファイルを新しいものに差し替えてアップロードし直すだけです
（GitHub Pages への反映には数分かかることがあります）

---

## うまくいかないとき

- 「この端末だけの保存中」が消えない
  → config.js の URL とキーが両方入っているか、引用符（"）が閉じているかを確認する
- 開くと「読み込めませんでした」が出る
  → SQLが実行されているか、URLがプロジェクトのものと一致しているかを確認する
- 暗証番号を忘れた
  → config.js の `APP_PIN` を見れば分かります。変えたいときは値を書き換えて再アップロードする

## 補足

- Supabase の無料枠で十分に動きます（データは1行のJSONだけなので、容量はほとんど使いません）
- アプリの仕組みや、機能追加の進め方は `HANDOVER.md` にまとめてあります
