# AgriFamilie

森を拓いて蓄え、毎年きびしくなる冬を一家で越えていく開拓ゲームです。
ゲームオーバーになると、名前を入れてみんなのランキングに登録できます（Supabase）。

## 1. Supabase に表を作る（最初に1回だけ）

「てくてくまかせ」と同じ Supabase プロジェクトに、このゲーム用の表を新しく作ります。
左メニューの「SQL Editor」を開き、次の内容を貼り付けて「Run」を押します。

```sql
create table public.agrifamilie_scores (
  id bigint generated always as identity primary key,
  name text not null check (char_length(name) between 1 and 12),
  score int not null check (score between 0 and 10000000),
  winters int not null check (winters between 0 and 999),
  year int not null check (year between 1 and 999),
  created_at timestamptz not null default now()
);

alter table public.agrifamilie_scores enable row level security;

create policy "anyone can read" on public.agrifamilie_scores for select to anon using (true);
create policy "anyone can add"  on public.agrifamilie_scores for insert to anon with check (true);

create index agrifamilie_scores_score_idx on public.agrifamilie_scores (score desc);
```

- 誰でも「読む」「追加する」はできますが、書き換えと削除はできません。
- てくてくまかせの `scores` 表とは別の表なので、ランキングは混ざりません。

## 2. GitHub Pages で公開する

1. GitHub で新しいリポジトリを作る（例：`agrifamilie`、Public）。
2. 「Add file」→「Upload files」で `index.html` と `README.md` をアップロードし、「Commit changes」。
3. リポジトリの「Settings」→「Pages」を開く。
4. Source を「Deploy from a branch」、Branch を `main` / `/ (root)` にして「Save」。
5. 1〜2分待つと、公開URL（`https://ユーザー名.github.io/agrifamilie/`）が表示される。

## 3. 動作確認

1. 公開URLを開き、タイトル画面の「ランキング」を押す。「まだ登録がない」と出れば接続できています。
2. 1回遊んでゲームオーバーにし、名前を入れて「登録する」を押す。
3. Supabase の「Table Editor」→ `agrifamilie_scores` に行が入っていれば完成です。

うまくいかないときは、画面に出る番号で原因が分かります。

- 401：鍵が違う
- 404：表の名前が違う（手順1を実行していない）
- 403：行レベルセキュリティのポリシーが足りない

## メモ

- `index.html` に書いてある `sb_publishable_...` の鍵は、公開してよい鍵です。`sb_secret_...` の鍵は絶対にファイルに書かないでください。
- 不適切な名前やおかしなスコアは、「Table Editor」から行を選んで削除できます。
- ブラウザの開発者ツールを使えば、偽のスコアを送れてしまう仕組みです。
- claude.ai 上で公開しているゲームは外部と通信できないため、そちらでは claude.ai 内だけのランキングが表示されます。
