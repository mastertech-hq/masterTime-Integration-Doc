# masterTime API for Software 3rd-Party Documentation (v0.9.0)

## หัวข้อ

- [ข้อมูลเบื้องต้น](#ข้อมูลเบื้องต้น)
- [Root Endpoint](#root-endpoint)
- [Client](#client)
- [Error](#error)
- [การ Login ของ Client](#การ-login-ของ-client)
- [Refresh Token และ Access Token](#refresh-token-และ-access-token)
    - [Refresh Token](#refresh-token)
    - [Access Token](#access-token)
- [API ที่สามารถใช้งานได้](#API-ที่สามารถใช้งานได้)
    - [การเพิ่มพนักงานใหม่](#การเพิ่มพนักงานใหม่)

## ข้อมูลเบื้องต้น

- API ของ masterTime เป็น `RESTful`
- Request Body และ Response Body เป็น `JSON`
- เราใช้ HTTP Response Code เพื่อบอกว่าการ request นั้น success หรือ error ตามความหมายของโค้ดนั้น ๆ
- API ของ masterTime ยอมรับ `https` เท่านั้น

## Root Endpoint

- สำหรับ `development` : `https://api-demo.mastertime.io/api/v1/`
- สำหรับ `production` : `https://api.mastertime.io/api/v1/`

## Client

- Client คือ Software ที่ต้องการเชื่อมต่อกับระบบ masterTime
- ทาง developer ของ masterTime จะสร้าง Account ให้สำหรับ Software 3rd-Party ที่ต้องการเชื่อมต่อกับ masterTime
- ผู้พัฒนา Software 3rd-Party จะได้รับ Client ID และ Client Secret เพื่อให้ Client ใช้ในการ Login

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
  "refresh_token": "eyJDb21wYW55SUQiOjEsIkVtcGxveWVlSUQiOjAA=",
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

ให้ Client เก็บ `refresh_token` และ `access_token` ไว้ เพื่อใช้เรียกใช้งาน API ของ masterTime ต่อไป

##### 400 (Bad Request)

`Request` ไม่ถูกต้อง

##### 401 (Unauthorized)

Login ไม่สำเร็จ, `Client ID` หรือ `Client Secret` อาจจะผิด

##### 403 (Forbidden)

การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

#### 500 (Internal Server Error)

เกิดข้อผิดพลาดในระบบ masterTime

## Refresh Token และ Access Token

เมื่อ Client ทำการ Login สำเร็จ

### Refresh Token

### Access Token

## API ที่สามารถใช้งานได้

## การเพิ่มพนักงานใหม่

### Endpoint

`/function/new-employee`

### HTTP Method

`POST`

### Request Body

```json
{
  "company_uuid": "3fa85f6457174562b3fc2c963f66afa6",
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

```json
{
  "employee_code": "string",
  "employee_firstname_th": "string",
  "employee_lastname_th": "string",
  "employee_firstname_en": "string",
  "employee_lastname_en": "string",
  "company": {
    "company_uuid": "3fa85f6457174562b3fc2c963f66afa6",
    "company_code": "string",
    "title_th": "string",
    "title_en": "string"
  },
  "employee_type": {
    "employee_type_code": "string",
    "title_th": "string",
    "title_en": "string"
  },
  "organization": {
    "organization_code": "string",
    "title_th": "string",
    "title_en": "string"
  },
  "position": {
    "position_code": "string",
    "title_th": "string",
    "title_en": "string"
  },
  "shift": {
    "shift_code": "string",
    "title_th": "string",
    "title_en": "string",
    "time_in": "hh:mm",
    "time_out": "hh:mm"
  },
  "role": null
}
```

#### 400 (Bad Request)

`Request` ไม่ถูกต้อง

#### 500 (Internal Server Error)

เกิดข้อผิดพลาดในระบบ masterTime