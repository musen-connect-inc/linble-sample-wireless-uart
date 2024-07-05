# iOS版サンプルコード - 前書き

{docsify-updated}

本資料では、iOS版サンプルコードである[linble-sample-wireless-uart-ios](https://github.com/musen-connect-inc/linble-sample-wireless-uart-ios)について、その構造や要点について解説しています。

- 元となる設計情報については[BLEとLINBLEの基本制御フロー](common/flows/introduction.md)を参照してください。

- サンプルコード内のコメントはBLE制御をする上で重要になること・分かりづらいところについてのみ記載しています。

- BLE APIの使い方について紹介するのが本サンプルコードの目的となっています。iOSアプリ自体の基本的な開発方法に関しては解説は省略します。

  - UIの構築には[SwiftUI](https://developer.apple.com/jp/xcode/swiftui/)を使用しています。
  - 特に、モデルデータの管理には、`State`や`Observable`などの[SwiftUIの状態管理手法](https://developer.apple.com/documentation/swiftui/model-data)を使用していることに注意してください。

- 実際に参考にすべきコードは[`linble-sample-wireless-uart-ios/LinbleSampleWirelessUart/`](https://github.com/musen-connect-inc/linble-sample-wireless-uart-ios/tree/main/LinbleSampleWirelessUart)内に配置されています。

## 接続対象デバイス名の変更

アプリをビルドする前に、[`WirelessUartController`クラス内の`targetDeviceName`の文字列](https://github.com/musen-connect-inc/linble-sample-wireless-uart-ios/blob/master/LinbleSampleWirelessUart/Model/WirelessUartController.swift#L10)を適切に変更してください。

```swift
static let targetDeviceName: String = "LINBLE-Z1"
// `BTLX` コマンドで確認できるデバイス名をここに貼り付けてください。
```

この文字列は接続対象のデバイス名として使用されます。
コメントでも書いている通り、`BTLX`コマンドによって取得できた文字列をここにセットしてください。

AndroidではBDアドレスを指定していましたが、iOSではBDアドレスを取得することができず、代わりにiOS端末ごとに異なる値になる一意のidentifierが振られるようになっています。
identifierは実際にデバイスをスキャンしてみないとわからないので、ここではデバイス名を使って接続先を制御しています。
周囲に複数のLINBLEがある場合、区別しやすくするために一意のデバイス名を設定するのがおすすめです。
