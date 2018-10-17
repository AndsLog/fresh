---
layout: post
title: "chatbot-backstage"
date: 2018-09-28 00:00:00
description: 聊天機器人後台系統。
---

# service bot 後台管理系統

## 功能說明

| 功能 | 說明 |
| :---:  |:---: |
| watson assistant後台管理 | 連結帳號之 Watson Assistant 後台頁面，能對應工作區(Worksapce)連到相關頁面，包含意圖(Intent)、實體(Entity)、對話(Dialog)及數據總覽 |
| AI訓練 | 顯示對話的歷史記錄，修正意圖(Intent) |
| AI訓練明細 | 顯示被訓練過的語句及意圖(Intent) |
| 花費 | 以月為單位，顯示每天預估之花費 |
| 表格上傳 | 將 QA 以特定 Excel 表格上傳至 Watson Assistant 建立 意圖(Intent)、實體(Entity)及對話(Dialog)內容 |

---

## 地區對照表：

| 地區| 縮寫 |
| :---: | :---: |
| 美國南部(us-south) | ng |
| 雪梨(Sydney) | au-syd |
| 德國(Germany) | eu-de |
| 英國(United Kingdom) | eu-gb |

## 流程圖

### ![登入流程圖](../assets/img/backstage-login.png)

#### 使用之API：

- 取得該地區API的資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/info
```
```
RESPONSE:
{
    "name": "Bluemix",
    "build": "270030",
    "support": "http://ibm.biz/bluemix-supportinfo",
    "version": 0,
    "description": "IBM Bluemix",
    "authorization_endpoint": "https://login.ng.bluemix.net/UAALoginServerWAR",
    "token_endpoint": "https://uaa.ng.bluemix.net",
    "allow_debug": true
}
```
- 送出IBM帳號密碼至該API，取得access token
```http
POST  https://login.{地區縮寫}.bluemix.net/UAALoginServerWAR/oauth/token
```

Body：

|名稱|描述|
| :---: | :---: |
|username (必需)|帳號|
|password (必需)|密碼|
|grant_type (必需)|值固定為password|
```
body需用querystring格式傳遞:
username={帳號}&password={密碼}&grant_type=password
```
```
RESPONSE:
{
    "access_token": "<YOUR_ACCESS_TOKEN>",
    "token_type": "bearer",
    "refresh_token": "<YOUR_REFRESH_TOKEN>",
    "expires_in": 1209600,
    "scope": "openid network.write uaa.user cloud_controller.read password.write cloud_controller.write network.admin",
    "jti": "<YOUR_JTI>"
}
```

### ![Watson Assistant後台管理流程圖](../assets/img/backstage-waManage.png)

#### 使用之API：
- 取得帳號下所有組織資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/organizations
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/organizations/list_all_organizations.html)

- 取得組織下所有空間資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/organizations/{該組織guid}/spaces
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/organizations/list_all_spaces_for_the_organization.html)

- 取得空間下所有服務資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/spaces/{該空間guid}/service_instances
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/list_all_service_instances.html)

- 取得該服務的認證資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/service_instances/{該服務Guid}/service_keys
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/list_all_service_keys_for_the_service_instance.html)

- 取得該服務資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/service_instances?q=name:{該服務名稱}
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/list_all_service_instances.html)

#### 資料庫需使用之view：
用於取得與service bot連接的watson assistant資訊

View name：getWatson
```js
function (doc) {
  if(!doc.watson_assistant_out) {
    if(doc.watsonName && doc.workspaceId){
      emit({watsonName:doc.watsonName, workspaceId: doc.workspaceId}, 1);
    }
  } else {
    if(doc.watson_assistant_out.watsonName && doc.watson_assistant_out.workspaceId){
      emit({watsonName:doc.watson_assistant_out.watsonName, workspaceId: doc.watson_assistant_out.workspaceId}, 1);
    }
  }
  
}
```

### ![AI訓練流程圖](../assets/img/backstage-AItraining.png)

#### 使用之API：
- 取得該服務的認證資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/service_instances/{該服務Guid}/service_keys
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/list_all_service_keys_for_the_service_instance.html)

- 取得該服務資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/ v2/service_instances?q=name:{該服務名稱}
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/list_all_service_instances.html)

### ![花費流程圖](../assets/img/backstage-usage.png)

#### 使用之API：
- 取得該服務的認證資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/service_instances/{該服務Guid}/service_keys
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/list_all_service_keys_for_the_service_instance.html)

- 取得該組織下特定時間之花費 
```http
GET   https://rated-usage. {地區縮寫}.bluemix.net/v2/metering/organizations/{地區名稱}:{該組織guid}/usage/{欲查詢之時間(年-月-日/年-月)}
```

- 取得該服務方案資訊
```http
GET   https://api.{地區縮寫}.bluemix.net/v2/service_plans?q=unique_id:{服務方案guid}
```
[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_plans/list_all_service_plans.html)

- 創建服務
``` http
POST  https://api.{地區縮寫}.bluemix.net/v2/service_instances?accepts_incomplete=true
```
Body：

|名稱|描述|
| :---: | :---: |
|name (必需)|欲創建之服務名稱|
|service_plan_guid (必需)|該服務方案guid|
|space_guid (必需)|該空間guid|

[詳細資訊](https://apidocs.cloudfoundry.org/4.0.0/service_instances/creating_a_service_instance.html)

### ![表格匯入流程圖](../assets/img/backstage-fileUpload.png)

[範例表格](https://drive.google.com/file/d/12hsWzk45zqLX3y9ZEQXEcuDxvzGJa7bS/view?usp=sharing)

素材整理工作簿：

|表格欄位|對應之watson assistant屬性|
| :---: | :---: |
|分類|意圖(intent)及對話(dialog)的描述(description)|
|標準問題|意圖(intent)|
|延伸問題|意圖中例句(example)|
|標準答案|對話(dialog)|

專有名詞工作簿：

|表格欄位|對應之watson assistant屬性|
| :---: | :---: |
|分類|實體(entity)|
|專有名詞|值(value)|
|別稱|同義詞(synonym)|

---

## 檔案結構
```
├─controllers：為前端頁面及資料之間的處理層
│  │
│  ├─AI_training.js
│  │
│  ├─certificate.js
│  │
│  ├─fileUpload.js
│  │
│  └─usage.js
│
├─models：處理資料庫新增、修改、刪除及查找
│  │
│  └─cloudantdb.js
│
├─public
│  │
│  ├─js
│  │  │
│  │  ├─home.js
│  │  │
│  │  └─login.js
│  │
│  └─stylesheets
│     │
│     └─home.css
│
├─routes
│  │
│  ├─api.js
│  │
│  └─index.js
│
├─services：將ibm cloud foundary及watson assistant整理成promise物件
│  │
│  ├─ibmCloudFoundaryAPI.js
│  │
│  └─ibmWatsonAssistantAPI.js
│
└─views
   │
   ├─error.ejs
   │
   ├─index.ejs
   │
   └─login.ejs
```

---

## 使用套件說明

| 套件 | 用途 |
| :---: | :---: |
| admin-lte | 基於Bootstrap的後台管理系統的通用模板UI，包含jQuery 3、 Bootstrap 3、Datatables等套件|
| xlsx | 處理Excel表格 |
| eonasdan-bootstrap-datetimepicker | 時間選取樣式之套件 |
| bootstrap-select | 使select標籤有搜尋功能之套件 |
| querystring | 將body轉成query string形式 |
| request | request-promise依賴套件 |
| request-promise | 用於對IBM API提出請求 |
| @cloudant/cloudant | IBM ClounantDB 之SDK |
| watson-developer-cloud | IBM Waton Assistant 之SDK |

---

## 網址

https://service-bot-backstage-nodejs.mybluemix.net/login

## demo service bot QR code

![QR code](../assets/img/backstage-chatbot-QRcode.png)



