---
layout: post
title: "World of Tanks: Mod Packages ver. 0.1"
excerpt: "公式ドキュメント mod packages ver. 0.1 の翻訳"
date: 2017-05-19 17:00 +0900
last_modified_at: 2017-08-03 21:10 +0900
---
この文書は古いバージョンの仕様書を元にしています。
最新の仕様は [公式ドキュメントの翻訳](./#公式ドキュメントの翻訳) を参照してください。

(原文: [mod packages v0.1.pdf](../resources/mod_packages_v0.1.pdf))

World of Tanks の次期アップデートでは、mod パッケージという新しい手法が実装される予定です。

## これは何?
パッケージとは特定の mod のすべての中身を一つのファイルにまとめる手法です。

## なぜ必要なの?
現在 WoT の mod はファイルが `/res_mods/<WoT_version>` にインストールされます。
異なる mod のファイルが同じフォルダにあると、
どのファイルがどの mod のものかを特定することが難しくなることがよくあります。
パッケージの方式を変更することで、非常に簡単に mod ファイルをまとめることができます:
ユーザがパッケージをインストールするにはフォルダ `/mods/<WoT_version>` にコピーするだけでよく、
アンインストールするにはそのファイルを削除するだけです。

## パッケージはどうやって作成するの?
パッケージは非圧縮の *zip* ファイルで、拡張子を `.wotmod` としたものです。
*圧縮された zip ファイルは現在のバージョンではサポートされていないことに注意してください。
そのため、アーカイブを作成するときにオプション「圧縮レベル」を「非圧縮/Store」に設定する必要があります*

### パッケージの内部:
+ *必須:* `res` フォルダ。mod のリソースの置き場で、
これまで `res_mods/<WoT_version>` に置いていたファイルがここに置かれます
+ *推奨:* `meta.xml` (下記を参照)
+ *オプション:* mod 作者が必要としたその他のコンテンツ。readme やホームページの URL、説明書など

### 例:

    res/
    meta.xml
    readme.txt


## どうやってインストールするの?

パッケージは `<WoT_game_foler>/mods/<WoT_version>` にインストールします。
手動でコピーするか、mod や modpack に付属のインストーラを使います。

必要があれば、パッケージは modpack の作者によって mod の種類などのサブフォルダにまとめることができます。

## パッケージの命名規則
パッケージの名前に厳格な制限はありませんが、
配布の都合上、以下の推奨事項に従っておいた方がよいです。

+ 英数字とアンガースコアのみを使用
+ 各 mod は mod の名前と作者名からなるユニークな識別子を登録
+ 名前に mod のバージョンを含める

### 例:

    Mods/
        0.9.17.0.2/
            MultiHitLog_2.8.wotmod
            DamagePanel/
                Some_common_library_3.14.5.wotmod
                DamagePanel_2.6.wotmod
                DamagePanel_2.8.wotmod
                DamagePanel_2.8_patch1.wotmod

## ファイル `meta.xml`
省略可能なファイル `meta.xml` は mod に関する情報をオプションのフィールドに含みます。

```xml
<root>
    <!-- Techical MOD ID -->
    <id>DamagePanel</id>
    <!-- Package version -->
    <version>0.2.8</version>
    <!-- Human readable name -->
    <name>Damage Panel</name>
    <!-- Human readable description -->
    <description>New cool Damage Panel with feature1 ..... N</description>
</root>
```

これらのフィールドの値は、将来 mod 制御システムで使用される予定です。

## パッケージの読み込み順序は?
クライアントが起動すると、フォルダ `mods/` にインストールされているすべてのパッケージが
アルファベット順に並び替えられ A から Z に順に読み込まれます。 

必要に応じて (例えば modpack を提供する場合)、手動で読み込み順を制御することができます。
そのためには、パッケージのルートフォルダ `mods/<WoT_version>/` に
オプションの `load_order.txt` ファイルを置きます。

フォーマットは、プレーンテキスト; UTF8; CR+LF です。
このファイルは mod の別の読み込み順を指定することができます。
一行ごとに相対パスを記述します:

### 例:

    zzzz.wotmod
    bbbb.wotmod
    aaaa.wotmod

`load_order.txt` で指定したパッケージは、最初にファイル中で指定した順に読み込まれます。
その後で、通常の方法で残りのパッケージが読み込まれます。

## パッケージの競合解決


パッケージの競合の解決

通常、パッケージシステムは異なるパッケージの `res/` に同じ名前のファイルが存在することを許可しません。
こうした状況は競合とみなされます。
競合が発生すると、ユーザに競合が通知されるともに、パッケージは読み込みから完全に除外されます。

例えば、`res/scripts/entities.xml`　が `a.wotmod` と `b.wotmod` の両方に含まれている場合、
`a.wotmod` が正常に読み込まれ、競合する `b.wotmod` は読み込まれません。

競合の処理は以下が可能です:

1. ファイル `load_order.txt`。
このファイルで指定されているパッケージは競合とみなされず、名前の衝突を検査せずに読み込まれます。
パッケージの挙動を理解した上で `load_order.txt` を設置したと想定しているためです。
2. `meta.xml` のノード `<id>` の値。
`<id>` が同一の場合、同じ mod の異なるバージョンまたはパッチとみなし、競合は判定されません。

異なるパッケージに同名のファイルが存在する場合には、`load_order.txt` か `meta.xml` で競合を解決し、
最後のパッケージからファイルが読み込まれることになります。
