FORMAT: 1A

# Group メッセージ

## メッセージ一覧 [/v1/messages]

### メッセージ一覧取得 [GET]

#### 処理概要

* メッセージ一覧を取得する。

+ Request (application/json)

    + Headers

            Accept: application/json

+ Response 200 (application/json)

    + Attributes
        + message (array)
            + (object)
                + id: 22345 (number, required)
                + title: 今日の献立 (string, required)
                + body: ハンバーグとサラダ (string, required)
