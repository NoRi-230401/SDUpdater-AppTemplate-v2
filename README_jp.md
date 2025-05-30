# SDUpdater-AppTemplate-v2
**[`　English　`](README.md)**

## 1. はじめに

このドキュメントは、`SDUpdater-AppTemplate-v2\src\main.cpp` で提供されるM5Stackデバイス向けSDカードアップデーター機能付きアプリケーションテンプレートの使用方法について説明します。

このテンプレートを使用すると、デバイス起動時に特定の操作を行うことで、SDカードに保存された新しいファームウェアやメニュープログラム (`menu.bin`) を簡単にロードして実行できます。

## 2. 対象デバイス

このテンプレートは、以下のM5Stackデバイスに対応しています。

*   M5Stack Cardputer/Cardputer v1.1
*   M5Stack Core2/Core2 for AWS/Core2 v1.1
*   M5Stack CoreS3/CoreS3-SE

vsCodeの下部にある　`Switch PlatformIO Project enviroment`で対象デバイスに切替えてコンパイルしてください。

**他のデバイスにも容易に対応可能と思いますが、現在は確認できていません。**

## 3. 主な機能

*   **デバイス初期化**:
    *   シリアル通信のボーレート設定 (115200 bps)。
    *   ディスプレイの初期化（輝度、向き、背景色、文字サイズ）。
    *   SDカードの初期化。
        *   Cardputer: SPI2インターフェースを使用。
        *   Core2/CoreS3: 標準SPIインターフェース (GPIO_NUM_4) を使用。
    *   SDカード初期化失敗時のエラー表示と処理停止。
*   **SDアップデーターロビー (`SDU_lobby`)**:
    *   デバイス起動時に特定のキーが押されているかを検出します。
        *   **M5Stack Cardputer**: 'a' キー
        *   **M5Stack Core2 / CoreS3**: BtnA (一番左のボタン)
    *   キーが押されている場合、SDカードのルートディレクトリにある `/menu.bin` ファイルをロードし、デバイスを再起動して実行します。
    *   **M5Stack CoreS3 の場合**: ボタンが認識されにくい問題を考慮し、起動後3秒間「Press BtnA to load menu」というメッセージを表示し、その間にBtnAを押すことで `menu.bin` をロードします。

## 4. 使い方

### 4.1. 準備

1.  SDカードのルートディレクトリに、ロードしたいファームウェアまたはメニュープログラムを `menu.bin` という名前で保存します。
    *   このファイルは、M5StackUpdaterライブラリが対応する形式である必要があります。通常はコンパイル済みのバイナリファイルです。

### 4.2. SDアップデーターの起動方法

1.  **M5Stack Cardputer の場合**:
    *   'a' キーを押しながらデバイスの電源を入れるか、リセットボタンを押します。
2.  **M5Stack Core2 の場合**:
    *   BtnA (一番左のボタン) を押しながらデバイスの電源を入れるか、リセットボタンを押します。
3.  **M5Stack CoreS3 の場合**:
    *   デバイスの電源を入れるか、リセットボタンを押します。
    *   ディスプレイに「Press BtnA to load menu」という緑色のメッセージが表示されます。
    *   メッセージが表示されている3秒間の間に BtnA (一番左のボタン) を押します。

上記操作が成功すると、SDカードから `menu.bin` がロードされ、デバイスが再起動して新しいプログラムが実行されます。

### 4.3. 通常起動

上記のキー操作を行わずにデバイスを起動すると、`setup()` 関数内の `SDU_lobby()` が実行された後、`disp("Hello World!");` によってディスプレイに "Hello World!" と表示されます。その後、`loop()` 関数が繰り返し実行されます（現在のテンプレートでは `loop()` は空です）。

## 5. コードの概要

*   **`setup()`**:
    *   `m5stack_begin()`: デバイスの初期化処理を呼び出します。
    *   `SDU_lobby()`: SDアップデーターのロビー機能を呼び出します。
    *   `disp("Hello World!")`: 通常起動時にメッセージを表示します。
*   **`loop()`**:
    *   メインループです。独自のアプリケーションロジックをここに記述します。現在のテンプレートでは空です。
*   **`m5stack_begin()`**:
    *   M5.config() を使用してシリアルボーレートなどを設定します。
    *   Cardputerとその他のデバイス（Core2, CoreS3）でディスプレイとSDカードの初期化処理を分けています。
    *   SDカードの初期化を10回試行し、失敗した場合はエラーメッセージを表示して停止します。
*   **`SDU_lobby()`**:
    *   デバイスの種類に応じて、起動時のキー入力（Cardputer: 'a'キー, その他: BtnA）をチェックします。
    *   CoreS3の場合、ボタン押下を促すメッセージを3秒間表示する特別な処理が含まれています。
    *   キーが押されていれば、`updateFromFS(SD, "/menu.bin")` を呼び出してSDカードからファームウェアをロードし、`ESP.restart()` でデバイスを再起動します。
*   **`disp(String msg)`**:
    *   指定された文字列をディスプレイに表示します。Cardputerとその他のデバイスで、使用するディスプレイオブジェクトが異なります。

## 6. カスタマイズ

*   **独自のアプリケーション開発**:
    *   `loop()` 関数内に、あなたのアプリケーションのメインロジックを記述してください。
    *   必要に応じて、`setup()` 関数の `disp("Hello World!");` 以降に初期化コードを追加できます。
*   **ロードするファイル名の変更**:
    *   `SDU_lobby()` 関数内の `updateFromFS(SD, "/menu.bin");` の第二引数を変更することで、SDカードからロードするファイル名を変更できます。例: `updateFromFS(SD, "/myapp.bin");`
*   **SDカードピン設定の変更 (高度)**:
    *   `m5stack_begin()` 関数内で、SDカードのSPIピン設定や初期化パラメータを変更できますが、通常はデフォルトのままで問題ありません。

## 7. 注意事項

*   SDカードが正しく挿入され、フォーマットされていることを確認してください。
*   `menu.bin` ファイルがSDカードのルートディレクトリに存在することを確認してください。
*   SDカードの初期化に失敗した場合、ディスプレイに「ERR: SD begin ...」と表示され、プログラムは停止します。この場合、SDカードの接続や状態を確認してください。
*   M5Stack CoreS3で `menu.bin` をロードする際は、メッセージ表示中の3秒以内にBtnAを押す必要があります。


## 8. Link

[SDUpdater-AppTemplate-v2](https://github.com/NoRi-230401/SDUpdater-AppTemplate-v2)

[SimpleMenu-SDUpdater](https://github.com/NoRi-230401/SimpleMenu-SDUpdater)

[CardputerSimpleLauncher@shikarunochi](https://github.com/shikarunochi/CardputerSimpleLaucher)

[M5Stack-SD-Updater@tobozo](https://github.com/tobozo/M5Stack-SD-Updater)


