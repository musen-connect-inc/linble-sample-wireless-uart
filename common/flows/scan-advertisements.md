# アドバタイズのスキャン

{docsify-updated}

スキャン実行後、端末に内蔵されているBLEモジュールが空間中のアドバタイズを認識すると、アプリに対してアドバタイズ受信イベントが発生します。

アドバタイズ受信イベント発生の都度、そのアドバタイズデータ内のフィールドを確認します。
それが接続対象のLINBLEからのものだった場合、スキャンを止め、[BLE接続](common/flows/connect-to-target.md)を開始します。

![](../../out/plantuml/sequence_scan_advertisements.png)

!> LINBLEのデフォルト動作では、アドバタイズは100msに1回送出されます。スキャン中は空間中の他のBLEデバイスからのアドバタイズも受信するため、基本的にアドバタイズ受信イベントは大量に発生します。
