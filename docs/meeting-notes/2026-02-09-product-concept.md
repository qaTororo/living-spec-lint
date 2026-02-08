# living-spec-lint プロダクトコンセプト議事録

> **日付**: 2026-02-09
> **参加者**: プロジェクトオーナー + Claude
> **起点**: til-capture v1.1 ドキュメントスイープ作業で感じた痛み

## 背景

til-capture v1.0 → v1.1 のドキュメント更新作業で、以下の問題が顕在化した:

1. **ステータス不整合**: F-102/F-107 が `[Accepted]` のまま（実装済みなのに `[Implemented]` に更新されていない）
2. **古い参照の残存**: ADR-004 で削除された `~/til/` フォールバックへの参照が 4 ファイルに残存
3. **テスト件数のズレ**: v1.1 で session-start-hook に 4 テスト追加されたが、Spec のテスト件数は旧値のまま
4. **時制の古さ**: 「v1.0 での変更予定」というセクションが実装完了後も残っている

これらは **Living Spec を持つプロジェクトで共通に発生する問題** であり、自動検知ツールで解決できるのではないかという仮説が生まれた。

## 決定事項

### プロダクト概要

| 項目 | 決定 |
|------|------|
| **名称** | living-spec-lint |
| **形態** | Claude Code Plugin |
| **対象** | Living Spec（ADR + Spec 構造）を持つプロジェクト |
| **メインUI** | 手動 Skill `/spec-lint` |
| **サブUI** | git push hook（オプション、Deny が多いことを想定） |

### 発火タイミング

- **手動実行がメインパス**: `/spec-lint` で明示的にチェック
- **git push hook はオプション**: Atomic commit ではうるさい、Deny 率が高い現実を考慮
- **リリース前に 1 回** が自然なリズム

### 検知対象

全方位（以下の 3 層すべて）:

| 層 | 検知方法 | 例 | コスト |
|---|---|---|---|
| **Layer 1: 構造チェック** | パターンマッチ（Grep） | ステータスラベルの不整合、バージョン表記のズレ | 低 |
| **Layer 2: 参照チェック** | クロスリファレンス（Glob+Grep） | 削除済み機能への参照残存、テスト件数の実値との照合 | 中 |
| **Layer 3: 意味チェック** | AI 判断（Claude）| 時制の古さ、ADR の決定が Spec に反映されているか | 高 |

### token コスト制御

**Layer 1-2 + 差分ベースの Layer 3** を採用:

- Layer 1-2: Grep/Glob ベースで低コスト。常に全ファイル対象
- Layer 3: `git diff --name-only` で変更があったファイルのみ AI に読ませる
- token コストが「ドキュメント総量」ではなく「変更量」に比例する設計

### 出力

- **v0.1**: レポートのみ（不整合箇所の一覧）
- **将来**: 修正提案 → 自動修正と段階的に拡張

### プロジェクト構造の認識

- **config.json で明示的に定義**: Spec ディレクトリ、ADR ディレクトリ、ステータスラベルのルール
- AI による自動推定はしない（明示的な設定を信頼する設計思想）

## config.json スキーマ案

```jsonc
{
  "specDir": "docs/spec",
  "adrDir": "docs/adr",
  "statusLabels": ["Draft", "Proposed", "Accepted", "Implemented", "Out of Scope"],
  "rules": {
    "statusConsistency": true,     // [Accepted] だけど実装済みの機能を検知
    "staleReferences": true,       // 削除済み機能への参照残存を検知
    "versionSync": true            // バージョン表記・テスト件数のズレを検知
  },
  "ignore": ["docs/adr/TEMPLATE.md"]
}
```

## 処理フロー案

```
/spec-lint 実行
  │
  ├─ config.json 読み取り → 構造を把握
  │
  ├─ Layer 1: 構造チェック（低コスト、Grep ベース）
  │   ├─ ステータスラベルの一覧を抽出
  │   ├─ ADR の決定 → 対応 Spec のステータス昇格を検証
  │   └─ バージョン表記の一致を検証
  │
  ├─ Layer 2: 参照チェック（中コスト、Glob+Grep）
  │   ├─ Spec 間のクロスリファレンスを検証
  │   ├─ [Out of Scope] の機能への活性参照を検知
  │   └─ テスト件数の実値との照合
  │
  ├─ Layer 3: 意味チェック（高コスト、Claude 判断、差分ベース）
  │   ├─ git diff で変更ファイルを特定
  │   ├─ 変更ファイルのみを AI に読ませる
  │   ├─ 「v1.0 での変更予定」のような時制の古さを検知
  │   └─ ADR の決定内容と Spec の記述の矛盾を検知
  │
  └─ レポート出力（Markdown 形式）
```

## til-capture からの学び

| 学び | living-spec-lint への適用 |
|------|-------------------------|
| 安価なフィルタで高コスト処理のスコープを絞る | Layer 1-2 → Layer 3 のゲート構造 |
| config.json でユーザーの意図を明示させる | プロジェクト構造を config で定義 |
| 手動 Skill がメインパス | `/spec-lint` で明示的に実行 |
| Dogfooding が最良のフィードバック | til-capture 自身を最初のテスト対象にする |

## 未決事項

- [ ] Layer 1-2 の具体的な検知ルールの詳細設計
- [ ] レポートフォーマットの設計（Markdown? JSON?）
- [ ] til-capture を最初のテスト対象として PoC を実施
- [ ] テスト件数の「実値」をどう取得するか（bats の出力をパース？テストファイルを Grep？）
- [ ] ADR-Spec 間のリンク関係をどう表現するか（config? ファイル内の参照から推定?）

## 次回のアクション

- [ ] Living Spec の構造設計（ADR + Spec テンプレート）
- [ ] config.json スキーマの確定
- [ ] 最小限の Skill PoC（Layer 1 のステータスチェックのみ）
