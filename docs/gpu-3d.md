# 3Dアクセラレータ調査 — MM6 + retro-forge構成

## 1. MM6のレンダリング方式

### 結論: ソフトウェアレンダリング専用（3Dアクセラレーション非対応）

MM6はDirect3D・Glide等のハードウェアアクセラレーションを**一切サポートしていない**。
New World Computing（開発元）が意図的にソフトウェアレンダリングを選択した。

#### 開発元の説明（公式FAQ）

> "SOME games do run better w/ custom API or D3D programming support, BUT NOT ALL GAMES.
> And not Might and Magic VI. Actually 3d [support] hurts the graphical level of detail,
> the frame rate, and the information flow."

**技術的理由:**
- MM6は1レベルあたり**10MB以上のテクスチャ**を同時に表示
- 1997年当時の3Dカードのテクスチャキャッシュは**最大2MB**
- ハードウェアアクセラレーションでは全テクスチャをVRAMに収容できず、逆に性能低下
- ソフトウェアレンダラは**32bit Z-buffer**使用（3Dカードは16bit上限）→ 描画精度で優位
- ソフトウェア方式により**16bitトゥルーカラー**を制限なく使用可能

参考: [GOG Forum — MM6 software mode only](https://www.gog.com/forum/might_and_magic_series/might_magic_vi_just_software_mode)

### GrayFace Patchの状況

GrayFace MM6 Patch v2.5.7は以下の機能を追加するが、**ハードウェアアクセラレーションは追加しない**:
- Always Run、Quick Save、マウスルック
- MP3再生、倍速モード
- 高解像度テクスチャ（MMExtension経由、ソフトウェアレンダリングのまま）

※ MM7/MM8のGrayFace PatchにはD3Dモード改善があるが、MM6には適用されない。

参考: [GrayFace MM Patches](https://grayface.github.io/mm/)

### PC Watch記事の検証

[PC Watch 1998年12月記事](https://pc.watch.impress.co.jp/docs/article/981211/game.htm)では
「3Dアクセラレータボード必須」と記載があるが、テスト環境は**3D BLASTER Banshee**（2D+3D統合カード）。
K6/233MHz + Mystique 4MBで「表示がカクつく」との報告は、3Dアクセラレーションではなく
**2Dフレームバッファ転送速度とVRAM容量の差**によるものと考えられる。
ソフトウェアレンダリングでも、ビデオカードの2D性能（blitting速度）は画面出力に影響する。

### MM6での3Dカードの価値

| 用途 | 3Dカードの効果 |
|------|---------------|
| MM6の3Dレンダリング | **効果なし**（ソフトウェア専用） |
| MM6の画面出力速度 | 2D性能が高いカードなら**間接的に改善** |
| MM6の解像度向上 | **効果なし**（ソフトウェアレンダラの制限内） |
| 他の3D対応ゲーム | **効果あり**（Glide/D3D対応ゲーム全般） |

**★重要な結論: MM6だけが目的なら、現在のMatrox G200 AGPで十分。**
G200は2D描画性能が優秀であり、MM6のソフトウェアレンダラとの相性は良好。
3Dカード追加の動機は「MM6以外の3D対応ゲーム」のため。

---

## 2. AGPスロット問題（AGP 1基のみ）

### 前提
- マザーボード: 440BX (P2B/P3B-F) — AGPスロット×1
- 現在: Matrox G200 AGP装着済み（2D描画用）
- PCIスロット: 複数空き

### 運用案

#### 案A: G200を外してAGP 3Dカード1枚に統合

| 項目 | 評価 |
|------|------|
| 構成 | AGPに2D+3D統合カード（Banshee/TNT/G400等） |
| メリット | シンプル。1枚で完結 |
| デメリット | G200の優秀な2D性能を失う。カードによっては2D画質低下 |
| MM6への影響 | 統合カードの2D性能次第。Banshee/TNTの2D性能はG200より劣る |
| 推奨度 | **△** — G200の2D画質を重視するなら非推奨 |

#### 案B: AGP 3Dカード + PCI G200

| 項目 | 評価 |
|------|------|
| 構成 | AGPに3Dカード + PCIにMatrox G200 PCI |
| メリット | G200の2D画質を維持しつつ3D追加 |
| デメリット | G200 PCI版の入手が必要。2枚挿しのドライバ競合リスク |
| 実現性 | **G200 PCI版は実在する**（8MB SDRAM版、16MB版あり） |
| 推奨度 | **△** — 技術的には可能だが、Win98での2枚挿し運用は設定が複雑 |

参考: [TheRetroWeb — Matrox G200 PCI](https://theretroweb.com/expansioncards/s/matrox-mill-g200-pci)

#### 案C: PCI 3Dアドオンカード（AGPはG200のまま）

| 項目 | 評価 |
|------|------|
| 構成 | AGPにG200（2D） + PCIに3Dカード |
| メリット | G200の2D画質を維持。3Dは別カードで処理 |
| デメリット | PCI帯域制限（132MB/s）。3dfx Voodoo系はPCI専用設計で問題なし |
| MM6への影響 | なし（MM6はG200の2Dで描画、3DカードはMM6に使われない） |
| 推奨度 | **○** — Voodoo2以外のPCI 3Dカードなら安価に実現可能 |

#### 案D: G200をG400 AGPに差し替え（2D+3D統合アップグレード）

| 項目 | 評価 |
|------|------|
| 構成 | AGPにMatrox G400（G200の上位互換） |
| メリット | G200同等以上の2D画質 + そこそこの3D性能。Matrox統一で安心 |
| デメリット | G400の3D性能はBanshee/TNTに劣る。Glide非対応 |
| MM6への影響 | 2D性能はG200以上 → MM6は問題なし |
| 推奨度 | **◎** — MM6メインならベスト。3D性能は妥協だが2D画質維持 |
| 価格 | ヤフオク落札相場: **¥1,000〜5,000**（平均¥1,854） |

### 推奨: 案D（G400差し替え）または案C（PCI 3Dアドオン）

- **MM6メイン → 案D（G400）**: 2D画質最優先。3D性能は限定的だがD3D対応。最安。
- **3Dゲームも重視 → 案C**: G200でMM6の2Dを確保しつつ、PCI 3DカードでGlide/D3Dゲーム対応。

---

## 3. MM6に最適な3Dカード候補

**前提: MM6自体は3Dカード不要。以下は「retro-forge全体として3Dゲームも楽しむ場合」の候補。**

### 候補一覧（4軸評価）

| # | カード | 費用 | 入手難度 | 当時らしさ | MM6との相性 | 総合 |
|---|--------|------|---------|-----------|------------|------|
| 1 | **Matrox G400 AGP** | ◎ ¥1,000-5,000 | ◎ 潤沢 | ◎ G200上位 | ◎ 2D最強 | **★推奨** |
| 2 | **NVIDIA Riva TNT AGP** | ◎ ¥1,000-5,000 | ◎ 潤沢 | ○ 1998年 | ○ 2Dは普通 | ○ |
| 3 | **S3 Savage4 AGP** | ○ $15-55 | ○ やや少 | ○ 1999年 | ○ 2Dは普通 | △ |
| 4 | **ATI Rage 128 AGP** | ◎ $10-25 | ○ | ○ 1999年 | ○ 2Dは普通 | ○ |
| 5 | **3dfx Banshee AGP** | △ ¥8,000-12,000 | △ 少ない | ◎ Glide対応 | ○ 2Dは普通 | △ |
| 6 | **Riva TNT2 AGP** | ◎ ¥1,500-6,000 | ◎ 潤沢 | ○ 1999年 | ○ 2Dは普通 | ○ |

### 各候補の詳細

#### 1. Matrox G400 AGP 16MB — ★最推奨

- **価格**: ヤフオク落札相場 ¥1,000〜5,000（平均¥1,854）/ eBay $10-50
- **特徴**: G200の直系上位。2D画質はクラス最高。3D性能はBanshee/TNT以下だがD3D対応
- **MM6との相性**: 最高。2Dフレームバッファ性能がG200以上
- **3D対応ゲーム**: D3D対応。Glide非対応。3D性能は中の下
- **当時らしさ**: 1999年発売。PII 400MHz時代のハイエンド2Dカード
- **注意点**: G400 MAX（32MB、DualHead）は別物で高価。16MB版で十分

参考: [eBay — Matrox G400](https://www.ebay.com/shop/matrox-g400?_nkw=matrox+g400)

#### 2. NVIDIA Riva TNT AGP 16MB

- **価格**: ヤフオク ¥1,000〜5,000 / eBay $16-57
- **特徴**: 1998年のメインストリーム3Dカード。D3D/OpenGL対応。2D性能は平凡
- **MM6との相性**: 普通。2D性能はG200に劣る
- **3D対応ゲーム**: D3D/OpenGL対応。Glide非対応。3D性能はBansheeと同等〜やや上
- **当時らしさ**: 1998年発売。PII世代の定番

参考: [ヤフオク — Riva TNT](https://auctions.yahoo.co.jp/search/search/riva%20tnt/23461/)

#### 3. S3 Savage4 AGP 8-16MB

- **価格**: eBay $15-58
- **特徴**: S3のAGP 3Dカード。テクスチャ圧縮(S3TC)対応。ドライバ品質に難あり
- **MM6との相性**: 普通。2D性能は平凡
- **3D対応ゲーム**: D3D対応だがドライバ互換性問題が多い。当時「相性問題のカード」として有名
- **当時らしさ**: 1999年。マイナーだが存在感はあった
- **注意点**: Win98ドライバの安定性に不安。レトロゲーミングにはリスキー

参考: [eBay — S3 Savage4](https://www.ebay.com/shop/s3-savage4)

#### 4. ATI Rage 128 AGP 16-32MB

- **価格**: eBay $10-25
- **特徴**: ATIのミッドレンジ。DVD再生支援あり。2D/3D統合
- **MM6との相性**: 普通。2D性能は平凡
- **3D対応ゲーム**: D3D対応。Glide非対応。3D性能はBanshee同等
- **当時らしさ**: 1999年。OEMマシンでよく見かけたカード
- **注意点**: Rage Pro（旧世代）と混同注意。Rage 128以降が実用レベル

参考: [eBay — Rage 128](https://www.ebay.com/shop/rage-128?_nkw=rage+128)

#### 5. 3dfx Voodoo Banshee AGP 16MB

- **価格**: ヤフオク落札 ¥8,000〜12,000 / eBay $40-170
- **特徴**: 3dfx唯一の2D+3D統合AGPカード。Glide対応
- **MM6との相性**: 普通。2D性能はG200に劣るがまずまず
- **3D対応ゲーム**: **Glide対応が最大の強み**。D3Dも対応
- **当時らしさ**: ◎ 1998年。Voodoo2の2D+3D統合版
- **問題点**: **¥7,000予算を超える**（ヤフオク相場¥8,000〜）。プレミア価格化進行中

参考: [ヤフオク — Voodoo Banshee落札相場](https://auctions.yahoo.co.jp/closedsearch/closedsearch/voodoo%20banshee/2084211538/)

#### 6. NVIDIA Riva TNT2 AGP 32MB

- **価格**: ヤフオク ¥1,500〜6,000 / eBay $16-57
- **特徴**: TNTの後継。1999年のベストセラー3Dカード。D3D/OpenGL対応
- **MM6との相性**: 普通。2D性能はTNTとほぼ同等
- **3D対応ゲーム**: TNTより高速。Glide非対応だがほぼ全D3Dゲームで良好
- **当時らしさ**: 1999年。PII/PIII世代の超定番カード
- **注意点**: TNT2 M64（廉価版）はVRAM帯域が半分。無印TNT2/TNT2 Proを推奨

参考: [eBay — Riva TNT2](https://www.ebay.com/b/NVIDIA-NVIDIA-Computer-Graphics-Cards-NVIDIA-Riva-TNT2/27386/bn_7116862836)

---

## 4. 出品リスト（調査時点: 2026年3月）

### ヤフオク

| カード | 落札相場 | 出品状況 |
|--------|---------|---------|
| 3dfx Banshee AGP | ¥8,000〜12,000 | Diamond Monster Fusion 16MB: ¥12,000 / I-O DATA GA-VDB16: ¥8,750 |
| Matrox G400 AGP | ¥1,000〜5,000 (平均¥1,854) | 16MB版多数出品あり |
| Riva TNT AGP | ¥1,000〜5,000 | 複数出品あり。¥1,000〜¥1,500スタート |
| Riva TNT2 AGP | ¥1,500〜6,000 | 32MB版含め多数 |

### eBay

| カード | 価格帯 (USD) | 代表的出品 |
|--------|------------|-----------|
| 3dfx Banshee AGP | $40-170 | Diamond Monster Fusion 16MB, Creative CT6750 |
| Matrox G400 AGP | $10-50 | 16MB SGRAM版: $10-26, 32MB版: $30-65 |
| Riva TNT AGP | $16-57 | Creative CT6870, ASUS V3400TNT |
| S3 Savage4 AGP | $15-58 | Diamond Savage4 Pro 8MB: $15, Innovision 16MB: $55 |
| ATI Rage 128 AGP | $10-25 | 32MB版: $10-25 |
| Riva TNT2 AGP | $16-57 | 32MB版多数 |

---

## 5. 総合推奨

### MM6メインの場合（現構成で十分）

**追加カード不要。** Matrox G200 AGPの2D性能でMM6は快適に動作する。
MM6はソフトウェアレンダリング専用のため、3Dカードを追加しても描画は改善しない。

### 3Dゲームも楽しみたい場合

**第1推奨: Matrox G400 AGP（G200差し替え）**
- 予算: ¥1,000〜5,000（予算内）
- 理由: G200の2D画質を維持（むしろ向上）しつつ、D3D 3Dも対応
- MM6: 問題なし（2D性能はG200以上）
- 制約: Glide非対応（3dfxゲームは動かない）

**第2推奨: Riva TNT2 AGP（G200差し替え）**
- 予算: ¥1,500〜6,000（予算内）
- 理由: 3D性能はG400より上。D3D/OpenGL対応。当時の超定番
- MM6: 2D性能はG200より劣る → MM6での画面出力がわずかに遅くなる可能性
- 制約: Glide非対応。2D画質でG200/G400に劣る

**Glide対応が必要な場合: PCI 3Dアドオン（案C）**
- AGPにG200/G400を残しつつ、PCIにBanshee PCI等を追加
- ただしVoodoo系PCIカードも値上がり傾向

### ★最終推奨

| 優先度 | 選択 | 理由 |
|--------|------|------|
| **MM6最優先** | 現状維持（G200） | 3Dカード不要。追加投資ゼロ |
| **MM6+軽い3D** | G400差し替え（¥2,000前後） | 2D維持+D3D追加。最小コスト |
| **本格3Dゲーム** | TNT2差し替え（¥3,000前後） | 3D性能重視。2D画質は妥協 |
