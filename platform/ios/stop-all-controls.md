# iOS版サンプルコード - BLE制御の終了

{docsify-updated}

> 参考: BLEとLINBLEの基本制御フロー: [BLE制御の終了](common/flows/stop-all-controls.md)
>
> ![](../../out/plantuml/sequence_stop.png)

BLEスキャンを停止するためには、[`CBCentralManager.stopScan()`](<https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/stopscan()>)を呼び出します。

BLE通信の切断をするためには、[`CBCentralManager.cancelPeripheralConnection()`](<https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/cancelperipheralconnection(_:)>)を呼び出します。
