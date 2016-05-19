FORMAT: 1A

# Group ユーザ

## ユーザ [/v1/users]

### ユーザ登録 [POST]

#### 処理概要

* ユーザ情報を新しく登録する。
* 登録に成功した場合、アクセストークンを返す。

+ Request (application/json)

    + Headers

            Accept: application/json

    + Attributes
        + email: cocokarafine@classmethod.jp (string, required) - メールアドレス（format: email）
        + password: abc123 (string, required) - パスワード（pattern: ^[0-9A-Za-z]{6,16}$）

+ Response 201 (application/json)

    + Attributes
        + accessToken: f58ba22059f5a8aa8f346e0f40987adab326041fac99029c909bef2c6300821a (string, required) - アクセストークン

+ Response 400 (application/json)

    + Attributes
        + error (required)
            + code: 40000 (number, required)
            + message: The request is invalid. (string, required)

+ Response 409 (application/json)

    + Attributes
        + error (required)
            + code: 40900 (number, required)
            + message: The member already exists. (string, required)

+ Response 500 (application/json)

    + Attributes
        + error (required)
            + code: 50000 (number, required)
            + message: An unknown error occurred. (string, required)

## アプリ会員アクティベート [/v1/app_members/activate]

### アプリ会員アクティベーション [POST]

#### 処理概要

* 有効期限内で指定したPINコードとアクティベーショントークンに一致する仮登録情報があれば、アプリ会員を本登録する。
* デバイス情報がある場合、アプリ会員番号とデバイス情報をデバイス登録ワーカーに登録する。
* 本登録に成功した場合、ファインとスクラッチを付与し、アクセストークンと付与ファイン数、付与スクラッチカード数を返す。

#### レスポンス

| HTTPステータスコード | エラーコード | 説明 |
| --- | --- | --- |
| 201 | - | リクエストは成功。|
| 400 | 40000 | 属性が不正。 |
| 400 | 40002 | スクラッチが最大数（99枚）。 |
| 400 | 40004 | ファインが最大数（9,999,999,999ファイン）。 |
| 404 | 40401 | 指定したアクティベーショントークンに一致する仮登録情報が存在しない。 |
| 404 | 40402 | 指定したPINコードに一致する仮登録情報が存在しない。 |
| 409 | 40900 | 同じメールアドレスのアプリ会員が既に存在する。 |
| 500 | 50000 | サーバー内部エラー。 |

+ Request (application/json)

    + Headers

            Accept: application/json

    + Attributes
        + pinCode: 123456 (string, required) - PINコード（pattern: ^[0-9]{6}$）
        + activationToken: f58ba22059f5a8aa8f346e0f40987adab326041fac99029c909bef2c6300821a (string, required) - アクティベーショントークン
        + deviceToken: 0f744707bebcf74f9b7c25d48e3358945f6aa01da5ddb387462c7eaf61bbad78 (string) - デバイストークン
        + platform: apns (enum) - モバイルプラットフォーム（APNs/GCM）
            + apns (string)
            + gcm (string)

+ Response 201 (application/json)

    + Attributes
        + accessToken: cf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3e (string, required) - アクセストークン
        + appMemberId: 200001000000003 (number, required) - アプリ会員番号（minimum: 200001000000000、maximum: 200001999999999）
        + presents (array, required) - アプリ会員登録特典プレゼントの配列（minItems: 0、maxItems: 3）
            + present
                + type (enum, required) - プレゼント種別（fine: ファイン、premium: プレミアムスクラッチ、normal: ノーマルスクラッチ）
                    + fine (string)
                    + premium (string)
                    + normal (string)
                + value: 1000 (number, required) - プレゼント数（プレゼント種別がファインの場合はファイン数、、minimum = 0、maximum: 9999999999、スクラッチの場合はスクラッチ数、minimum: 0、maximum: 99）

+ Response 400 (application/json)

    + Attributes
        + error (required)
            + code: 40000 (number, required)
            + message: The request is invalid. (string, required)

+ Response 404 (application/json)

    + Attributes
        + error (required)
            + code: 40401 (enum, required)
                + 40401 (number)
                + 40402 (number)
            + message: Authentication is required. (enum, required)
                + The temporary registration information is not found. (string)
                + The temporary registration information is not found. (string)

+ Response 409 (application/json)

    + Attributes
        + error (required)
            + code: 40900 (number, required)
            + message: The member already exists. (string, required)

+ Response 500 (application/json)

    + Attributes
        + error (required)
            + code: 50000 (number, required)
            + message: An unknown error occurred. (string, required)

## メールアドレス更新 [/v1/app_members/me/email]

### メールアドレス更新 [PUT]

#### 処理概要

* 指定したパスワードが正しい場合、現在のメールアドレスから確認トークンを生成し、指定したメールアドレスを仮登録する。
* 指定したメールアドレスに確認リンクを含むメールアドレス更新確認メールを送信する。
* 仮登録情報の有効期限を24時間とする。

#### レスポンス

| HTTPステータスコード | エラーコード | 説明 |
| --- | --- | --- |
| 202 | - | リクエストは成功。|
| 400 | 40000 | 属性が不正。 |
| 401 | 40100 | 認証が必要。 |
| 401 | 40101 | アクセストークンが不正。 |
| 401 | 40102 | ログインが失敗（指定したパスワードが不正）。 |
| 409 | 40900 | 同じメールアドレスのアプリ会員が既に存在する。 |
| 500 | 50000 | サーバー内部エラー。 |

+ Request (application/json)

    + Headers

            Accept: application/json
            Authorization: Bearer cf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3e

    + Attributes
        + email: cocokarafine@classmethod.jp (string, required) - メールアドレス（format: email）
        + password: abc123 (string, required) - パスワード（pattern: ^[0-9A-Za-z]{6,16}$）

+ Response 202 (application/json)

        {}

+ Response 400 (application/json)

    + Attributes
        + error (required)
            + code: 40000 (number, required)
            + message: The request is invalid. (string, required)

+ Response 401 (application/json)

    + Attributes
        + error (required)
            + code: 40100 (enum, required)
                + 40100 (number)
                + 40101 (number)
                + 40102 (number)
            + message: Authentication is required. (enum, required)
                + Authentication is required. (string)
                + The access token is invalid. (string)
                + Login failed. (string)

+ Response 409 (application/json)

    + Attributes
        + error (required)
            + code: 40900 (number, required)
            + message: The member already exists. (string, required)

+ Response 500 (application/json)

    + Attributes
        + error (required)
            + code: 50000 (number, required)
            + message: An unknown error occurred. (string, required)

## メールアドレス更新確認 [/v1/app_members/email/confirm/{confirmationToken}]

### メールアドレス更新確認 [GET]

#### 処理概要

* メールアドレス更新確認メールに記載された確認リンクから呼び出される。
* 有効期限内で確認トークンに一致する仮登録情報があれば、メールアドレスを更新し、顧客基盤同期ワーカーに登録する。

#### レスポンス

| HTTPステータスコード | エラーコード | 説明 |
| --- | --- | --- |
| 200 | - | リクエストは成功。|
| 403 | - | パラメータが不正。<br>指定した確認トークンに一致する仮登録情報が存在しない。<br>同じメールアドレスのアプリ会員が既に存在する。 |
| 500 | - | サーバー内部エラー。 |

+ Parameters

    + confirmationToken: f58ba22059f5a8aa8f346e0f40987adab326041fac99029c909bef2c6300821a (string, required) - 確認トークン

+ Response 200 (text/html)

+ Response 403 (text/html)

+ Response 500 (text/html)

## 会員連携 [/v1/app_members/me/integrate]

### 会員連携 [PUT]

#### 処理概要

* 指定したカード会員とアプリ会員を融合する。
* 融合前に誕生月、誕生日で本人確認を行う。
* 融合に成功した場合、ファインとスクラッチを付与し、利用可能なココカラポイント、付与ファイン数、付与スクラッチカード数を返す。

#### レスポンス

| HTTPステータスコード | エラーコード | 説明 |
| --- | --- | --- |
| 200 | - | リクエストは成功。 |
| 400 | 40000 | 属性が不正。 |
| 400 | 40002 | スクラッチが最大数（99枚）。 |
| 400 | 40004 | ファインが最大数（9,999,999,999ファイン）。 |
| 401 | 40100 | 認証が必要。 |
| 401 | 40101 | アクセストークンが不正。 |
| 403 | 40301 | アプリ会員番号のステータスが無効（復旧不可）。 |
| 403 | 40302 | カード会員番号のステータスが無効（復旧不可）。 |
| 403 | 40304 | カード会員の入会店舗がFC店、または統合カード指定エラー（LFカード）。 |
| 404 | 40403 | 指定したカード会員番号と誕生月、誕生日に一致するカード会員が存在しない（静態未登録）。 |
| 404 | 40404 | 指定したカード会員番号と誕生月、誕生日に一致するカード会員が存在しない（静態不足）。 |
| 404 | 40405 | 指定したカード会員番号と誕生月、誕生日に一致するカード会員が存在しない（静態不一致）。 |
| 409 | 40901 | 指定したカード会員が既に融合済み（顧客基盤ステータス64）。 |
| 409 | 40902 | 指定したアプリ会員が既に融合済み（顧客基盤ステータス68、または69）。 |
| 500 | 50000 | サーバー内部エラー。 |

+ Request (application/json)

    + Headers

            Accept: application/json
            Authorization: Bearer cf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3e

    + Attributes
        + birthdate (required)
            + year: 1980 (number, required) - 誕生年（minimum: 1, maximum: 9999）
            + month: 1 (number, required) - 誕生月（minimum: 1, maximum: 12）
            + day: 1 (number, required) - 誕生日（minimum: 1, maximum: 31）
        + cardMember (required)
            + id: 200000000000000 (number, required) - カード会員番号（minimum: 0、maximum: 299999999999999）
            + distributorCode: 1 (enum, required) - 販社コード（1: セイジョー、2: セガミ、3: ジップ、5: スズラン、9: CF）
                + 1 (number)
                + 2 (number)
                + 3 (number)
                + 5 (number)
                + 9 (number)

+ Response 200 (application/json)

    + Attributes
        + point: 1500 (number, required) - 利用可能ポイント（minimum: -9999999999、maximum: 9999999999）
        + presents (array, required) - 会員連携特典プレゼントの配列（minItems: 0、maxItems: 3）
            + present
                + type (enum, required) - プレゼント種別（fine: ファイン、premium: プレミアムスクラッチ、normal: ノーマルスクラッチ）
                    + fine (string)
                    + premium (string)
                    + normal (string)
                + value: 5000 (number, required) - プレゼント数（プレゼント種別がファインの場合はファイン数、、minimum = 0、maximum: 9999999999、スクラッチの場合はスクラッチ数、minimum: 0、maximum: 99）

+ Response 400 (application/json)

    + Attributes
        + error (required)
            + code: 40000 (number, required)
            + message: The request is invalid. (string, required)

+ Response 401 (application/json)

    + Attributes
        + error (required)
            + code: 40100 (enum, required)
                + 40100 (number)
                + 40101 (number)
            + message: Authentication is required. (enum, required)
                + Authentication is required. (string)
                + The access token is invalid. (string)

+ Response 403 (application/json)

    + Attributes
        + error (required)
            + code: 40301 (enum, required)
                + 40301 (number)
                + 40302 (number)
                + 40304 (number)
            + message: The app member number is invalid. (enum, required)
                + The app member number is invalid. (string)
                + The card member number is invalid. (string)
                + The card can not be used. (string)

+ Response 404 (application/json)

    + Attributes
        + error (required)
            + code: 40403 (enum, required)
                + 40403 (number)
                + 40404 (number)
                + 40405 (number)
            + message: The card member is not found. (string, required)

+ Response 409 (application/json)

    + Attributes
        + error (required)
            + code: 40901 (enum, required)
                + 40901 (number)
                + 40902 (number)
            + message: The card member is already integrated. (enum, required)
                + The card member is already integrated. (string)
                + The app member is already integrated. (string)

+ Response 500 (application/json)

    + Attributes
        + error (required)
            + code: 50000 (number, required)
            + message: An unknown error occurred. (string, required)
