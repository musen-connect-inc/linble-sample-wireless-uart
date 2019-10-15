# BLE接続

{docsify-updated}

[スキャン](common/flows/scan-advertisements.md)で発見した機器に対して接続APIを呼び出すと、対象とのパケット交換が行われ、これによって空間上でのBLE接続が確立します。

接続が完了したらOSからのコールバックが発生するので、このタイミングで`Linble`クラスのオブジェクトを生成し、ここにBLE制御に必要なリソース（切断制御用のオブジェクト、および、[GATTのキャラクタリスティックオブジェクト](common/flows/prepare-gatt.md)）を集約していきます。

?> `BluetoothCentralController`クラスはBluetooth状態の監視・スキャン・接続確立についての責務、`Linble`クラスは接続済みデバイスの制御についての責務を担当します。この`Linble`クラスのような**BLE制御リソース集約役**を用意しておくと、複数のPeripheralデバイスとの通信を管理する際に役立ちます。

![](../../out/plantuml/sequence_connection.png)
