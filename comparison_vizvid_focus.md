# VizVid側争点ベース比較メモ（VVMW vs YamaPlayer）

- 比較対象
  - VizVid (VVMW): `repos/VVMW`（submodule: `7592411f048ac830b25aba1d6392a9409c02cbda`）
  - YamaPlayer: `repos/YamaPlayer`（submodule: `ef2837ea5a322865fa37551cf40929aa6c0acfd4`）
- 方針
  - 「実装の一致度」よりも、**VizVid側から見た争点（仕様思想・フローの近似）**を優先。
  - 断定ではなく、証拠ベースで「強い/中程度/弱い」を付与。

---

## 1) YTDLP Solverの引数が酷似

**判定: 中〜強い類似（Medium-High）**

- VizVid 側 `YtdlpResolver` は、yt-dlp 実行時に次の引数列を使用。（`repos/VVMW/Packages/idv.jlchntoz.vvmw/Editor/Common/YtdlpResolver.cs:Line 123-133`）
  - `--flat-playlist --no-write-playlist-metafiles --no-exec -sijo - {url}`
- YamaPlayer 側 `YtdlpResolver` も、同じ主引数列を同順で組み立てて実行（先頭に `--extractor-args "youtube:lang=ja"` 追加）。（`repos/YamaPlayer/Editor/Playlist/YtdlpResolver.cs:Line 186-195`）
- ダウンロード元も双方とも `https://github.com/yt-dlp/yt-dlp/releases/latest/download/` をベースにしている。（`repos/VVMW/Packages/idv.jlchntoz.vvmw/Editor/Common/YtdlpResolver.cs:Line 16-23`, `repos/YamaPlayer/Editor/Playlist/YtdlpResolver.cs:Line 31-32`）

**反論（「VRChat動画プレイヤーなら近くなるのは自然」）への評価**
- この反論は妥当で、`yt-dlp` をプレイリスト取得に使う場合、`--flat-playlist` などは実務上選ばれやすい。
- したがって「引数が似ている」単体では、決定打としては弱い。

**所見**
- 争点として使う場合は、**引数一致を単独主張にせず**、Playlist import の内部データ対応など他の具体要素と束ねて提示するのが安全。

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

**判定: 中程度の類似（Medium）**

- 両者とも EditorWindow + `ReorderableList` で、
  - 左: プレイリスト一覧
  - 右: エントリ/トラック一覧
  - JSON Import/Export
  - YouTube Playlist 読み込み（yt-dlp 連携）
  - タイトル再取得・並べ替え系操作
  という構成を採用。
- YamaPlayer には VizVid 取り込み経路が明示実装されており、VizVid の内部変数名（`playListTitles`, `playListUrlOffsets`, `playListUrls`, `playListEntryTitles`, `playListPlayerIndex`）を直接読んで変換している。
- 対応表（1行要約）: `playListTitles -> PlaylistData.name | playListUrlOffsets -> urlOffset/urlCount（tracks切り出し境界） | playListUrls -> PlaylistTrack.url | playListEntryTitles -> PlaylistTrack.title | playListPlayerIndex -> PlaylistTrack.playerType`（`repos/YamaPlayer/Editor/Playlist/PlaylistImporter.cs:Line 213-217, 226-228, 237-247, 250-255` / 定義元 `repos/VVMW/Packages/idv.jlchntoz.vvmw/Runtime/VVMW/FrontendHandler_PlayList.cs:Line 9-13`）

**反論（「OSSなら互換importは自然」「Unity Editor拡張ならUIが似る」）への評価**
- この反論も妥当。OSS のエコシステムでは互換 import 実装は一般的で、Unity の `EditorWindow + ReorderableList` も定番構成。
- そのため、ここも単独では決定打にしにくい。

**所見**
- 使うなら「互換性を意図して内部変数へ直接アクセスしている事実」の提示に留め、違法性・不当性の評価とは切り分けるのが安全。

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

- **中程度根拠の争点**
  1. Playlist UI/操作導線 + VizVid内部データ互換 import（ただしOSS互換実装・Unity標準UI構成で説明可能）
  2. YTDLP Solver 引数列の近似（ただし業界/用途上の収束可能性あり）
- **補強根拠として使える争点**
  3. Event 駆動設計（属性配線 vs 文字列イベント駆動）
  4. Shader include による共通処理化フロー
  5. エディタ内 Package 自己更新導線

### 実務的な出し方（推奨）

- 本メモの5争点はいずれも、現状コード上は「設計・実装の近似」を示す材料であり、単体で強い断定根拠にはしにくい。
- とくに Playlist UI/互換import・YTDLP引数は、OSS文化やUnity/用途上の収束で合理的に説明できる余地を明示した上で使う。
- 主張する場合は、ライセンス条項・クレジット・由来説明など、非コード証拠と組み合わせて総合評価する。
