# living-spec-lint

![version](https://img.shields.io/badge/version-0.0.0-blue)
![license](https://img.shields.io/badge/license-MIT-green)
![status](https://img.shields.io/badge/status-concept-yellow)

Living Spec の整合性をチェックする Claude Code プラグイン。ADR と Spec ドキュメント間のステータス不整合、古い参照の残存、バージョン表記のズレを検知します。

## 動機

Living Spec を持つプロジェクトでは、機能の実装・削除・変更に伴ってドキュメントの不整合が蓄積しやすい:

- `[Accepted]` のまま放置された実装済み機能のステータス
- 削除された機能への参照が複数ファイルに残存
- テスト件数やバージョン表記が古いまま

これらは手動のドキュメントスイープで対処可能だが、見落としが発生しやすい。living-spec-lint はこの問題を 3 層の検知ロジックで自動化する。

## 検知レイヤー

| 層 | 検知方法 | コスト |
|---|---|---|
| **Layer 1: 構造チェック** | パターンマッチ（Grep） | 低 |
| **Layer 2: 参照チェック** | クロスリファレンス（Glob+Grep） | 中 |
| **Layer 3: 意味チェック** | AI 判断（Claude、差分ベース） | 高 |

## ステータス

コンセプト段階。設計議事録は [docs/meeting-notes/](docs/meeting-notes/) を参照。

## ライセンス

[MIT](LICENSE)
