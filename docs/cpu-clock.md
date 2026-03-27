# CPUクロック・スイートスポット調査 (cmd_306)

最終更新: 2026-03-27

## 1. 背景・課題

retro-forgeの対象タイトル2本が相反するCPUクロック要件を持つ:
- **MTG Shandalar (1997)**: CPUサイクル＝ゲーム速度。速いCPUだとオーバーワールドが操作不能
- **Might and Magic VI (1998)**: リアルタイム3D RPG。高速CPUほど快適（FPSキャップなしだが致命的ではない）

この矛盾を分析し、実機・仮想環境それぞれで最適なCPUを特定する。

## 2. MTG Shandalar 調査結果

### 公式システム要件
- 最低: 486 100MHz
- 推奨: Pentium 120MHz以上
- AI思考時間はCPUが速いほど短い（メリット）
- 描画・アニメーション速度がCPUクロックに直結（バグ）

### 実稼働報告

| ソース | CPU | 状態 | 備考 |
|--------|-----|------|------|
| Usenet | 486DX2/66MHz | やや遅いが十分プレイ可能 | [source](https://groups.google.com/g/rec.games.trading-cards.magic.strategy/c/4hmdHP7LIWY) |
| Usenet | Pentium 75MHz | 正常動作 | 同上 |
| Usenet | Pentium 100MHz | オーバーワールド問題なし | デュエル突入時のみ遅め |
| AnandTech | Pentium II 400MHz | AI思考10分待ち | オーバーワールド速度への言及なし |
| Abandonware DOS | 686クラス（200-450MHz帯） | 速すぎて歩き回れない | [source](https://www.abandonwaredos.com/abandonware-game.php?abandonware=Magic:+The+Gathering&gid=1967) |
| AnandTech | Pentium III（600MHz-1GHz帯推定） | マップ移動が速すぎ制御不能 | Win95 VM内でも発生 [source](https://forums.anandtech.com/threads/old-game-magic-the-gathering-runs-too-fast.1370170/) |
| AnandTech | AMD FX-53（2.4GHz） | ウィザード移動が速すぎ操作不能 | 同上 |
| VOGONS | 現代CPU | very very too fast | Throttle+CPUKiller→very laggy [source](https://www.vogons.org/viewtopic.php?t=738) |

### 快適ゾーン・閾値

| ゾーン | クロック範囲 | 状態 |
|--------|-------------|------|
| 快適ゾーン | **66MHz〜120MHz** | 正常動作報告あり |
| 推奨ターゲット | **100〜133MHz** | マニュアル推奨レンジ |
| 境界帯（不確定） | 133MHz〜200MHz | 報告が薄い |
| 速すぎゾーン | **200MHz以上** | 確実に速度問題発生 |

### コミュニティパッチの速度対応状況

| パッチ | 速度問題修正 | 備考 |
|--------|-------------|------|
| 2010 Edition (Hip63) | **なし** | グラフィック描画バグ修正が主目的 |
| 2012 Revisited | **なし** | OS互換性・ポータブル版 |
| Shandalar 2003 (chillbrain) | **明示なし** | カード追加が主目的 |
| Manalink 3.0 | **明示なし** | ウィンドウモード対応 |

**結論**: オーバーワールド速度のクロック依存問題を根本修正したパッチは確認できなかった。

### cpukillerの実用性

- VOGONS: Throttle + CPUKiller → 速度は近づくが「very laggy」
- Mo'Slo: 効果の確認報告なし
- **結論: ソフトウェアCPUスロットリングは実用的でない**

## 3. Might and Magic VI 調査結果

### 公式推奨スペック

| 項目 | 最低 | 推奨 | リアルタイム戦闘 |
|------|------|------|-----------------|
| CPU | Pentium 90MHz | Pentium 166MHz以上 | PII/K6 266MHz |
| RAM | 16MB | 32MB | — |

### CPU速度依存の特性

**MTGとは本質的に異なる**:
- MTG: CPUサイクル＝ゲーム速度（完全依存）
- MM6: Windowsタイマーベースの時間管理（部分的依存）

高速CPUで発生する問題:
- 方向キーで580度回転（旋回速度過敏）→ Turn Rate「Smooth」で緩和可能
- キャラ作成画面アニメーション高速回転（ゲーム本編への影響は軽微）
- CPU使用率異常（i5 4570 3.2GHzで35%）→ FPSキャップなしが原因

### GrayFace MM6 Patch v2.5.7

- TurnSpeedNormal/TurnSpeedDoubleオプションで旋回速度制御
- 32bitカラー対応（ウィンドウ/ボーダーレスモード）
- Win7/10/11との互換性確保
- **速度関連問題の実用的な解決策として有効**
- https://grayface.github.io/mm/

### 実稼働報告

| ソース | CPU | 結果 |
|--------|-----|------|
| VOGONS | Pentium MMX 200MHz + Win98SE | 推奨タイトルとして強く推薦 |
| VOGONS | AMD Athlon 1.3GHz + Win98SE | 動作OK |
| Steam | 現代PC | Turn Rate「Smooth」で解決 |
| GOG | i5 4570 3.2GHz | CPU使用率35%（FPSキャップなし）|

**結論**: MM6はPentium 166MHz以上で快適動作。高速CPUでもGrayFace Patch + Turn Rate設定で実用上問題なし。**低クロックCPUでも動作する**（最低Pentium 90MHz）。

## 4. スイートスポット分析

### 矛盾の構造

| | MTG Shandalar | Might and Magic VI |
|--|---------------|-------------------|
| 快適ゾーン | 66〜120MHz | 166MHz以上（快適は266MHz+） |
| 問題発生帯 | 200MHz以上（速すぎ） | なし（高速でもパッチで対処可） |
| パッチで解決? | **不可**（根本修正パッチなし） | **可能**（GrayFace Patch） |

### 解決策の比較

#### シナリオA: パッチ前提（推奨）

MM6側はGrayFace Patchで高速CPU問題を解決できるため、**MTGの快適ゾーンに合わせる**のが合理的。

- **共通快適ゾーン: Pentium 100〜133MHz**
- MTG: マニュアル推奨レンジ内、快適動作確認済み
- MM6: 最低ラインぎりぎり〜推奨未満だが動作する（Pentium 90MHzが最低要件）
- MM6のリアルタイム戦闘はPII 266MHz推奨だが、ターン制モードなら100MHzでもプレイ可能
- GrayFace PatchのTurnSpeed設定で旋回速度は制御可能

**86Box構成**: Pentium 100〜133MHz設定が最適。MTGに最適化しつつMM6も動作可能。

#### シナリオB: パッチなし

パッチなしの場合、両タイトルの快適ゾーンに重なりがない:
- MTG: 120MHz以下が必要
- MM6: 166MHz以上が推奨（リアルタイム戦闘は266MHz以上）

**妥協点: Pentium 133〜166MHz**
- MTG: 境界帯（速すぎる可能性あり。要実測）
- MM6: ぎりぎり推奨ライン

ただしMM6をパッチなしで高速CPUで動かすと旋回速度が580度回転になるため、パッチ適用を強く推奨する。

#### シナリオC: 86Boxで動的切り替え（仮想環境のみ）

86Boxなら設定変更→VM再起動でCPUクロックを切り替え可能:
- MTGプレイ時: Pentium 100MHz設定
- MM6プレイ時: Pentium II 300MHz設定

これが最も柔軟だが、ゲーム切り替えのたびにVM再起動が必要。

#### cpukillerでの対処

- MTG: Throttle + CPUKiller → 「very laggy」（VOGONS報告）で非実用的
- ソフトウェアスロットリングはレイテンシが不均一で、リアルタイム制御が必要なゲームには不向き
- **結論: cpukillerは推奨しない**

### 分析まとめ

| 方式 | MTG | MM6 | 実用性 |
|------|-----|-----|--------|
| **86Box 100-133MHz + GrayFace Patch** | ◎ 快適 | ○ 動作可（ターン制推奨） | **★推奨** |
| 86Box動的切り替え | ◎ | ◎ | ○（VM再起動必要） |
| 実機PII 400MHz + cpukiller | △ laggy | ◎ 快適 | × 非実用的 |
| 実機Pentium 133MHz + パッチなし | ○ 境界帯 | △ 最低ライン | △ |
| 実機Pentium 133MHz + GrayFace | ○ 境界帯 | ○ 動作可 | ○ |

## 5. Pentium II 入手性・価格

※実機構築を検討する場合の参考データ。仮想環境（86Box）ならCPU購入は不要。

### 各クロック別 出品状況

| CPU | ヤフオク出品数 | ヤフオク相場 | eBay出品数 | eBay相場 | 入手容易性 |
|-----|--------------|-------------|-----------|---------|------------|
| PII 233MHz (Klamath) | 1件 | ¥491〜¥5,500 | 少数 | $5〜$20 | 稀少 |
| PII 266MHz | 1件(モバイル) | ¥540〜¥1,000 | 少数 | $5〜$15 | 稀少 |
| PII 300MHz (Deschutes) | 0件 | ¥2,000(落札相場) | 複数 | $8〜$20 | ヤフオク稀少/eBay普通 |
| PII 333MHz | 1件(モバイル) | ¥511(落札相場) | 12〜14件 | $11〜$61 | ヤフオク極稀少/eBay普通 |
| PII 350MHz | 2件 | ¥1,500〜¥3,980 | 複数 | $10〜$30 | 普通 |
| **PII 400MHz** | **3件** | ¥1,000〜¥3,500 | **65件以上** | $8〜$40 | **★最も入手しやすい** |
| Celeron 300A | 1件 | ¥798〜¥3,980 | 10件以上 | $5〜$15 | eBay充実 |

### クーラー付き出品リスト

#### Pentium II 400MHz（クーラー付き最多）

**ヤフオク:**
- M0739# 動作品 PII SL2S7 400MHz クーラーファン付き — ¥2,500（即決¥3,500）

**eBay:**
- PII 400MHz SL357 Slot 1 Cooler 3-Pin — https://www.ebay.com/p/17067918265
- PII 400mhz SL2U6 + Cooler — https://www.ebay.com/p/10018471899
- PII 400mhz SL357 + Cooler — https://www.ebay.com/p/1151320056
- PII SL3D5 400mhz + Cooler — https://www.ebay.com/itm/405009170917
- PII SL2SH 400MHz with heatsink — https://www.ebay.com/itm/233467310552

#### その他クロック（クーラー付き）

| CPU | ソース | URL |
|-----|--------|-----|
| PII 350MHz | eBay | https://www.ebay.com/itm/174510531295 |
| PII 300MHz | eBay | https://www.ebay.com/p/1709159686 |
| PII 233MHz | eBay | https://www.ebay.com/p/1526265733 |
| Celeron 300A | eBay | https://www.ebay.com/itm/404544308760 |

#### 別途Slot1クーラー

CPU単体購入の場合:
- Intel PII/PIII SECC2 Slot1 HeatSink + Fan (Sanyo Denk) — $29.95 https://www.ebay.com/itm/196893085080
- NEW Slot 1 CPU Cooler CoolerOne — $22.99 https://www.ebay.com/itm/234585395639

## 6. 最終推奨

### 仮想環境（86Box）— 推奨

**86Box設定: Pentium 100〜133MHz + GrayFace Patch適用**

- MTG: 快適ゾーン内で問題なく動作
- MM6: GrayFace Patchで速度問題対処。ターン制戦闘なら十分快適
- CPU購入不要、クロック切り替えも容易（VM再起動のみ）
- MM6を本格的にリアルタイム戦闘でプレイしたい場合のみ、一時的にPII 300MHz設定に切り替え

### 実機構築の場合

**推奨CPU: Pentium II 400MHz（クーラー付き）**

理由:
1. **入手性が圧倒的**: eBay 65件以上、クーラー付き出品も最多
2. **価格が手頃**: $8〜$40（クーラー付きでも$20前後）
3. **MM6に最適**: 推奨スペックを大きく上回り、リアルタイム戦闘も快適
4. **MTGは86Boxで対応**: 実機PII 400MHzではMTGは速すぎるため、MTGプレイ時のみホスト上で86Box（Pentium 100MHz設定）を使用
5. **440BXマザーボードとの相性**: Slot 1 Deschutesコア、100MHz FSB。440BXの設計ターゲットそのもの

**注意**: 実機PII 400MHzでMTGを直接プレイすることはできない（速すぎる）。MTGは86Box内でプレイする前提。MM6 + その他Win98ゲームは実機で快適に動作する。
