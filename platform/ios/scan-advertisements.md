# iOS版サンプルコード - アドバタイズのスキャン

{docsify-updated}

> 参考: BLEとLINBLEの基本制御フロー: [アドバタイズのスキャン](common/flows/scan-advertisements.md)
>
> ![](../../out/plantuml/sequence_scan_advertisements.png)

iOSでのBLEスキャンでは、[`CBCentralManager`](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager)の[`scanForPeripherals`](<https://developer.apple.com/documentation/corebluetooth/cbcentralmanager/scanforperipherals(withservices:options:)>)を使用します。

スキャン結果は[`centralManager(_:didDiscover:advertisementData:rssi:)`](<https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/centralmanager(_:diddiscover:advertisementdata:rssi:)>)で渡される[`CBPeripheral`](https://developer.apple.com/documentation/corebluetooth/cbperipheral)オブジェクトや`advertisementData`辞書オブジェクトから取り出すことができます。

`advertisementData`辞書オブジェクトからは、デバイス名やManufacturerDataなどを取得できます。
取得できる内容については[Advertisement Keys](https://developer.apple.com/documentation/corebluetooth/advertisement-data-retrieval-keys)を参照してください。

!> `CBPeripheral`オブジェクトと`advertisementData`オブジェクトの両方からデバイス名を取得することができますが、`CBPeripheral`から取得できるデバイス名はiOS内部でキャッシュされやすく、LINBLEのデバイス名が変更された場合に、古いデバイス名が取得されてしまうことがあります。LINBLEのデバイス名を頻繁に変える使い方を想定している場合、CBAdvertisementDataLocalNameKeyからデバイス名を取得したほうが安全です。
