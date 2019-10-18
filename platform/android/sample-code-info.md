# Android版サンプルコード - 前書き

{docsify-updated}

本資料では、Android版サンプルコードである[linble-sample-wireless-uart-android]( https://github.com/musen-connect-inc/linble-sample-wireless-uart-android )について、その構造や要点について解説しています。

* 元となる設計情報については[BLEとLINBLEの基本制御フロー](common/flows/introduction.md)を参照してください。

* サンプルコード内のコメントはBLE制御をする上で重要になること・分かりづらいところについてのみ記載しています。

* BLE APIの使い方について紹介するのが本サンプルコードの目的となっています。Androidアプリ自体の基本的な開発方法に関しては解説は省略します。
    * 特に、`Acitivity`まわりには[Android Jetpack]( https://developer.android.com/jetpack?hl=JA )の[ViewModel]( https://developer.android.com/topic/libraries/architecture/viewmodel?hl=JA )・[LiveData]( https://developer.android.com/topic/libraries/architecture/livedata?hl=JA )を使用していることに注意してください。

* 実際に参考にすべきコードは[`app/src/main/java/com.musenconnect.linble.sample.wirelessuart.android/`]( https://github.com/musen-connect-inc/linble-sample-wireless-uart-android/tree/master/app/src/main/java/com/musenconnect/linble/sample/wirelessuart/android )内に配置されています。

## 接続対象BDアドレスの変更

アプリをビルドする前に、[`WirelessUartController`クラス内の`defaultTargetLinbleAddress`の文字列]( https://github.com/musen-connect-inc/linble-sample-wireless-uart-android/blob/master/app/src/main/java/com/musenconnect/linble/sample/wirelessuart/android/common/WirelessUartController.kt#L16 )を適切に変更してください。

```kotlin
companion object {
    val defaultTargetLinbleAddress: String = "FFFFFFFFFFFF" // `BTM` コマンドで確認できるBDアドレス文字列をここに貼り付けてください。
        .insertColons()
}
```

この文字列は接続対象のBDアドレスとして使用されます。
コメントでも書いている通り、`BTM`コマンドによって取得できた文字列をここにセットしてください。
