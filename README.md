# Poached Eggs

自作の左右分割キーボード **Poached Eggs** 用の [ZMK](https://zmk.dev/) ファームウェア（シールドモジュール）です。

このリポジトリは ZMK の **外部モジュール** 兼 **ユーザー設定 (user config)** として構成されており、GitHub Actions で左右それぞれのファームウェア（`.uf2`）をビルドできます。

---

## 特徴

- **ZMK ファームウェア**（`v0.3.0` に固定）
- **左右分割**構成（左=セントラル / 右=ペリフェラル、Bluetooth 接続）
- **ZMK Studio 対応** — PC やブラウザからキーマップをリアルタイム編集可能（ロック機能付き）
- **4 レイヤー** — QWERTY / 数字・記号 / ファンクション・矢印 / Bluetooth 設定
- スリープ対応で省電力

## ハードウェア構成

| 項目 | 内容 |
|------|------|
| MCU | [Seeeduino XIAO BLE](https://wiki.seeedstudio.com/XIAO_BLE/)（`seeeduino_xiao_ble`） |
| マトリクス | 4 行 × 14 列、`col2row` ダイオード方向 |
| 物理レイアウト | 上 3 段は左右 6 列ずつ + 親指キー（計 50 キー） |
| 行 GPIO | `xiao_d` 10, 9, 8, 7 |
| 列 GPIO | `xiao_d` 0〜6（右手は列オフセット +7） |

## リポジトリ構成

```
.
├── build.yaml                 # GitHub Actions のビルドマトリクス（左右の .uf2 を生成）
├── zephyr/module.yml          # ZMK 外部モジュールとしての定義（board_root: .）
├── config/
│   ├── west.yml               # 取得する ZMK 本体のバージョン (v0.3.0)
│   ├── poached_eggs.keymap    # キーマップ（全レイヤー）
│   ├── poached_eggs.json      # ZMK Studio / keymap-editor 用の物理レイアウト
│   └── boards/shields/poached_eggs/
│       ├── poached_eggs.dtsi             # 物理レイアウト・マトリクス変換・kscan 定義
│       ├── poached_eggs_left.overlay     # 左手の列 GPIO
│       ├── poached_eggs_right.overlay    # 右手の列 GPIO + col-offset
│       ├── poached_eggs.conf             # 左右共通の設定 (CONFIG_*)
│       ├── poached_eggs_left.conf        # 左手固有の設定（Studio 有効化など）
│       ├── poached_eggs_right.conf       # 右手固有の設定
│       ├── Kconfig.shield                # シールド名の定義
│       ├── Kconfig.defconfig             # 分割・セントラル役割などのデフォルト
│       └── poached_eggs.zmk.yml          # シールドのメタデータ（Studio 用）
└── .github/workflows/build.yml           # ビルドワークフロー
```

## ビルド方法

### GitHub Actions（推奨）

1. このリポジトリに push する（または GitHub の **Actions** タブから手動実行）。
2. ワークフローが完了したら、実行結果ページ下部の **Artifacts** から `firmware` をダウンロード。
3. ZIP の中に `poached_eggs_left.uf2` と `poached_eggs_right.uf2` が含まれます。

### ローカルでビルドする場合

ZMK の[ローカルビルド環境](https://zmk.dev/docs/development/local-toolchain/setup)を用意した上で、別の west ワークスペースにこのリポジトリをモジュールとして取り込みます。詳細は ZMK 公式ドキュメントを参照してください。

## 書き込み（フラッシュ）方法

XIAO BLE への書き込みは UF2 ブートローダ経由で行います。

1. キーボードを USB で PC に接続する。
2. **リセットボタンを素早く 2 回押す**（ダブルタップ）。
3. `XIAO-SENSE` などの名前で USB ドライブとしてマウントされる。
4. 対応する `.uf2` ファイル（左手なら `poached_eggs_left.uf2`）をドライブにコピーする。
5. 自動的に再起動して書き込み完了。左右それぞれで実施する。

> 初回は左右ともに書き込み、その後ペアリングしてください。

## キーマップ

`config/poached_eggs.keymap` で定義しています。レイヤーは 4 つです。

| # | レイヤー | アクセス方法 | 用途 |
|---|----------|--------------|------|
| 0 | default | （ベース） | QWERTY 配列 |
| 1 | lower | 左親指 `&mo 1` を長押し | 数字・記号 |
| 2 | raise | 右親指 `&mo 2` を長押し | ファンクション・矢印・括弧 |
| 3 | bt | lower + raise を同時に長押し（各レイヤーの `&mo 3`） | Bluetooth 選択/クリア・Studio ロック解除 |

### default レイヤー

```
 TAB   Q   W   E   R   T  │  Y   U   I   O   P   -
CTRL   A   S   D   F   G  │  H   J   K   L   ;   '
SHFT   Z   X   C   V   B  │  N   M   ,   .   /  SHFT
            GUI ALT L1 SPC │ RET L2 BSPC GUI
```

### lower レイヤー（数字・記号）

```
 ESC   1   2   3   4   5  │  6   7   8   9   0   `
   `   !   @   #   $   %  │  ^   &   *   (   )   ~
 RET   ·   ·   ·   ·   ·  │  ·   ·   ·   {   }   |
                  DEL L3
```

### raise レイヤー（ファンクション・矢印）

```
  F1  F2  F3  F4  F5  F6  │  F7  F8  F9 F10 F11 F12
   ·   ·   ·   ·   ·   ·  │  ·   ←   ↓   ↑   →   ·
   ·   ·   ·   ·   ·   ·  │  +   -   =   [   ]   \
              L3      DEL
```

### bt レイヤー（Bluetooth）

```
BT_CLR  BT0 BT1 BT2 BT3 BT4 │  ·   ·   ·   ·   ·   ·
STUDIO   ·   ·   ·   ·   ·  │  ·   ·   ·   ·   ·   ·
   ·     ·   ·   ·   ·   ·  │  ·   ·   ·   ·   ·   ·
```

- `BT_CLR` … 現在のプロファイルのペアリング情報を消去
- `BT0`〜`BT4` … Bluetooth プロファイルの切り替え
- `STUDIO` … `&studio_unlock`（ZMK Studio の編集ロック解除）

## ZMK Studio での編集

このキーボードは ZMK Studio に対応しています。USB 接続した状態で [ZMK Studio](https://zmk.studio/) を開き、bt レイヤーの `&studio_unlock` でロックを解除すると、キーマップをその場で編集・反映できます。
