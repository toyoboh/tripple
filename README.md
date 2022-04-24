# Tripple

## 概要
---
- カップル（家族）で旅行に行った場所を思い出として記録する
- 他のユーザの旅行プランを参考にできる

## 基本機能
---
- ログイン、ゲストログイン（自動ログイン）
- ログアウト
- ユーザ登録
- 思い出登録
- 思い出編集
- 思い出削除
- 思い出検索

## 機能詳細
---
### ログイン、ゲストログイン（自動ログイン）
#### ●概要
- ログインにはメールアドレス（もしくはユーザID）とパスワードを使用する
- 自動ログイン機能は32文字のトークンをCookieとDBで管理することで実装する  
なおトークンは自動ログインする度に更新する（Cookieはトークン値の更新、DBは前回のトークン値の削除と新規トークン値の登録）
- なお、自動ログイン機能の有無については、ログイン時にユーザが決められるものとする（ログインフォームにチェックボックスを付ける）

#### ●ログイン時の処理の流れ
  1. ユーザ情報の受け取り
  2. バリデーションチェック
      - 情報不足または条件を満たしていない場合は、エラー情報を記載したログインフォームを返す
  3. ユーザの情報をDBから取得
      - メールアドレス（またはユーザID）をもとにユーザの情報を取得  
      ユーザ情報がない場合は、エラー情報を記載したログインフォームを返す
  4. パスワードの一致確認
      - ユーザが入力したパスワードと項番3で取得したパスワードが一致するか検証  
      一致する場合は、ホーム画面（未定）を表示  
      一致しない場合は、エラー情報を記載したログインフォームを返す

#### ●ログイン済みかどうかに関して、以下のパターンで処理をそれぞれ分岐する
- Session情報も自動ログイントークン（Cookie）も存在しない場合
  - ログイン画面へリダイレクトさせる
- Session情報はあるが、自動ログイントークン（Cookie）が存在しない場合
  - SessionIDの再発行
  - Session情報の登録
  - 会員ページのアクセスを許可する
- Session情報はないが、自動ログイントークンが存在する場合
  - 自動ログイントークンの値が、CookieとDBで一致するか確認  
    - 一致する場合
      - SessionIDの再発行
      - Session情報の登録
      - 自動ログイントークンの更新
      - 会員ページのアクセスを許可
    - 一致しない場合
      - Cookieの自動ログイントークンを削除
      - ログイン画面へリダイレクトさせる
- Session情報も自動ログイントークンも存在する
  - SessionIDの再発行
  - Session情報の登録
  - 自動ログイントークンの更新
  - 会員ページのアクセスを許可

---
### ログアウト
#### ●概要
- 本システムにおいてログアウトの状態は、Session情報がないこと、自動ログイントークン（Cookie）がないことを指す  

#### ●ログアウト時の処理の流れ
  1. Session情報の破棄
  2. 自動ログイントークンの削除（CookieとDB）
      - DBのトークンに関しては、トークン値の一致するレコードのみ削除（ユーザIDに紐づくトークンを全て消す訳ではない）

---
### ユーザ登録
#### ●概要
- ユーザ登録に必要な情報は以下とする
  - メールアドレス
  - パスワード
  - ユーザ名
- ユーザ登録時にはメールアドレス認証を含む

#### ●ユーザ登録時の処理の流れ
1. ユーザ情報の受け取り
    - この段階ではメールアドレスのみが送られる
2. バリデーションチェック
    - 情報不足または条件を満たしていない場合は、エラー情報を記載した登録フォームを返す
3. 同一のメールアドレスでDBに登録されているユーザがいるか確認
    - 存在する場合は、エラー情報を記載した登録フォームを返す
4. 一時ユーザテーブルにユーザの情報と認証用トークンを登録
    - この時に登録する情報は以下のみ
      - メールアドレス
5. 入力されたメールアドレスに対して、項番4で登録した認証用トークンを含むリンクを送付
    - 有効期限は24時間と記載する
6. （ユーザ）認証用トークンが含まれたリンクにアクセス
7. 一時ユーザテーブルに認証用のトークンと一致するデータがあるか確認
    - 存在しないもしくは有効期限が切れている場合、その旨を記載したページを表示  
    一時ユーザテーブルのデータについては、バッチにて有効期限を確認し、期限が切れている場合は削除される  
8. 認証用トークンが有効の場合、以下の入力欄のあるフォームを返す
    - メールアドレス
    - パスワード
    - 認証用トークン（hiddenで送信される）
9. 情報の受け取り
10. バリデーションチェック
    - 情報不足または条件を満たしていない場合は、エラー情報を記載したフォームを返す
11. 同一のメールアドレスでDBに登録されているユーザがいるか確認
    - 存在する場合は、エラー情報を記載した登録フォームを返す
12. 一時ユーザテーブルに保存されているメールアドレスと認証用トークンであれば、ユーザテーブルに登録。一時ユーザテーブルのレコードは削除

---
### 思い出登録
#### ●概要
- 思い出登録に必要な情報（ユーザが入力する情報）は以下とする
  - タイトル
  - 行った都道府県（複数選択可）
  - 行った場所の名前（複数登録可）
  - 文章

#### ●思い出登録の流れ
1. 情報の受け取り
2. バリデーションチェック
3. 思い出テーブルにデータ保存
4. 登録完了（もしくは失敗）のアラートを出す

---
### 思い出編集
#### ●概要
- 思い出登録で登録した内容が編集できるようにする

---
### 思い出削除
#### ●概要
- 思い出登録したものを削除できるようにする  
実態としては論理削除

---
### 思い出検索
#### ●概要
- 以下の条件から他の人の思い出を検索できる
  - 都道府県
  - 思い出登録にてテキストとして登録されているもの（タイトル、行った場所の名前、文章）

---
### 入力バリデーションについて
- メールアドレス
  - 255文字以内
- ユーザID
  - 30文字以内
- パスワード
  - 8文字以上16文字未満

## DB設計
---
### users table（ユーザテーブル）
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|account_id|id(11)|NO|PRI|NULL|AUTO_INCREMENT|
|user_id|varchar(30)|NO|UNI|NULL|||
|user_name|varchar(50)|NO||NULL|||
|mail_address|varchar(255)|NO|UNI|NULL||単体でユニーク|
|password|varchar(255)|NO||NULL|||
|created_at|datetime|NO||CURRENT_TIMESTAMP|||
|updated_at|datetime|NO||CURRENT_TIMESTAMP|on update CURRENT_TIMESTAMP||
|deleted_at|datetime|YES||NULL|||
### Association
- has_many: memories, places


### temp_users table（一時ユーザテーブル）
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|temp_user_token|varchar(64)|NO|PRI|NULL|||
|mail_address|varchar(255)|NO||NULL|||
|created_at|datetime|NO||CURRENT_TIMESTAMP||
### Association
- is not exists

### memories table（思い出テーブル）
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|memory_id|int(11)|NO|PRI|NULL|AUTO_INCREMENT||
|account_id|varchar(11)|NO|FOR|NULL|||
|title|varchar(50)|NO||NULL|||
|sentence|varchar(6000)|NO||NULL|||
|created_at|datetime|NO||CURRENT_TIMESTAMP|||
|updated_at|datetime|NO||CURRENT_TIMESTAMP|on update CURRENT_TIMESTAMP||
|deleted_at|datetime|YES||NULL|||
### Association
- belongs_to: users
- belongs_to_many: memory_place

### places（場所テーブル）
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|place_id|int(11)|NO|PRI|NULL|AUTO_INCREMENT||
|place_name|varchar(100)|NO|UNI|NULL|||
|created_at|datetime|NO||CURRENT_TIMESTAMP|||
|created_account_id|int(11)|NO|FOR|NULL|||
### Association
- belongs_to: users
- belongs_to_many: memory_place


### prefectures（都道府県テーブル）
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|prefecture_id|int(11)|NO|PRI|NULL|AUTO_INCREMENT||
|prefecture_name|varchar(20)|NO|UNI|NULL|||
### Association
- belongs_to_many: memory_prefecture

### memory_place
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|memory_id|int(11)|NO|PRI<br>FOR|NULL||place_idと合わせてPRIMARY KEY|
|place_id|int(11)|NO|PRI<br>FOR|NULL|||
### Association
- ?（関係名称不明）

### memory_prefecture
|Field|Type|Null|KEY|Default|Extra|Comment|
|-----|----|----|---|-------|-----|-------|
|memory_id|int(11)|NO|PRI<br>FOR|NULL||prefecture_idと合わせてPRIMARY KEY|
|prefecture_id|int(11)|NO|PRI<br>FOR|NULL|||
### Association
- ?（関係名称不明）
