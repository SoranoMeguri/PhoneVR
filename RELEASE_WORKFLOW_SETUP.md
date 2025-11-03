# GitHub Actions Release Workflow 設定手順書

## 1. 問題の概要

現在、リポジトリのリリース自動化ワークフローが以下のエラーで失敗しています。

```
Error: Input required and not supplied: app_id
```

これは、ワークフローがリリース資産のアップロードなどの操作を行うために必要な`GitHub App`の認証情報（App IDと秘密鍵）を見つけられないことが原因です。

## 2. 解決策の概要

この問題を解決するには、以下の手順で`GitHub App`を作成し、その認証情報をリポジトリの`Secrets`に設定する必要があります。

1.  **GitHub Appを新規作成する**
2.  **Appに必要な権限を設定する**
3.  **Appをリポジトリにインストールする**
4.  **App IDを控える**
5.  **秘密鍵を生成し、控える**
6.  **リポジトリに`Secrets`としてApp IDと秘密鍵を登録する**

以下に、それぞれの詳細な手順を説明します。

## 3. 詳細な設定手順

### ステップ 3.1: GitHub Appの新規作成

1.  リポジトリを所有するGitHubアカウントまたはOrganizationの**[設定]**ページに移動します。
    *   **個人アカウントの場合:** `https://github.com/settings/apps`
    *   **Organizationの場合:** `https://github.com/organizations/YOUR_ORG/settings/apps` ( `YOUR_ORG` はご自身のOrganization名に置き換えてください)
2.  **[New GitHub App]** ボタンをクリックします。
3.  以下の情報を入力します。
    *   **App name:** 任意の名前に設定します（例: `PhoneVR Release Bot`）。
    *   **Homepage URL:** リポジトリのURLを入力します（例: `https://github.com/SoranoMeguri/PhoneVR`）。
    *   **Webhook:** `Webhook`セクションの`Active`のチェックを**外します**。今回の用途では不要です。
4.  **[Repository permissions]** セクションに進みます。

### ステップ 3.2: Appに必要な権限を設定する

`release.yml`ワークフローがリリースを作成し、アセットをアップロードするために、以下の権限が必要です。

1.  **[Repository permissions]** セクションを見つけます。
2.  **Contents:** ドロップダウンメニューから **[Read and write]** を選択します。
    *   これにより、ワークフローがGitタグの作成やリリース資産のアップロードを行えるようになります。
3.  他の権限はすべて`No access`のままで問題ありません。
4.  ページ下部の **[Create GitHub App]** ボタンをクリックして、Appを作成します。

### ステップ 3.3: Appをリポジトリにインストールする

1.  Appを作成すると、そのAppの設定ページが表示されます。
2.  左側のサイドバーから **[Install App]** を選択します。
3.  **[Install]** ボタンをクリックし、`SoranoMeguri/PhoneVR`リポジトリを選択してインストールします。（`All repositories`または`Only select repositories`のどちらかを選び、対象リポジトリへのアクセスを許可してください）

### ステップ 3.4: App IDを控える

1.  Appの設定ページの上部に表示されている **`App ID`** を見つけ、その数値をコピーして安全な場所にメモしてください。（例: `123456`）
    *   これは後でリポジトリのSecretとして設定します。

### ステップ 3.5: 秘密鍵を生成する

1.  Appの設定ページの中程にある **[Private keys]** セクションを見つけます。
2.  **[Generate a private key]** ボタンをクリックします。
3.  キーが生成され、`.pem`という拡張子のファイルが自動的にダウンロードされます。
    *   **【重要】このキーファイルは一度しかダウンロードできません。** 必ず安全な場所に保管してください。

### ステップ 3.6: リポジトリにSecretsを登録する

最後に、取得した`App ID`と`秘密鍵`をリポジトリの暗号化されたSecretsとして登録します。

1.  `SoranoMeguri/PhoneVR`リポジトリの**[Settings]**タブに移動します。
2.  左側のメニューから **[Secrets and variables]** > **[Actions]** を選択します。
3.  **[New repository secret]** ボタンをクリックして、以下の2つのSecretを登録します。

    *   **1つ目のSecret (App ID):**
        *   **Name:** `APP_ID`
        *   **Secret:** ステップ 3.4 で控えた **App ID** の数値を貼り付けます。

    *   **2つ目のSecret (秘密鍵):**
        *   **Name:** `PRIVATE_KEY`
        *   **Secret:** ステップ 3.5 でダウンロードした `.pem` ファイルの内容を**すべて**コピーし、このテキストボックスに貼り付けます。（`-----BEGIN RSA PRIVATE KEY-----`から`-----END RSA PRIVATE KEY-----`まで含めてください）

## 4. 完了

以上の設定が完了すれば、`release.yml`ワークフローは必要な認証情報をSecretsから読み込めるようになり、エラーは解消されるはずです。次回のリリース時にワークフローが正常に動作するかご確認ください。
