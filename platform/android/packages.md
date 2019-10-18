# Android版サンプルコード - パッケージ構造

{docsify-updated}

## appパッケージ

アプリのエントリポイントとなる`MainActivity`、および、[Runtime Permission制御]( platform/android/launch-app?id=位置情報についてのruntime-permission取得 )の処理を行うための`RuntimePermissionHandler`クラスが配置されています。

## commonパッケージ

**サンプルコードの設計**で記述した

* [BLEとLINBLEの基本制御フロー](common/flows/introduction.md)
* [使用するUART通信仕様](common/command-interface.md)
* [全体構造](common/classes.md)

の設計を、**Android BLE APIを使わずに**そのままKotlinでコーディングした抽象層です。

## concreteパッケージ

commonパッケージのBLE抽象化部分について、**実際のAndroid BLE APIを使用するようにして**具体的に実装した層です。
