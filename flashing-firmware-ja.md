# ファームウェアの更新手順

GitHub Actions でビルドしたファームウェアを、Kinesis Advantage 360 Professional に書き込む手順です。

参考資料:

- [README.md の「Flashing firmware」](README.md#flashing-firmware)
- [Kinesis 公式: Firmware Update Instructions (PDF)](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-Firmware-Update-Instructions-9.5.24-KB360-PRO.pdf)
- [Kinesis 公式: Settings Reset Instructions (PDF)](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-Settings-Reset-Instructions-11.22.23-KB360-PRO-GBR.pdf)
- [Kinesis 公式サポートページ](https://kinesis-ergo.com/support/kb360pro/#firmware-updates)

## ショートバージョン

[README.md の「Flashing firmware」](README.md#flashing-firmware) の Overview の日本語訳です。慣れている場合はこれだけで十分です。詳しい手順は「1. ファームウェアをダウンロードする」以降を見てください。

書き込みには [Quick Start Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf) の 8 ページにあるプログラミング手順に従ってください。

1. GitHub のビルドジョブからダウンロードしたアーカイブ (クラウドでビルドした場合) か、firmware フォルダー (ローカルでビルドした場合) からファームウェアを取り出します。
1. 左手側を USB でつなぎます。
1. Mod + Macro1 を押して左手側をブートローダーモードにします。USB ドライブとしてパソコンにつながるはずです。
1. `left.uf2` を USB ドライブにコピーします。コピーするとドライブは切断されます。
1. 左右両方の電源を切ります (USB を外し、スイッチがオフになっていることを確認します)。
1. 左手側をスイッチで起動します。
1. 右手側を USB でつないで電源を入れます。
1. Mod + Macro3 を押して右手側をブートローダーモードにし、USB ドライブとしてつなぎます。
1. `right.uf2` をマウントされたドライブにコピーします。
1. 右手側の USB を外し、電源を入れ直します。
1. 完了です！

> [!NOTE]
> 左右それぞれに物理的なリセットボタンもあり、ブートローダーモードに入るときと抜けるときに使えます。ボタンの位置は [User Manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-ZMK-KB360-PRO-Users-Manual-v3-10-23.pdf) の 9 ページ、2.7 節に、使い方は 14 ページ、5.9 節に書かれています。

> [!NOTE]
> OS によっては、settings-reset ファイルを書き込んだ後にドライブが取り出されたものとして扱われなかったり、意味のないエラーが表示されたりすることがあります。これは書き込みが失敗したという意味ではありません。

## 1. ファームウェアをダウンロードする

GitHub に push すると、`.github/workflows/build.yml` の「Build」ワークフローが動き、次の 2 種類の Artifact ができます。

| Artifact 名 | 内容 |
|---|---|
| `firmware-clique` | ZMK Studio (Kinesis の Clique) に対応した版。左側だけ `CONFIG_ZMK_STUDIO=y` でビルドされます。 |
| `firmware-no-clique` | ZMK Studio を含まない従来版。 |

Clique を使っている (キーボードが Clique 対応のファームウェアになっている) なら `firmware-clique` を選んでください。

### ブラウザーでダウンロードする場合

1. GitHub のリポジトリーを開き、「Actions」タブを選びます。
2. 書き込みたいコミットの「Build」の実行結果を開きます。
3. ページ下部の「Artifacts」から、使う方の Artifact をダウンロードします。
4. ダウンロードした zip を展開します。

### `gh` コマンドでダウンロードする場合

```sh
# 最新の実行結果を確認する
gh run list --workflow build.yml --limit 5

# Artifact をダウンロードする (<run-id> は上の結果から選ぶ)
gh run download <run-id> -n firmware-clique -D firmware/
```

展開すると、次の 2 つのファイルが入っています。

```
<日時>-<コミット>-left.uf2   ← 左手側用
<日時>-<コミット>-right.uf2  ← 右手側用
```

ファイル名の `<日時>-<コミット>` で、どのコミットからビルドしたものか確認できます。左右のファイルは必ず同じビルドのものを組み合わせて使います。

## 2. 左手側に書き込む

1. 右手側の電源スイッチを切り、USB ケーブルも外しておきます。
2. 左手側を USB ケーブルでパソコンにつなぎます。
3. 左手側をブートローダーモードにします。次のどちらかの方法を使います。
   - **キー操作:** Mod キーを押しながら、左手 2 段目 (Tab の段) のいちばん内側のキーを押します (README では「Mod + Macro1」と書かれています。このリポジトリーでは Mod レイヤーのこの位置に `&bootloader` が割り当てられています)。
   - **リセットボタン:** 左手側のブートローダーボタンを、クリップなどで素早く 2 回押します。
4. LED が白く点滅してから緑の点灯に変わり、パソコンに `ADV360PRO` というドライブが現れます。
5. `...-left.uf2` を `ADV360PRO` ドライブにコピーします。
6. LED が青く点滅して書き込みが始まり、終わるとドライブは自動で取り外されます。
7. 左手側の電源スイッチを入れ、USB ケーブルを外します。

> [!NOTE]
> macOS では「ディスクの不正な取り出し」の警告が出ることがありますが、無視して構いません。ファイルのコピーでエラーが表示されても、書き込みは成功していることがほとんどです。

## 3. 右手側に書き込む

左手側がスリープしないうちに (30 秒程度) 手早く進めます。スリープしそうなら、左手側の Fn キーを軽く押すとタイマーがリセットされます。

1. 右手側を USB ケーブルでパソコンにつなぎます。
2. 右手側をブートローダーモードにします。
   - **キー操作:** Mod キーを押しながら、右手 2 段目のいちばん内側のキーを押します (README では「Mod + Macro3」)。
   - **リセットボタン:** 右手側のブートローダーボタンを素早く 2 回押します。
3. `ADV360PRO` ドライブが現れたら、`...-right.uf2` をコピーします。
4. 青い点滅が終わったら USB ケーブルを外し、右手側の電源スイッチを入れます。

## 4. 動作を確認する

1. Mod キーを押したまま、左右のレイヤー LED が両方とも緑に点灯することを確認します。
2. テキストエディターを開き、Mod + V を押します。ファームウェアのビルド日時・ブランチ・コミットが入力されるので、書き込んだビルドと一致しているか確認します。

## 5. Bluetooth を接続し直す (必要な場合)

HID の設定 (たとえば `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT`) を変えたファームウェアを書き込んだ後は、パソコンが古いキーボード情報を覚えたままで、一部のキーが効かないことがあります。その場合は次の手順でペアリングをやり直します。

1. キーボード側で、使っているプロファイルの接続情報を消します (Mod レイヤーの `&bt BT_CLR`。Kinesis の初期設定では Mod + Windows キー)。
2. パソコンの Bluetooth 設定で「Adv360Pro」を削除します。
3. もう一度ペアリングします。

USB でつないで使う場合は、ケーブルを一度抜き差しすれば十分です。

## うまくいかないとき

- **キーボードが反応しない:** 左右の電源を何度か入れ直します。電源は必ず**左手側を先に入れ、左手側を最後に切ります**。
- **右手側の LED が赤く点滅する (左右が同期しない) :** 左右に Settings Reset ファイルを書き込んでから、もう一度ファームウェアを書き込みます。
  1. [Kinesis のサポートページ](https://kinesis-ergo.com/support/kb360pro/#firmware-updates)から Settings Reset ファイル (`.uf2`) をダウンロードします。
  2. 左手側をブートローダーモードにして、Settings Reset ファイルをコピーします。完了するとドライブが一度閉じて開き直し、LED が緑の点灯になります。
  3. 左手側の電源を切って USB を外し、右手側も同じように Settings Reset ファイルを書き込みます。
  4. リセット後はキー入力もキー操作でのブートローダー起動もできないので、この文書の「2. 左手側に書き込む」以降をリセットボタンで行い、ファームウェアを書き込み直します。

## 注意点

- 書き込みは USB 接続でしかできません。
- 左右のドライブを同時に開かないでください。どちらのドライブか区別がつかず、左右を取り違えるおそれがあります。
- LED が青く点滅している間 (書き込み中) は、USB ケーブルを抜かないでください。
- ドライブが開いている間は、キー入力やキー操作によるコマンドが効きません。
