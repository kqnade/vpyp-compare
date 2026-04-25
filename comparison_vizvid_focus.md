# VizVid (VVMW) / YamaPlayer 比較メモ（情報整理）

- 比較対象
  - VizVid (VVMW): `repos/VVMW`（submodule: `7592411f048ac830b25aba1d6392a9409c02cbda`）
  - YamaPlayer: `repos/YamaPlayer`（submodule: `ef2837ea5a322865fa37551cf40929aa6c0acfd4`）
- 目的
  - 主張や評価ではなく、実装事実の対応関係を整理する。

---

## 1. YTDLP Resolver（引数・取得元URL）

### VizVid (VVMW)
- `YtdlpResolver` でダウンロード基底URLを定義。
  - `https://github.com/yt-dlp/yt-dlp/releases/latest/download/`
- プレイリスト取得時の実行引数:
  - `--flat-playlist --no-write-playlist-metafiles --no-exec -sijo - {url}`
- 参照: `repos/VVMW/Packages/idv.jlchntoz.vvmw/Editor/Common/YtdlpResolver.cs:Line 16-23, 123-133`

### YamaPlayer
- `YtdlpResolver` でダウンロードURLを定義。
  - `https://github.com/yt-dlp/yt-dlp/releases/latest/download/{FILENAME}`
- プレイリスト取得時の実行引数:
  - `--extractor-args "youtube:lang=ja" --flat-playlist --no-write-playlist-metafiles --no-exec -sijo - "{playlistUrl}"`
- 参照: `repos/YamaPlayer/Editor/Playlist/YtdlpResolver.cs:Line 31-32, 186-195`

---

## 2. Event Attribute / Event配線

### YamaPlayer
- `RegisterEventAttribute` / `RegisterEventTriggerAttribute` を定義。
- ビルド処理で属性を走査し、`UnityEventTools.AddStringPersistentListener(...)` を使って `SendCustomEvent` を登録。
- 参照:
  - `repos/YamaPlayer/Runtime/Attributes/RegisterEventAttribute.cs`
  - `repos/YamaPlayer/Editor/Attribute/RegisterEventAttributeBuildProcess.cs`

### VizVid (VVMW)
- `Core` が `UdonSharpEventSender` を継承。
- 文字列イベント名ベースの呼び出し（`SendEvent(...)` / `SendCustomEvent...`）を利用する構成。
- 参照:
  - `repos/VVMW/Packages/idv.jlchntoz.vvmw/Runtime/VVMW/Core.cs`

---

## 3. Playlist Editor UI / VizVid互換インポート

### UI構成（共通点）
- 両者とも Unity Editor 拡張として `EditorWindow` + `ReorderableList` を使用。
- 参照:
  - VizVid: `repos/VVMW/Packages/idv.jlchntoz.vvmw/Editor/VVMW/PlayListEditorWindow.cs`
  - Yama: `repos/YamaPlayer/Editor/Playlist/PlaylistEditorWindow.cs`

### YamaPlayerのVizVid取り込み実装
- `ImportPlaylistsFromVizVid` で VizVid 側のプログラム変数名を直接取得。
  - `playListTitles`
  - `playListUrlOffsets`
  - `playListUrls`
  - `playListEntryTitles`
  - `playListPlayerIndex`
- 参照: `repos/YamaPlayer/Editor/Playlist/PlaylistImporter.cs:Line 208-257`

### 変数対応（1行対応表）
- `playListTitles -> PlaylistData.name | playListUrlOffsets -> urlOffset/urlCount（tracks境界） | playListUrls -> PlaylistTrack.url | playListEntryTitles -> PlaylistTrack.title | playListPlayerIndex -> PlaylistTrack.playerType`
- 参照:
  - 取り込み先: `repos/YamaPlayer/Editor/Playlist/PlaylistImporter.cs:Line 213-217, 226-228, 237-247, 250-255`
  - VizVid側定義: `repos/VVMW/Packages/idv.jlchntoz.vvmw/Runtime/VVMW/FrontendHandler_PlayList.cs:Line 9-13`

---

## 4. Screen Shader include フロー

### VizVid (VVMW)
- 複数 shader から共通 `cginc` を include。
  - 例: `VideoSurface.shader` / `VideoUnlit.shader` から `VideoShaderCommon.cginc` 参照。
- 参照:
  - `repos/VVMW/Packages/idv.jlchntoz.vvmw/Shaders/VideoSurface.shader`
  - `repos/VVMW/Packages/idv.jlchntoz.vvmw/Shaders/VideoUnlit.shader`
  - `repos/VVMW/Packages/idv.jlchntoz.vvmw/Shaders/VideoShaderCommon.cginc`

### YamaPlayer
- 複数 shader から共通 `cginc` を include。
  - 例: `Screen.shader` / `AVProBlit.shader` から `YamaPlayerShader.cginc` 参照。
- 参照:
  - `repos/YamaPlayer/Assets/Shaders/Screen.shader`
  - `repos/YamaPlayer/Assets/Shaders/AVProBlit.shader`
  - `repos/YamaPlayer/Assets/Shaders/YamaPlayerShader.cginc`

---

## 5. Package更新導線（Editor側）

### VizVid (VVMW)
- `VVMWEditorBase` 内で `PackageSelfUpdater` を生成し、バージョン通知UIを描画する構成。
- 参照: `repos/VVMW/Packages/idv.jlchntoz.vvmw/Editor/VVMW/EditorBase.cs`

### YamaPlayer
- `PackageManager`（`[InitializeOnLoad]`）で更新確認処理を実行。
- `VpmResolver` でリポジトリ追加、バージョン列挙、更新実行。
- 参照:
  - `repos/YamaPlayer/Editor/Package/PackageManager.cs`
  - `repos/YamaPlayer/Editor/Package/VpmResolver.cs`

---

## 付記

- 本メモは、現行コード上で確認できる対応関係・処理フローの整理のみを記載。
