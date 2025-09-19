## Summary
- 変更点：

## Related Issue
- Closes #<issue-number>（クローズしない場合は Refs に変更）

## Score
- Before: <score> (pass=true)
- After:  <score> (pass=true)
- 差分: +<delta>

## Why（根拠/仮説）
- 例）N+1によりDB待ちが支配的 → JOIN化＋Index活用で削減

## Changes
- ソース: <path/to/file>
- DB（あれば）: `CREATE INDEX ...`
- ミドル（あれば）: <nginx/my.cnf等の差分要約>

## Validation
- 再ベンチ: 同一環境/並列度で実行
- EXPLAIN/ログで効果確認

## Risk & Rollback
- 影響範囲：
- 問題時は `<commit-hash>` に戻す

## Checklist
- [ ] webapp/ で git 管理（初期スナップショットあり）
- [ ] Before/After のスコアを記載
- [ ] 変更は小さく・再現可能な手順を添付
