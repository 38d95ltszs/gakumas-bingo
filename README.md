# 学園セトリ予想ビンゴ

セットリスト予想ビンゴゲーム。ID+パスワードでログインし、予想内容・ベット・持ち点はSupabaseに保存されて次回続きから再開できる。

ビルド不要の素のHTML/CSS/JavaScript。GitHub Pagesでそのまま公開できる。

## 1. Supabaseプロジェクトを作る

1. https://supabase.com にログイン(なければアカウント作成)
2. 「New project」でプロジェクトを作成(リージョンは Tokyo (ap-northeast-1) を推奨)
3. 作成後、左メニュー **SQL Editor** を開き、このリポジトリの `supabase-schema.sql` の中身を全部貼り付けて実行
4. 左メニュー **Authentication > Providers > Email** を開き、**「Confirm email」をOFF** にする
   (このアプリはIDを疑似メールアドレスに変換してログインさせるため、実際には届かない確認メールを待つ設定のままだとサインアップ後にログインできなくなる)
5. 左メニュー **Project Settings > API** を開き、以下をメモする
   - **Project URL**(例: `https://abcdefgh.supabase.co`)
   - **anon public key**(長い文字列)

## 2. Supabaseの接続情報を書き込む

`index.html` の中盤にある `<script type="module">` 内、以下の部分を書き換える。

```js
// ▼▼▼ ここをSupabaseプロジェクトの値に書き換えてください ▼▼▼
const SUPABASE_URL = "あなたのプロジェクトURL";
const SUPABASE_ANON_KEY = "あなたのanon key";
const FAKE_EMAIL_DOMAIN = "gakumas-bingo.local";
// ▲▲▲ ここまで ▲▲▲
```

> **anon keyはpublicで問題ない値です**(Supabaseの設計上、フロントエンドに埋め込む前提のキー)。
> データの保護は `supabase-schema.sql` で設定した RLS(Row Level Security)ポリシーが担っている
> ( = 自分の行しか読み書きできない)。GitHubにそのままコミットして問題ない。

## 3. GitHubにアップロードする

既存のリポジトリを使う場合:

```bash
cd gakumas-bingo
git init
git remote add origin https://github.com/38d95ltszs/gakumas-bingo.git
git add .
git commit -m "学園セトリ予想ビンゴを追加"
git branch -M main
git push -u origin main
```

すでにリポジトリがある場合は `git remote add` を省略し、既存リポジトリのディレクトリにこれらのファイルをコピーしてから `git add / commit / push` すればよい。

## 4. GitHub Pagesを有効化する

1. GitHubのリポジトリページ > **Settings > Pages**
2. **Source** を `Deploy from a branch` にし、ブランチを `main` / フォルダを `/(root)`(またはこれらのファイルを置いたフォルダ)に設定
3. 数分後、`https://38d95ltszs.github.io/gakumas-bingo/` で公開される

## 5. Supabase側にPagesのURLを登録

Supabase > Authentication > URL Configuration で、上記のGitHub PagesのURLを **Site URL** に追加しておく(認証まわりのリダイレクトで使われる場合に備えて)。

## ファイル構成

```
index.html          画面・見た目・ロジックすべてを含む単一ファイル
                     (★中盤のSupabase接続情報だけ書き換える)
supabase-schema.sql  Supabaseに流し込むテーブル定義・RLS設定
```

## 動作の仕組みメモ

- ID+パスワードは内部で `ID@gakumas-bingo.local` という疑似メールアドレスに変換してSupabase Authに渡している(実在ドメインではないので確認メールは届かない → Confirm emailをOFFにする必要がある)
- ボード内容・ベット配分・担当アイドル・持ち点は `game_saves` テーブルに1ユーザー1行で保存される
- 「判定する」を押すと、その時点の持ち点をもとに演算し、結果を新しい持ち点として保存する
- 「新しいラウンドを始める」は、持ち点はそのままにボード・ベット・正解セトリだけをリセットする(次の公演の予想を始めるときに使う)
- 楽曲リスト・担当アイドルはすべてテスト用のダミーデータ。実際のセトリ候補曲・歌唱メンバーに差し替える場合は `index.html` 内の `SONG_POOL` / `IDOL_POOL` を編集する
