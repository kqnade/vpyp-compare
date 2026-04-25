# VizVid側争点ベース比較メモ（VVMW vs YamaPlayer）

- 比較対象
  - VizVid (VVMW): `repos/VVMW`（submodule: `7592411f048ac830b25aba1d6392a9409c02cbda`）
  - YamaPlayer: `repos/YamaPlayer`（submodule: `ef2837ea5a322865fa37551cf40929aa6c0acfd4`）
- 方針
  - 「実装の一致度」よりも、**VizVid側から見た争点（仕様思想・フローの近似）**を優先。
  - 断定ではなく、証拠ベースで「強い/中程度/弱い」を付与。

---

## 1) YTDLP Solverの引数が酷似

**判定: 強い類似（High）**

- VizVid 側 `YtdlpResolver` は、yt-dlp 実行時に次の引数列を使用。
  - `--flat-playlist --no-write-playlist-metafiles --no-exec -sijo - {url}`
- YamaPlayer 側 `YtdlpResolver` も、同じ主引数列を同順で組み立てて実行。
- ダウンロード元も双方とも `https://github.com/yt-dlp/yt-dlp/releases/latest/download/` をベースにしている。

**所見**
- 単なる yt-dlp 利用では説明しづらいレベルで、引数の並び/意図（軽量 JSON 抽出）まで一致。

---

## 2) Event Attributeの設計思想が酷似

**判定: 中程度の類似（Medium）**

- YamaPlayer は `RegisterEventAttribute` / `RegisterEventTriggerAttribute` を持ち、ビルド時に属性を走査して `UnityEventTools.AddStringPersistentListener(...)` で Udon 側 `SendCustomEvent` を配線する設計。
- VizVid 側は属性配線そのもののコードは比較対象内で直接確認できないが、`Core : UdonSharpEventSender` と、`SendEvent("...")` / `SendCustomEvent...` による文字列イベント駆動が全体設計として多用される。

**所見**
- 「Unity 側イベント → Udon の文字列イベント名へ橋渡しする」思想は近い。
- ただし、**属性の実装体そのものの一致はこの比較範囲では確認できず**、完全一致の断定は保留。

---

## 3) Playlist UIが酷似

**判定: 強い類似（High）**

- 両者とも EditorWindow + `ReorderableList` で、
  - 左: プレイリスト一覧
  - 右: エントリ/トラック一覧
  - JSON Import/Export
  - YouTube Playlist 読み込み（yt-dlp 連携）
  - タイトル再取得・並べ替え系操作
  という構成を採用。
- YamaPlayer には VizVid 取り込み経路が明示実装されており、VizVid の内部変数名（`playListTitles`, `playListUrlOffsets`, `playListUrls`, `playListEntryTitles`, `playListPlayerIndex`）を直接読んで変換している。

**所見**
- UI/操作導線だけでなく、VizVid 互換 import の内部データマッピングまで含めると、単なる偶然一致の範囲は超えている。

---

## 4) Screen shader includeの処理フローが酷似

**判定: 弱〜中程度の類似（Low-Medium）**

- 双方とも「各 shader エントリが共通 `*.cginc` を `#include` し、AVPro 向け UV 補正/アスペクト補正を共通関数化」という**構造パターン**は一致。
- ただし、共通 include ファイルの中身は YamaPlayer 側が簡素化されており、VizVid 側の stereo/size mode など多機能実装と 1:1 で同一ではない。

**所見**
- 「構造は似ているが、実装粒度は異なる」。
- 争点化するなら「処理フロー（分離設計）」は示せるが、コード一致率の主張は弱め。

---

## 5) Package Auto Updaterの処理の流れに強い類似

**判定: 中程度の類似（Medium）**

- VizVid 側:
  - Editor 初期化時に `PackageSelfUpdater(... listingsID, listingsURL ...)` を生成し、バックグラウンドで導入状態確認、Inspector 上で更新通知 UI を描画。
- YamaPlayer 側:
  - `[InitializeOnLoad]` で起動時に更新確認、VPM repository の migrate、バージョン列挙、セマンティック比較、更新実行という流れを実装。
- いずれも「エディタ起動/表示導線で自パッケージ更新を案内・実行する」自己更新 UX を実現。

**所見**
- フロー思想は近い。
- ただし実装クラスや API は異なり、VizVid の `PackageSelfUpdater` 本体コードが比較範囲外のため、強い同一性断定は避けるべき。

---

## 総合（VizVid側主張を主軸にした整理）

- **強い根拠が置ける争点**
  1. YTDLP Solver 引数列の酷似
  2. Playlist UI/操作導線 + VizVid内部データ互換 import
- **補強根拠として使える争点**
  3. Event 駆動設計（属性配線 vs 文字列イベント駆動）
  4. Shader include による共通処理化フロー
  5. エディタ内 Package 自己更新導線

### 実務的な出し方（推奨）

- 主張の芯は「**引数一致**」「**Playlist編集UX + VizVid互換 import の具体実装**」に置く。
- 残り3点は「設計思想の連続性」を示す補助線として提示し、過度な同一コード断定は避ける。
