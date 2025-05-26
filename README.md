# radigo_relocation

Radigoで録音したファイルをGoogle Driveの所定の場所に移動するスクリプトです。

## 要件

- Python 3 以上
- pydrive2 ライブラリ

## 事前準備

1.  **必要なライブラリをインストールします。**

    ```sh
    pip install pydrive2
    ```

2.  **Google Cloud Platform で OAuth 2.0 クライアント ID を作成します。**
    *   [Google Cloud Console](https://console.cloud.google.com/) にアクセスします。
    *   新しいプロジェクトを作成するか、既存のプロジェクトを選択します。
    *   「APIとサービス」 > 「認証情報」に移動します。
    *   「認証情報を作成」 > 「OAuth クライアント ID」を選択します。
    *   アプリケーションの種類として「デスクトップアプリ」を選択し、名前を付けます。
    *   作成後、JSON ファイルをダウンロードし、`client_secrets.json` という名前でこのプロジェクトのルートディレクトリに保存します。

3.  **設定ファイルを準備します。**
    *   `settings.yaml.sample` をコピーして `settings.yaml` を作成します。
    *   `settings.yaml` 内の各項目を以下のように設定します。
        *   `client_config_file`: 上記で準備した `client_secrets.json` のパスを指定します (通常は `client_secrets.json` のままで問題ありません)。
        *   `output_folder_id`: Google Drive 上で、Radigo の録音ファイルが保存されているフォルダの ID を指定します。このフォルダ内のファイルが整理対象となります。フォルダ ID は、Google Drive で該当フォルダを開いた際の URL (`https://drive.google.com/drive/folders/ここに表示されるID`) から取得できます。
        *   `target_folder_path`: Google Drive 内での基本的な保存先パスを指定します。デフォルトは `audio/radio/` です。このパスの配下に、放送局ごとのフォルダが作成されます。
        *   必要に応じて、`save_credentials_file` や `get_refresh_token` の設定を行ってください。

## 実行方法

以下のコマンドでスクリプトを実行します。

```sh
python radio_furiwake.py
```

### オプション

-   `--dry-run`: ドライランモードで実行します。
    このモードでは、実際にファイルは移動されず、ログに移動予定のファイルとフォルダが記録されます。
-   `--max-retries <回数>`: Google Drive とのセッションが切れた場合の再試行の最大回数を指定します。デフォルトは `3` 回です。

## GitHub Actions による自動実行

このリポジトリには、`Move_Radigo-Files` という名前の GitHub Actions ワークフローが含まれており、スクリプトを自動実行することができます。

### 実行トリガー

-   **手動実行 (`workflow_dispatch`)**: GitHub の Actions タブから手動でワークフローを実行できます。
    -   `dry_run` パラメータ: 手動実行時に `true` を指定すると、ドライランモードで実行されます。デフォルトは `false` です。
-   **スケジュール実行 (コメントアウト中)**: `.github/workflows/move-radigo-files.yml` ファイル内のスケジュール実行の設定を有効にすることで、定期的にスクリプトを実行することも可能です (例: 毎日午前9時 JST)。

### 必要な Secrets

GitHub Actions でスクリプトを実行するためには、以下の Secrets をリポジトリに設定する必要があります。

-   `GOOGLE_CLIENT_ID`: Google Cloud Platform で作成した OAuth 2.0 クライアント ID。
-   `GOOGLE_CLIENT_SECRET`: Google Cloud Platform で作成した OAuth 2.0 クライアントシークレット。
-   `OUTPUT_FOLDER_ID`: Radigo の録音ファイルが保存されている Google Drive フォルダの ID。
-   `TARGET_FOLDER_PATH`: Google Drive 内での基本的な保存先パス。
-   `GOOGLE_CREDENTIALS_JSON`: Google Drive API の認証に使用する `credentials.json` の内容。これは、初回ローカル実行時に生成される `credentials.json` の中身をコピーして設定します。

これらの Secrets は、リポジトリの「Settings」 > 「Secrets and variables」 > 「Actions」から設定できます。

## ログ

スクリプトの実行中に発生したイベントは、ログファイルに記録されます。
ログファイルは、スクリプト実行時に `logs` ディレクトリ配下に `log_YYYYMMDDHHMMSS.txt` (重複時は `log_YYYYMMDDHHMMSS-XXX.txt`) という形式で保存されます。

## エラー処理

-   スクリプト実行中にエラーが発生した場合、エラーメッセージがログファイルに記録されます。
-   Google Drive とのセッションが切れた場合、指定された回数まで自動的にセッションの再取得を試みます。

## 注意事項

-   初回実行時には、ブラウザが起動し Google アカウントでの認証が求められます。認証を許可してください。
-   `client_secrets.json`、`settings.yaml` および `credentials.json` には認証情報が保存されるため、これらのファイルの取り扱いには十分注意してください。特に、Gitリポジトリに誤ってコミットしないように `.gitignore` に追加することを推奨します。
