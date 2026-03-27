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

## 7. シャンダラー用CPU調査 (cmd_307)

440BXマザーボードでMTG Shandalerの快適ゾーン（66〜120MHz）に入るCPUが存在するか検証した。

### 440BX Slot 1 搭載可能CPU一覧（クロック昇順）

#### Slot 1ネイティブ

| CPU | クロック | FSB | 倍率 | L2キャッシュ | コア |
|-----|---------|-----|------|-------------|------|
| PII 233 | 233MHz | 66MHz | 3.5x | 512KB（半速） | Klamath |
| Celeron 266 | 266MHz | 66MHz | 4.0x | なし | Covington |
| PII 266 | 266MHz | 66MHz | 4.0x | 512KB（半速） | Klamath |
| Celeron 300 | 300MHz | 66MHz | 4.5x | なし | Covington |
| Celeron A 300A | 300MHz | 66MHz | 4.5x | 128KB（オンダイ） | Mendocino |
| PII 300 | 300MHz | 66MHz | 4.5x | 512KB（半速） | Klamath |
| PII 333 | 333MHz | 66MHz | 5.0x | 512KB（半速） | Deschutes |
| Celeron A 333 | 333MHz | 66MHz | 5.0x | 128KB | Mendocino |
| PII 350 | 350MHz | 100MHz | 3.5x | 512KB（半速） | Deschutes |
| ... | 350MHz〜1000MHz | 66〜133MHz | — | — | Deschutes〜Coppermine |

※Slotket経由でSocket 370 CPU（Celeron 300A〜533MHz、PIII 500〜1100MHz、Tualatin 1.0〜1.4GHz）も搭載可能だが、すべて300MHz以上。

#### 全CPUの最低クロック

**PII 233MHz（Klamath）= 440BX Slot 1で動作する最低クロックCPU**

### 判定: MTG快適ゾーンに入るか？

| 基準 | クロック範囲 | PII 233MHzとの比較 |
|------|------------|-------------------|
| 快適ゾーン | 66〜120MHz | **113MHz超過** — 入らない |
| 推奨ターゲット | 100〜133MHz | **100MHz超過** — 入らない |
| 境界帯（不確定） | 133〜200MHz | **33MHz超過** — 入らない |
| 速すぎゾーン | 200MHz以上 | **★該当する** |

**結論: PII 233MHzは速すぎゾーン（200MHz以上）に該当。440BXで搭載可能なCPUでMTG快適ゾーンに入るものは存在しない。**

### FSBダウンクロックの可能性

| 方法 | 実現性 | 結果 |
|------|--------|------|
| P2B/P3B-FのFSBを66MHz未満に | × 標準では66MHzが下限 | 不可 |
| P2B-S（ICS 9150-08搭載）で50MHz FSB | △ 特定リビジョンのみ | 50MHz×3.5x=175MHz（まだ速すぎゾーン）|
| Klamath倍率変更（1998年33週以前） | △ 初期ロットのみアンロック | 66MHz×2.0x=132MHz（境界帯。快適ゾーンには未到達）|
| cpukiller/Throttle | × 「very laggy」（VOGONS報告）| 非実用的 |

**FSBダウンクロック+倍率変更を両方適用しても、到達可能な最低は132MHz（境界帯）。快適ゾーン（120MHz以下）には入らない。**

### 低クロックCPU入手性

| CPU | eBay | eBay価格 | ヤフオク | 備考 |
|-----|------|---------|---------|------|
| PII 233MHz (Klamath) | 5〜8件 | $10〜$21+送料 | 1件（¥5,500） | Slot 1最低クロック |
| Celeron 266MHz (Covington) | 5〜10件 | $7.50〜$24+送料 | 0件 | L2なし、PII 233以下の性能 |
| Slotket（Slot 1→Socket 370） | 常時複数 | $15〜$50 | 0件 | ASUS S370-DL/MSI MS-6905等 |
| Celeron 300A (Mendocino/Socket 370) | 非常に豊富 | $5〜$15 | 1件（¥798〜） | Slotket必須。300MHzでMTGには速すぎ |

### 代替マザーボード選択肢

| 選択肢 | 追加費用 | MTG最適度 | 既存パーツ流用 | 備考 |
|--------|---------|----------|--------------|------|
| 440BX + PII 233MHz | $10〜$30 | × 速すぎゾーン | ◎全流用 | 最もシンプルだがMTGには不適 |
| 440LX マザー | $60〜$113+送料 | × PII 233〜333MHz制限で同様に速すぎ | △CPU流用可 | 推奨しない（440BXより高価で同じ結果）|
| Socket 7 全面移行 | $40〜$155 | ◎ Pentium 100〜233MHz直接搭載 | ✕全入替 | MTGにはドンピシャだが全パーツ入替 |
| Slotket + Celeron 300A on 440BX | $20〜$65 | × 300MHzは速すぎゾーン | ◎全流用 | PII 233より悪化 |

### 最終推奨

**86Boxでの仮想環境が唯一の実用的解決策。**

理由:
1. 440BX Slot 1の最低クロックCPU（PII 233MHz）はMTG快適ゾーン（66〜120MHz）に入らない
2. FSBダウンクロック・倍率変更を駆使しても到達可能な最低は132MHz（境界帯止まり）
3. cpukillerはVOGONS報告で「very laggy」。ソフトウェアスロットリングは非実用的
4. Socket 7に全面移行すればPentium 100〜133MHzでMTGに最適だが、$40〜$155のコストと全パーツ入替が必要
5. 86BoxならPentium 100MHz設定で直接快適ゾーンを実現可能。CPU購入不要、設定変更のみ

**結論: 「シャンダラー用にCPUを差し替えて遊ぶ」運用は440BXプラットフォームでは不可能。MTGは86Box（Pentium 100〜133MHz設定）で対応し、実機440BXはMM6+その他Win98タイトルに集中する構成が最適。**

## 8. 133-233MHz境界帯 徹底調査 (cmd_308)

cmd_306/307で「快適ゾーン: 66〜120MHz」「200MHz以上: 速すぎ」と判定したが、133-233MHz帯は報告が薄く判定が不確定だった。retro-forgeの実機CPU（PII 233MHz）がこの帯域に該当するため、本調査で集中的に情報収集した。

### 収集した報告一覧

| CPU/クロック | 種別 | 動作状況 | ソース |
|-------------|------|---------|--------|
| 486DX2/66MHz | 実報告 | やや遅いが十分プレイ可能 | Usenet rec.games.trading-cards |
| Pentium 100MHz | 実報告 | デュエル画面OK、Shandalarモードでやや重い | Usenet 1997 |
| Pentium 120MHz | 公式推奨 | MicroProse推奨最低ライン | [公式マニュアル](https://www.filfre.net/misc/magic_manual.pdf) |
| **Pentium 133-200MHz** | **推定** | **快適ゾーン（直接報告なし）** | — |
| **Pentium MMX 233MHz** | **推定** | **快適上限付近（直接報告なし）** | — |
| **PII 233MHz** | **推定** | **速すぎの可能性あり（直接報告なし）** | — |
| PII 333MHz | 間接報告 | 「P2 333を組んでプレイする」= 正常動作のギリギリとして認識 | [AnandTech](https://forums.anandtech.com/threads/old-game-magic-the-gathering-runs-too-fast.1370170/) |
| PII 400MHz | 実報告 | AI思考10分待ち（マップ速度への言及なし = 動作自体はしている） | AnandTech同上 |
| VIA Eden 533MHz (≈PII 266相当) | 間接報告 | Shandalar目的でビルド、動作結果未報告 | [VOGONS t=106157](https://www.vogons.org/viewtopic.php?t=106157) |
| Pentium III (500MHz+) | 実報告 | マップ上のキャラが速すぎて操作不能 | AnandTech, GOG, 複数ソース |
| AMD FX-53 (2.4GHz) | 実報告 | 速すぎて操作不能 | AnandTech |

**★重要な発見**: 133-233MHz帯の直接的な実動作報告はほぼ皆無。この「空白」自体が重要な情報。

### 空白の理由

1. **1997年の発売時期**: ゲーム発売時の主流CPUがPentium 100-200MHz。推奨120MHzの環境で遊んだユーザーは「普通に動いた」ため報告する動機がない
2. **速度問題の顕在化タイミング**: 速度問題が話題になるのはPentium III以降（500MHz+）。133-233MHz世代は「問題が起きない程度に速い」ため議論の対象外
3. **レトロコミュニティの時間軸**: VOGONS等は2000年代以降に活発化。その時点で133-233MHz機を持つ人はPIII/P4世代に移行済み

### 速度問題の二面性（重要）

Shandalarの速度問題には2つの**独立した**要素がある:

1. **マップ探索速度**: キャラクター移動が速すぎる（CPUクロック依存）→ 高クロックで問題
2. **AI思考時間**: カードバトル中のコンピュータ思考（CPUが速いほど短い）→ 高クロックでメリット

PII 400MHzの「10分待ち」はAI思考時間の問題であり、マップ速度とは別。高クロックCPUでは「マップは速すぎるがAIターンは遅い」という矛盾した体験もありうる。

### PII vs Pentium 実効性能比較

#### アーキテクチャの違い

| 項目 | Pentium (P5) / MMX | Pentium II (P6) |
|------|-------------------|-----------------|
| 実行方式 | インオーダー（命令順序通り） | アウトオブオーダー（動的実行） |
| パイプライン | 2本（U-pipe/V-pipe） | 3本デコード + リオーダーバッファ |
| L1キャッシュ | 16KB+16KB (P5) / 32KB (MMX) | 16KB+16KB |
| L2キャッシュ | 外部（マザーボード上） | SECC内蔵 512KB（半速） |

#### ベンチマーク比較（同一233MHz）

| ベンチマーク | Pentium MMX 233MHz | Pentium II 233MHz | PII優位率 |
|-------------|-------------------|-------------------|----------|
| Sandra CPU Multimedia | 455 | 625 | **+37%** |
| Sandra Memory | 479 | 885 | **+85%** |
| 3DMark99 CPU | 1,230 | 1,839 | **+50%** |
| Unreal avg FPS | 29.3 fps | 39.5 fps | **+35%** |

ソース: [VOGONS Sandra/3DMark](https://www.vogons.org/viewtopic.php?t=43351), [VOGONS Unreal](https://www.vogons.org/viewtopic.php?t=94493)

#### IPC比（ベンチマークから逆算）

- 整数演算: PII ≒ Pentium MMXの**約1.35〜1.50倍**
- メモリ帯域: PII ≒ Pentium MMXの**約1.85倍**
- ゲーム実効: PII ≒ Pentium MMXの**約1.35倍**

#### PII 233MHz ≒ Pentium何MHz相当か？

- **ゲーム実効ベース: PII 233MHz ≒ Pentium MMX 315〜350MHz相当**（IPC 1.35倍換算）
- メモリ帯域ベース: PII 233MHz ≒ Pentium MMX 430MHz相当（1.85倍換算）

### 「200MHz以上は速すぎ」の基準分析

cmd_306で確認した「200MHz以上は速すぎ」の閾値は**Pentium (P5)基準**と考えられる:

1. MTG Shandalar発売: 1997年4月。開発時の想定CPUはP5
2. Pentium IIは1997年5月発売（ゲームとほぼ同時）→ 開発時にPIIは存在しない
3. 「200MHz以上」の報告の多くは1997-1999年のP5ユーザーからのもの
4. PII 333MHzを「正常動作のギリギリ」として組もうとするユーザーがいた → PII 333MHz ≒ Pentium 450MHz相当 → 閾値付近として辻褄が合う

DOSゲーミング全般でも「最大500MHz（CPU種別により異なる）、233MHzまでが推奨」がVOGONSの一般的コンセンサス（[VOGONS t=102094](https://www.vogons.org/viewtopic.php?t=102094)）。

### PII 233MHzでのシャンダラー動作判定

#### 判定根拠

| 基準 | PII 233MHzとの比較 | 判定 |
|------|-------------------|------|
| 快適ゾーン (66-120MHz P5基準) | PII 233 ≒ P5 315-350MHz → **200MHz以上超過** | × |
| 境界帯 (133-200MHz P5基準) | **大きく超過** | × |
| 速すぎゾーン (200MHz+ P5基準) | **該当** (315-350MHz相当) | ★ |
| PII 333MHz「ギリギリ」(AnandTech) | PII 233はPII 333より30%低い | やや有利 |
| PII 400MHz「動作する」(AnandTech) | PII 233はPII 400より42%低い | 有利 |
| PIII 500MHz+「操作不能」 | PII 233はPIII 500よりかなり低い | 操作不能ではない |

#### 最終判定

## **△ 速すぎるが不可能ではない（cpukillerなしでギリギリ）**

根拠:
- PII 233MHz ≒ Pentium 315-350MHz相当 → 「200MHz以上は速すぎ」ゾーンに入る
- しかしPII 333MHzを「正常動作ギリギリ」として構築したユーザーがいる → PII 233MHzはそれより下
- PII 400MHzでもAI思考の遅さは報告されたがマップの操作不能は言及されていない
- 「操作不能」の明確な報告はPentium III（500MHz+、P5換算675MHz+）以上
- **推測の域を出ない**: 直接報告がないため、◎や○の確信は持てない

「速いが慣れればプレイできるかもしれない。ただし快適ではない」が最も妥当な推定。

### 殿への推奨

#### 結論は変わらない: MTGは86Boxが最適

| 方式 | MTG快適度 | 備考 |
|------|----------|------|
| **86Box (Pentium 100-133MHz設定)** | **◎ 確実に快適** | **★推奨（変更なし）** |
| 実機PII 233MHz直接プレイ | △ 速すぎの可能性大 | 「試す価値はある」が期待しすぎないこと |
| 実機PII 233MHz + cpukiller | × very laggy | 非実用的（VOGONS報告） |

#### 実機で「試してみる」場合のアドバイス

PII 233MHzの実機でShandalarをそのまま起動し、マップ移動速度を確認する価値はある。最悪でも「操作不能」ではなく「速すぎるが動く」程度の可能性が高い。もしギリギリ遊べるなら、86Boxを経由せずプレイできるメリットは大きい。

ただし過度に期待しないこと。快適ゾーン（66-120MHz P5基準）から大きく外れており、「遊べる」と「快適」は別物。

#### エミュレータ設定リファレンス

| エミュレータ | MTG推奨設定 | 備考 |
|-------------|-----------|------|
| 86Box | Pentium 166MHz → 133MHz → 100MHzと段階的に下げる | PII設定は避ける（同クロックでも1.35倍速い） |
| DOSBox-X | `cycles=fixed 20000` (≒Pentium 133-166MHz) | Ctrl+F11(減速)/F12(加速)でリアルタイム調整可能 |

調査情報源: VOGONS, AnandTech Forums, GOG.com Forums, Usenet (1997), CCGHQ/SlightlyMagic, 86Box GitHub Discussions, Archive.org, MicroProseマニュアル, dos486.com ほか計13ソース
