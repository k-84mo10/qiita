---
title: GASで会計システムを作ってみた!!
tags:
  - GAS
private: false
updated_at: '2023-12-16T06:31:55+09:00'
id: 4132a1b0e47f11d46d7c
organization_url_name: null
slide: false
ignorePublish: false
---
この記事は、[EEIC Advent Calendar 2023](https://qiita.com/advent-calendar/2023/eeic)の16日日、12/16の記事として書かれています。

# 目次
- [はじめに](#-はじめに)
- [会計システムの概要](#-会計システムの概要)
- [予算使用申請フェーズの実装](#-予算使用申請フェーズの実装)
  - [予算使用申請フォームの作成](##-予算使用申請フォームの作成)
  - [予算使用許可フォームの作成](##-予算使用許可フォームの作成)
  - [データベースの作成](##-データベースの作成)
- [経費返金フェーズの実装](#-経費返金フェーズの実装)
- [最後に](#-最後に)
- [余談](#-余談)

# はじめに
こんにちは、EEIC3年のツイ廃、ライルです。
EEICは毎年、5月に本郷キャンパスで開催される「五月祭」において、「近未来体験」という企画を行っています。
（以下は去年のHP）

https://2023.eeic.jp/

私は現在、近未来体験2024の会計を担当しています。
そこで、今回は近未来体験の会計システムをGASで作ってみました。

# 会計システムの概要
近未来体験の会計システムは、以下のような流れで動作します。
![会計システム.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/dd9969c4-7b6d-a011-83e7-17659b13a208.png)
予算使用申請フェーズでは、申請者Aが「これ買いたいよ」という申請を行い、会計担当者はその許可の可否を決定します。
許可した場合はその取引のIDを発行して申請者Aに連絡します。
経費返金フェーズでは、予算使用申請フェーズで発行されたIDを用いて、経費の返金の申請を行います。

# 予算使用申請フェーズの実装
## 予算使用申請フォームの作成
まず、予算使用申請フォームを作成します。
フォームの内容は以下の通りです。
- メールアドレス
- 新規回答 or 編集回答
- 予算使用申請者の名前
- 予算使用申請者のチーム
- 予算使用予定金額
- 購入予定品・サービス名
- 購入予定品・サービスのURL
- 申請理由

実際に作成したフォームは以下の通りです。
![予算申請フォーム.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/346bd302-895b-852e-ad69-6bf9cfd9de3c.png)

この時、フォームが入力されたら、会計担当者にメールが送信されるようGASで設定します。
フォームの設定画面から、スクリプトエディタを開きます。
GASのコードは以下の通りです。

```js:sendMail.gs
EMAIL_ADDRESS_TO_ACCOUNTANT = "会計担当者のメールアドレス";

function onFormSubmitted(event) {
  try {
    myFunction(event);
  } catch (_error) {
    try {
      myFunction(event);
    } catch (error) {
      errorNotice();
    }
  }
}

class answerList {
  constructor (emailAddress, isNewAnser, name, team, amount, item, url, reason) {
    this.emailAddress = emailAddress;
    this.isNewAnser = isNewAnser;
    this.name = name;
    this.team = team;
    this.amount = amount;
    this.item = item;
    this.url = url;
    this.reason = reason;
  }
}

function myFunction(event) {
  
  // fetch answer
  const answer = fetchAnswer(event);

  // send email
  sendEmail(answer);
}

function fetchAnswer(event) {
  const emailaddress = event.response.getRespondentEmail();
  const itemResponses = event.response.getItemResponses();
  const isNewAnser = (itemResponses[0].getResponse() == "新規回答") ? 1 : 0;
  const answer = new answerList (
    emailaddress,
    isNewAnser, 
    itemResponses[1].getResponse(), 
    itemResponses[2].getResponse(), 
    itemResponses[3].getResponse(), 
    itemResponses[4].getResponse(), 
    itemResponses[5].getResponse(),
    itemResponses[6].getResponse()
  );
  return answer;
}

function sendEmail(answer) {
  
  if (answer.isNewAnser == 1) {
    // メールの件名
    const subject = "【近未来体験2024】予算使用申請新規回答の連絡";
    // メール本文
    const body = `会計担当へ

予算使用申請がありました。
内容を確認し、申請の受理の可否を考えてください。

下記の内容が申請内容です。
-----
氏名: ${answer.name}
メールアドレス: ${answer.emailAddress}
チーム: ${answer.team}
申請金額: ${answer.amount}
申請品: ${answer.item}
申請品のURL: ${answer.url}
申請理由: ${answer.reason}
-----

ご確認お願い致します。

これは近未来体験2024会計システムからの自動送信です。
本メールに対するお問い合わせは下記メールアドレスまでお願いいたします。
MailTo: ${EMAIL_ADDRESS_OF_ACCOUNTANT}`

  // 会計担当にメール送信
  GmailApp.sendEmail(EMAIL_ADDRESS_TO_ACCOUNTANT, subject, body);
  } else {
    // メールの件名
    const subject = "【近未来体験2024】予算使用申請編集の連絡";
    // メール本文
    const body = `会計担当へ

予算使用申請の編集がありました。
内容を確認し、申請の受理の可否を考えてください。

下記の内容が再申請内容です。
-----
氏名: ${answer.name}
メールアドレス: ${answer.emailAddress}
チーム: ${answer.team}
申請金額: ${answer.amount}
申請品: ${answer.item}
申請品のURL: ${answer.url}
申請理由: ${answer.reason}
-----

ご確認お願い致します。

これは近未来体験2024会計システムからの自動送信です。
本メールに対するお問い合わせは下記メールアドレスまでお願いいたします。
MailTo: ${EMAIL_ADDRESS_OF_ACCOUNTANT}`

  // 会計担当にメール送信
  GmailApp.sendEmail(EMAIL_ADDRESS_TO_ACCOUNTANT, subject, body);
  }
}

function errorNotice() {
    const subject = "【近未来体験2024】予算使用申請フォームにおけるエラー連絡";
    // メール本文
    const body = `会計担当へ

予算申請フォームにおいて、回答がなされましたが、エラーが発生しました。
原因究明をお願いいたします。
MailTo: ${EMAIL_ADDRESS_OF_ACCOUNTANT}`

  // 会計担当にメール送信
  GmailApp.sendEmail(EMAIL_ADDRESS_TO_ACCOUNTANT, subject, body);
}
```
次に、フォームの設定画面から、トリガーを設定します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/0f7580b7-8902-4f43-f089-9f5e271b9c1d.png)
このままではエラーがでるので、以下の手順でエラーを解消します。
設定を開いて、「「appsscript.json」マニフェスト ファイルをエディタで表示する」のチェックを入れます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/8bdb7d91-4abb-1206-7015-da1d48fdfe4c.png)
次に、appsscript.jsonを以下のように書き換えます。
```json:appsscript.json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {
  },
  "oauthScopes" : [
    "https://www.googleapis.com/auth/forms.currentonly",
    "https://www.googleapis.com/auth/forms",
    "https://www.googleapis.com/auth/gmail.send",
    "https://www.googleapis.com/auth/gmail.compose",
    "https://www.googleapis.com/auth/gmail.modify",
    "https://mail.google.com/",
    "https://www.googleapis.com/auth/gmail.addons.current.action.compose"
  ],
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```
これで、予算使用申請フォームの作成は完了です。
問題無く動作するか、実際にフォームを送信して確認してみるといいでしょう。

## 予算使用許可フォームの作成
申請内容が妥当だと判断したら、会計担当者は予算使用許可フォームを入力してデータベースに登録します。
そこで、予算使用許可フォームを作成します。
フォームの内容は以下の通りです。
- 申請者の氏名
- 申請者のメールアドレス
- 申請者のチーム
- 申請金額
- 申請品・サービス

申請理由や申請品のURLはあくまで申請の認可を決める際に参考にする情報であるため、わざわざデータベースに登録する必要はないでしょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/08b3ce6b-6824-22fa-8ba9-c1e36d03f0c3.png)
次に、フォームが送信されたら、以下の動作をするようにGASを設定します。
- フォームの内容をデータベースに登録する
- データベースで予算オーバーになっていないか確認する
- 予算オーバーになっていない場合は、取引IDを発行し、申請者にメールで連絡する
- 予算オーバーになっている場合は、申請者にメールで連絡する

GASのコードは以下の通りです。
```js:sendDatabase.gs
EMAIL_ADDRESS_TO_ACCOUNTANT = "会計担当者のメールアドレス";
DATABASE_ID = "スプレッドシートのID";

function onFormSubmitted(event) {
  try {
    myFunction(event);
  } catch (_error) {
    try {
      myFunction(event);
    } catch (error) {
      errorNotice();
    }
  }
}

function myFunction(event) {

  // fetch answer
  const answer = fetchAnswer(event);

  // send database and determin id
  const id = sendDatabase(answer);

  // send email 
  sendEmail(answer, id);
}

class answerList {
  constructor (name, emailAddress, team, amount, item) {
    this.name = name;
    this.emailAddress = emailAddress;
    this.team = team;
    this.amount = amount;
    this.item = item;
  }
}

function fetchAnswer(event) {
  const itemResponses = event.response.getItemResponses();
  const answer = new answerList (
    itemResponses[0].getResponse(),
    itemResponses[1].getResponse(),
    itemResponses[2].getResponse(),
    Number(itemResponses[3].getResponse()),
    itemResponses[4].getResponse(),
  )
  return answer;
}

function sendDatabase(answer) {
  const sht = SpreadsheetApp.openById(DATABASE_ID).getSheetByName(answer.team);
  const lastRow = sht.getLastRow();
  const lastSum = (lastRow == 1) ? 0 : Number(sht.getRange(lastRow, 6).getDisplayValue()) 
  const nowSum = answer.amount+lastSum;
  let id = setTeamId(answer)+lastRow;

  if (checkBudgetOver(nowSum, answer)) id = 'Y';

  sht.getRange(lastRow+1, 1).setValue(id);
  sht.getRange(lastRow+1, 2).setValue(answer.name);
  sht.getRange(lastRow+1, 3).setValue(answer.emailAddress);
  sht.getRange(lastRow+1, 4).setValue(answer.item);
  sht.getRange(lastRow+1, 5).setValue(answer.amount);
  sht.getRange(lastRow+1, 6).setValue(nowSum);

  return id;
}

function setTeamId(answer) {
  let teamId = 'Z'; 
  switch(answer.team) {
    case "teamA":
      teamId = 'A';
      break;
    case "teamB":
      teamId = 'B';
      break;
    case "teamC":
      temaId = 'C';
      break;
    case "teamD":
      temaId = 'D';
      break;
    case "teamE":
      teamId = 'E';
      break;
    case "teamF":
      teamId = 'F';
      break;
    case "teamG":
      teamId = 'G';
      break;
    case "teamH":
      teamId = 'H';
      break;
    default:
      console.log("team may not be existed.")
      break;
  }
  return teamId;
}

function checkBudgetOver(nowSum, answer) {
  let BudgetOver = true;
  switch(answer.team) {
    case "teamA":
      if (nowSum <= 10000) BudgetOver = false;
      break;
    case "teamB":
      if (nowSum <= 20000) BudgetOver = false;
      break;
    case "teamC":
      if (nowSum <= 30000) BudgetOver = false;
      break;
    case "teamD":
      if (nowSum <= 40000) BudgetOver = false;
      break;
    case "teamE":
      if (nowSum <= 50000) BudgetOver = false;
      break;
    case "teamF":
      if (nowSum <= 60000) BudgetOver = false;
      break;
    case "teamG":
      if (lastSum <= 70000) BudgetOver = false;
      break;
    case "teamH":
      if (lastSum <= 80000) BudgetOver = false;
      break;
    default:
      console.log("team may not be existed.")
      break;
  }
  return BudgetOver;
}

function sendEmail(answer, id) {
  if (id == 'Y') {
    // メールの件名
    const subject = "【近未来体験2024】予算使用申請の却下";
    // メール本文
    const body = `${answer.name}様
    
以下の内容の予算申請が却下されました。
理由は予算オーバーです。

下記の内容が却下された申請内容です。
-----
氏名: ${answer.name}
チーム: ${answer.team}
申請金額: ${answer.amount}
申請品: ${answer.item}
-----

ご確認お願い致します。

MailTo: ${EMAIL_ADDRESS_TO_ACCOUNTANT}`

  GmailApp.sendEmail(answer.emailAddress, subject, body);  

  } else {
    // メールの件名
    const subject = "【近未来体験2024】予算使用申請の承認";
    // メール本文
    const body = `${answer.name}様

以下の内容の予算申請が受理されました。
商品の購入、サービスの使用をして構いません。
領収書が無いと返金できないため、領収書は失くさずに保管してください。
また、取引終了後は経費申請フォームの回答を忘れずに行ってください。

なお、今回の予算使用IDは「${id}」です。
経費申請フォームで使用するため、忘れないようにしてください。


下記の内容が受理された申請内容です。
-----
氏名: ${answer.name}
チーム: ${answer.team}
申請金額: ${answer.amount}
申請品: ${answer.item}
-----

ご確認お願い致します。

これは近未来体験2024会計システムからの自動送信です。
本メールに対するお問い合わせは下記メールアドレスまでお願いいたします。
MailTo: ${EMAIL_ADDRESS_TO_ACCOUNTANT}`

  GmailApp.sendEmail(answer.emailAddress, subject, body);   
  }
}

function errorNotice() {
    const subject = "【近未来体験2024】予算使用許可フォームにおけるエラー連絡";
    // メール本文
    const body = `会計担当へ

予算使用許可フォームにおいて、回答がなされましたが、エラーが発生しました。
原因究明をお願いいたします。
MailTo: ${EMAIL_ADDRESS_TO_ACCOUNTANT}`
  // 会計担当にメール送信
  GmailApp.sendEmail(EMAIL_ADDRESS_TO_ACCOUNTANT, subject, body);
}
```
スプレッドシートのIDは、後述のデータベースの作成で作成したスプレッドシートのIDを入力してください。
checkBudgetOver関数は、予算オーバーになっていないか確認する関数です。
各チームの予算をここに記入していきましょう。
IDの発行は「チーム名+発行番号」で行います。（例えば、teamAの場合はA1,A2,A3...となります。）
トリガーの設定は予算使用申請フォームと同様に行います。
これで、予算使用許可フォームの作成は完了です。

## データベースの作成
次にデータベースという名のスプレッドシートを作成します。
スプレッドシートはチームごとにシートを分けて以下のように作ってあげるといいでしょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/47346801-a1a3-5687-dd24-28611ce1c4db.png)
これで、予算使用申請フェーズの実装は完了です。
予算使用許可フォームを送信して、データベースにちゃんと登録されているか、メールが申請者に送信されているか確認してみましょう。

# 経費返金フェーズの実装
お買い物が終わったら、申請者は経費返金フォームを送信します。
フォームの内容は以下の通りです。
- メールアドレス
- 氏名
- チーム
- 予算使用ID
- 使用金額
- 領収書の画像

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3632184/bba74ae7-9484-cb34-3d6f-d0ce195a0326.png)
次に、フォームが送信されたら、以下の動作をするようにGASを設定します。
- 実際に掛かった金額をデータベースに登録する
- 現在までに使用した金額を計算する
- 会計担当にメールで連絡する

本来はここで、使用した金額が予算を超えていないか確認する必要がありますが、今回は省略します。
（予算超えても何とかなるように予算を設定したので。）
GASのコードは以下の通りです。
```js:sendDatabase.gs
EMAIL_ADDRESS_TO_ACCOUNTANT = "会計担当者のメールアドレス";
DATABASE_ID = "スプレッドシートのID";

function onFormSubmitted(event) {
  try {
    myFunction(event);
  } catch (_error) {
    try {
      myFunction(event);
    } catch (error) {
      errorNotice();
    }
  }
}

function myFunction(event) {
  // fetch answer
  const answer = fetchAnswer(event);

  // send Database
  sendDatabase(answer);

  // send Email
  sendEmail();
}

class answerList {
  constructor (name, team, id, amount) {
    this.name = name;
    this.team = team;
    this.id = id;
    this.amount = amount;
  }
}

function fetchAnswer(event) {
  const itemResponses = event.response.getItemResponses();
  const amount = (Number(itemResponses[3].getResponse()) > 0) ? Number(itemResponses[3].getResponse()) : 0 ;
  const answer = new answerList (
    itemResponses[0].getResponse(),
    itemResponses[1].getResponse(),
    itemResponses[2].getResponse(),
    amount,
  )
  return answer;
}

function sendDatabase(answer) {
  const sht = SpreadsheetApp.openById(DATABASE_ID).getSheetByName(answer.team);
  let idNumber = parseInt((answer.id).replace(/[^0-9]/g, ""));
  const lastRow = sht.getLastRow();

  sht.getRange(idNumber+1, 7).setValue(answer.amount);

  let sum = 0;
  for (let i=2; i <= lastRow; i++) {
    sum = sum + Number(sht.getRange(i, 7).getDisplayValue()) 
    sht.getRange(i, 8).setValue(sum);
  }
}

function errorNotice() {
    const subject = "【近未来体験2024】経費申請フォームにおけるエラー連絡";
    // メール本文
    const body = `会計担当へ

経費申請フォームにおいて、回答がなされましたが、エラーが発生しました。
原因究明をお願いいたします。
MailTo: ${EMAIL_ADDRESS_TO_ACCOUNTANT}`
  // 会計担当にメール送信
  GmailApp.sendEmail(EMAIL_ADDRESS_TO_ACCOUNTANT, subject, body);
}

function sendEmail() {
    const subject = "【近未来体験2024】経費申請フォームの提出";
    // メール本文
    const body = `会計担当へ

経費申請フォームにおいて、回答がなされました。
対応をお願いいたします。
MailTo: ${EMAIL_ADDRESS_TO_ACCOUNTANT}`
  // 会計担当にメール送信
  GmailApp.sendEmail(EMAIL_ADDRESS_TO_ACCOUNTANT, subject, body);
}
```
トリガーの設定は予算使用申請フォームと同様に行います。
これで、経費返金フォームの作成は完了です。
実際にフォームを送信して、データベースにちゃんと登録されているか確認してみましょう。

# 最後に
いかがでしたか？
今回は、GASを用いて会計システムを作成してみました。
GASは、JavaScriptを用いて簡単にWebアプリケーションを作成できるので、ぜひ使ってみてください。
また、今年も近未来体験は五月祭で開催されます。
実際に電子工作を体験したり、プログラミング教室に参加したり、私達EEIC生の作ったVRゲームを遊んだり、EEICの研究内容に触れてみたり……、様々な体験ができます。
ぜひ、遊びに来てくださいね♪

# 余談
今回初めてJavaScriptを触ったのですが、1000+0=10000となり驚きましたね。
動的型付けは私には向いていないようです。
