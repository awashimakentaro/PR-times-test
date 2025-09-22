### 課題解決のために行ったこと

## 環境構築
- 実装: `webapp/python`
- 起動: Docker Compose（`app.build.context: python/`）
- 初期化:
```bash
make init
cd webapp
docker compose up -d
```
- アクセス: http://localhost

## ベンチマーク実行と保存
```bash
cd benchmarker
docker build -t private-isu-benchmarker .
mkdir -p bench_results
docker run --network host -i private-isu-benchmarker /bin/benchmarker -t http://host.docker.internal -u /opt/userdata | tee bench_results/$(date +%Y%m%d_%H%M%S).json
```

## 変更内容
#1 fix/sourcecode
  アプリのパスワードハッシュ計算を高速化しました。

  - 変更ファイル: `webapp/python/app.py`
  - 変更内容: `digest()` が外部コマンド（`openssl`）ではなく Python 標準の `hashlib.sha512` を使用するように変更
  - 効果: ログイン/登録時のハッシュ計算でサブプロセス起動コストを削減し、ベンチ実行時のスコア改善が期待できます（機能仕様 は不変）
#2 fix/score
  - users 認証の最適化
    - `CREATE INDEX idx_users_account_del ON isuconp.users (account_name, del_flg);`
  - comments 取得の最適化
    - `CREATE INDEX idx_comments_post_created ON isuconp.comments (post_id, created_at);`
  - 適用（Compose 上の MySQL へ実行）
```bash
cd webapp
docker compose exec -T mysql mysql -uroot -proot -e "CREATE INDEX idx_users_account_del ON isuconp.users (account_name, del_flg);"
docker compose exec -T mysql mysql -uroot -proot -e "CREATE INDEX idx_comments_post_created ON isuconp.comments (post_id, created_at);"
```

## スコア
- Before: 1688
- After: 14314
  
## リスクとロールバック
- 影響範囲:
  - def make_posts(results, all_comments=False):
  - def try_login(account_name, password):
- ロールバック:
```bash
docker compose exec -T mysql mysql -uroot -proot -e "DROP INDEX idx_users_account_del ON isuconp.users;"
docker compose exec -T mysql mysql -uroot -proot -e "DROP INDEX idx_comments_post_created ON isuconp.comments;"
```
- ソースは該当コミットへ戻す

## 総括
- 小さな変更（外部プロセス除去）と安全なインデックス追加だけでも、DB待ちとアプリCPUオーバーヘッドが減り、スコアが底上げできた。
- 今回の課題でsqlの動きや仕組みを少しだけ理解することができたがまだまだ理解が浅いので、
自分でも開発の中で積極的に使用して使えるようにしていきたいと思いました。

