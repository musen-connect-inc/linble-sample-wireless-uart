# Android版サンプルコード - パッケージ構造

{docsify-updated}

## appパッケージ

アプリのエントリポイントとなる`MainActivity`、および、[Runtime Permission制御]( platform/android/launch-app?id=位置情報についてのruntime-permission取得 )の処理を行うための`RuntimePermissionHandler`クラスが配置されています。

## modelパッケージ

**サンプルコードの設計**で記述した

* [BLEとLINBLEの基本制御フロー](common/flows/introduction.md)
* [使用するUART通信仕様](common/command-interface.md)
* [全体構造](common/classes.md)

の設計を、Android BLE APIを使ってKotlinでコーディングした層です。

