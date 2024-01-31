# BLE接続

{docsify-updated}

[スキャン](common/flows/scan-advertisements.md)で発見した機器に対して接続APIを呼び出すと、対象とのパケット交換が行われ、これによって空間上でのBLE接続が確立します。

![](../../out/plantuml/sequence_connection.png)

接続が完了したら、[GATT準備](common/flows/prepare-gatt.md)を行います。

?> このサンプルコードでは、単一のLINBLEとの通信を行う状況を想定しています。<br><br>複数のLINBLEとの通信を制御したい場合は、`BluetoothCentralController`クラスの「Bluetooth状態の監視・スキャン・接続確立・GATT準備」と「実際のデータ通信」が分かれるようにクラス設計を行うと効果的です。
