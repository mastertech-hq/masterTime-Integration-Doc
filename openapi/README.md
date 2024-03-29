# Open API

![](https://img.shields.io/badge/version-v0.11.0-brightgreen)
![](https://img.shields.io/badge/last%20update-27%20Oct%202022-blue)

## หัวข้อ

- [ข้อมูลเบื้องต้น](#ข้อมูลเบื้องต้น)
- [Root Endpoint](#root-endpoint)
- [Client](#client)
- [Error](#error)
- [การ Login ของ Client](#การ-login-ของ-client)
- [Refresh Token และ Access Token](#refresh-token-และ-access-token)
    - [Refresh Token](#refresh-token)
    - [Access Token](#access-token)
    - [การเรียกใช้ API ด้วย Access Token](#การเรียกใช้-api-ด้วย-access-token)
    - [การขอ Access Token ใหม่ ด้วย Refresh Token](#การขอ-access-token-ใหม่-ด้วย-refresh-token)
- [API ที่สามารถใช้งานได้](#API-ที่สามารถใช้งานได้)
    - [การเพิ่มพนักงานใหม่](#การเพิ่มพนักงานใหม่)
    - [การให้สิทธิ์พนักงานลงเวลานอกสถานที่](#การให้สิทธิ์พนักงานลงเวลานอกสถานที่)

## ข้อมูลเบื้องต้น

- API ของ masterTime เป็น `RESTful`
- Request Body และ Response Body เป็น `JSON`
- เราใช้ `HTTP Response Status Code` เพื่อบอกว่าการ `Request` นั้น `Success` หรือ `Error` ตามความหมายของโค้ดนั้น ๆ
- API ของ masterTime ยอมรับ `https` เท่านั้น

## Root Endpoint ของ masterTime

```
https://api.mastertime.io/api/v1/
```

## Client

- `Client` คือ Software ที่ต้องการเชื่อมต่อกับระบบ masterTime
- ทาง developer ของ masterTime จะสร้าง Account ให้สำหรับ Software 3rd-Party ที่ต้องการเชื่อมต่อกับ masterTime
- ผู้พัฒนา Software 3rd-Party จะได้รับ `Client ID` และ `Client Secret` เพื่อให้ Client ใช้ในการ Login

## Error

ในระบบ masterTime เมื่อ return `HTTP Response Status Code` เป็น `4xx` หรือ `5xx` จะมี `Response Body` ดังนี้

```json
{
  "status": 500,
  "message": "some messages that the programmer can describe this error",
  "time": "2022-03-01T11:08:34.318Z",
  "log_id": "string",
  "error_code": "string",
  "error_args": {}
}
```

#### คำอธิบาย

| ชื่อ Field | คำอธิบาย                                                                                 |
|------------|------------------------------------------------------------------------------------------|
| status     | `Number` ที่เป็น `Integer` ตรงกับ `HTTP Response Status Code` ที่ return กลับมา          |
| message    | ข้อความที่อธิบายสาเหตุของ error                                                          |
| time       | เวลาที่เกิด error (ใช้มาตรฐาน [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339)) |
| log_id     | Log ID ในระบบ masterTime สำหรับแต่ละ Request                                             |
| error_code | รหัสสำหรับแต่ละ Error ที่เกิดขึ้นในระบบ masterTime                                       |
| error_args | ยังไม่ใช้งาน (ตอนนี้จะส่งกลับมาเป็น `NULL` เสมอ)                                         |

## การ Login ของ Client

ก่อนที่ Client จะสามารถสื่อสารกับ masterTime ได้ จะต้อง login ก่อน

#### Endpoint

`/auth/login/software3rdparty`

#### Request Body

```json
{
  "app_id": "string",
  "secret": "string",
  "login_type_id": 3,
  "company_uuid": "string"
}
```

##### คำอธิบาย

| ชื่อ Field    | คำอธิบาย                                                       |
|---------------|----------------------------------------------------------------|
| app_id        | Client ID ที่ masterTime กำหนดให้                              |
| secret        | Client Secret ที่ masterTime กำหนดให้                          |
| login_type_id | ประเภทของการ Login ซึ่ง Software 3rd-Party ต้องส่งค่า `3` เสมอ |
| company_uuid  |                                                                |

#### Response

##### 200 (Login สำเร็จ)

```json
{
  "refresh_token": "eyJDb21wYW55SUQiOjEsIkVtcGxMeWVlSUQiOjAA=",
  "access_token": "4N2UyMmIzOWFlMDUyOGQ0M2QyMmJkMDRjZTgzMWM=",
  "company": {
    "company_uuid": "string",
    "company_code": "string",
    "title_th": "string",
    "title_en": "string"
  },
  "login_type": {
    "login_type_id": 3,
    "title_th": "แอพพลิเคชั่นซอฟต์แวร์",
    "title_en": "Software Application"
  },
  "developer": null,
  "taff_admin": null,
  "company_admin": null,
  "employee": null,
  "hardware": null,
  "software_3rd_party": {
    "app_id": "{Client ID}",
    "is_active": true
  },
  "mobile_unique_code": "",
  "vendor": "",
  "model": "",
  "web_browser": "",
  "os_version": "",
  "sw_version": ""
}
```

ให้ Client เก็บ `refresh_token` และ `access_token` ไว้ เพื่อเรียกใช้งาน API ของ masterTime ต่อไป

##### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

##### 401 (Unauthorized)

- Login ไม่สำเร็จ, `Client ID` หรือ `Client Secret` อาจจะผิด

##### 403 (Forbidden)

- การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

## Refresh Token และ Access Token

เมื่อ Client ทำการ Login สำเร็จ จะได้รับ `refresh_token` และ `access_token` เพื่อเรียกใช้งาน API ของ masterTime ต่อไป

### Refresh Token

เป็น `token` ที่มีอายุค่อนข้างยาว ใช้สำหรับขอ `access_token` ใหม่ เมื่อ `access_token` หมดอายุ

### Access Token

เป็น `token` ที่มีอายุค่อนข้างสั้น ใช้สำหรับเรียกใช้ API ของ masterTime โดยจะต้องแนบ `access_token`
ใน `HTTP Authorization` header โดยใช้เป็น `bearer token`

## การเรียกใช้ API ด้วย Access Token

Client ต้องแนบ `access_token` ใน `HTTP Authorization` header โดยใช้เป็น `bearer token` ทุกครั้งที่เรียก API ของ
masterTime

ซึ่งในการเรียกใช้งาน API แต่ละครั้ง จะมีการตรวจสอบสิทธิ์การใช้งานจาก `access_token`

ซึ่งมีโอกาสที่ `access_token` จะใช้งานไม่ได้ ดังนี้

#### 2xx (Success)

- การ Request ของ Client สำเร็จ และจะได้รับผลลัพธ์กลับมาทาง Request Body

#### 401 (Access Token is invalid)

- ในกรณีนี้หมายถึง Access Token หมดอายุ, Client จะต้องขอ Access Token ใหม่ ด้วย Refresh Token (
  ดู [การขอ Access Token ใหม่ ด้วย Refresh Token](#การขอ-access-token-ใหม่-ด้วย-refresh-token))

#### 403 (No Permission)

- ในกรณีนี้หมายถึง Access Token นี้ไม่มีสิทธิ์ในเรียกใช้งาน API ดังกล่าว

## การขอ Access Token ใหม่ ด้วย Refresh Token

เมื่อ Access Token หมดอายุ, Client จะต้องขอ Access Token ใหม่ ด้วย Refresh Token โดยมีรายละเอียดดังนี้

#### Endpoint

`/auth/request-access-token`

#### HTTP Method

`POST`

#### Request Body

```json
{
  "refresh_token": "{refresh_token}"
}
```

#### Response

##### 200 (Success)

```json
{
  "refresh_token": "eyJDb21wYW55SUQiOjEsIkVtcGxMeWVlSUQiOjAA=",
  "access_token": "4N2UyMmIzOWFlMDUyOGQ0M2QyMmJkMDRjZTgzMWM="
}
```

- ให้ Client เก็บ `refresh_token` และ `access_token` ไว้ เพื่อเรียกใช้งาน API ของ masterTime ต่อไป

##### 401 (Unauthorized)

- Client จะได้รับ `401` เมื่อขอ `Access Token` ใหม่ `ไม่สำเร็จ`, Client จะต้อง Login อีกครั้ง (
  ดู [การ Login ของ Client](#การ-login-ของ-client))

## API ที่สามารถใช้งานได้

ระบบ masterTime ให้ Software ภายนอกที่ได้รับอนุญาต สามารถเรียกใช้งาน API ได้

โดยการเรียก API ทุกครั้ง Client ต้องแนบ `access_token` ใน `HTTP Authorization` header โดยใช้เป็น `bearer token`
มาด้วยทุกครั้ง (ดู [การเรียกใช้ API ด้วย Access Token](#การเรียกใช้-api-ด้วย-access-token))

API ที่สามารถใช้งานได้ มีดังนี้

## การเพิ่มพนักงานใหม่

### Endpoint

`/function/new-employee`

### HTTP Method

`POST`

### Request Body

```json
{
  "company_uuid": "string",
  "employee_code": "string",
  "employee_firstname_th": "string",
  "employee_lastname_th": "string",
  "employee_firstname_en": "string",
  "employee_lastname_en": "string",
  "organization_code": "string",
  "position_code": "string",
  "employee_type_code": "string",
  "shift_code": "string",
  "role_code": "string"
}
```

#### คำอธิบาย

| ชื่อ Field            | คำอธิบาย                                                     |
|-----------------------|--------------------------------------------------------------|
| company_uuid          | UUID ของบริษัทที่ต้องการเพิ่มพนักงานใหม่                     |
| employee_code         | รหัสพนักงาน                                                  |
| employee_firstname_th | ชื่อจริง (ภาษาไทย)                                           |
| employee_lastname_th  | นามสกุล (ภาษาไทย)                                            |
| employee_firstname_en | ชื่อจริง (ภาษาอังกฤษ)                                        |
| employee_lastname_en  | ชื่อจริง (ภาษาอังกฤษ)                                        |
| organization_code     | รหัสของหน่วยงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)      |
| position_code         | รหัสของตำแหน่ง (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)       |
| employee_type_code    | รหัสของประเภทพนักงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime) |
| shift_code            | รหัสของกะการทำงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)    |
| role_code             | รหัสของสิทธิ์การใช้งาน ในตอนนี้ต้องส่งค่า `NULL`             |

### Response

#### 200 (Successful)

- การเพิ่มพนักงานใหม่สำเร็จ

#### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

#### 401 (Access Token is invalid)

- `Access Token` หมดอายุ, Client จะต้องขอ `Access Token` ใหม่ ด้วย `Refresh Token` ก่อนที่จะกลับมาเรียกใช้ API
  นี้อีกครั้ง (
  ดู [การขอ Access Token ใหม่ ด้วย Refresh Token](#การขอ-access-token-ใหม่-ด้วย-refresh-token))

#### 403 (No Permission)

- ในกรณีนี้หมายถึง Access Token ที่แนบมา ไม่มีสิทธิ์ในเรียกใช้งาน API นี้

#### 409 (Conflict)

- ในระบบ masterTime มีรหัสพนักงานนี้อยู่แล้ว (รหัสพนักงานซ้ำ)

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

## การให้สิทธิ์พนักงานลงเวลานอกสถานที่

### Endpoint

`/function/grant-time-attendance-offsite`

### HTTP Method

`POST`

### Request Body

```json
{
  "company_uuid": "string",
  "employee_code": "string"
}
```

#### คำอธิบาย

| ชื่อ Field    | คำอธิบาย                                                 |
|---------------|----------------------------------------------------------|
| company_uuid  | UUID ของบริษัทที่ต้องการให้สิทธิ์พนักงานลงเวลานอกสถานที่ |
| employee_code | รหัสพนักงาน                                              |

### Response

#### 200 (Successful)

- การให้สิทธิ์พนักงานลงเวลานอกสถานที่สำเร็จ

#### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

#### 401 (Access Token is invalid)

- `Access Token` หมดอายุ, Client จะต้องขอ `Access Token` ใหม่ ด้วย `Refresh Token` ก่อนที่จะกลับมาเรียกใช้ API
  นี้อีกครั้ง (
  ดู [การขอ Access Token ใหม่ ด้วย Refresh Token](#การขอ-access-token-ใหม่-ด้วย-refresh-token))

#### 403 (No Permission)

- ในกรณีนี้หมายถึง Access Token ที่แนบมา ไม่มีสิทธิ์ในเรียกใช้งาน API นี้

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

