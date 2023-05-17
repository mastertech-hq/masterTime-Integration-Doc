# masterTime's Integration APIs

![](https://img.shields.io/badge/version-v0.5.0-brightgreen)
![](https://img.shields.io/badge/last%20update-17%20May%202023-blue)

## หัวข้อ

- [ข้อมูลเบื้องต้น](#ข้อมูลเบื้องต้น)
- [Root Endpoint](#root-endpoint)
- [Client](#client)
- [Error](#error)
- [Basic Authentication](#basic-authentication)
- [HTTP Response Status Code](#http-response-status-code)
- [API ที่สามารถใช้งานได้](#api-ที่สามารถใช้งานได้)
    - [1. การเพิ่มพนักงานใหม่](#1-การเพิ่มพนักงานใหม่)
    - [2. แก้ไขข้อมูลพนักงาน](#2-แก้ไขข้อมูลพนักงาน)
    - [3. กำหนดสิทธิ์พนักงานลงเวลานอกสถานที่](#3-กำหนดสิทธิ์พนักงานลงเวลานอกสถานที่)

## ข้อมูลเบื้องต้น

- API ของ masterTime เป็น `RESTful`
- Request Body และ Response Body เป็น `JSON`
- เราใช้ `HTTP Response Status Code` เพื่อบอกว่าการ `Request` นั้น `Success` หรือ `Error` ตามความหมายของโค้ดนั้น ๆ
- API ของ masterTime ยอมรับ `https` เท่านั้น

## Root Endpoint

```
https://api.mastertime.io/api/v1/integration/
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

## Basic Authentication

- ผู้ที่ต้องการใช้งาน Integration APIs จาก masterTime จะได้รับ `client_id` และ `client_secret` จาก masterTime
- Client จะต้องส่ง Basic Authentication มาใน `HTTP Authorization` header ทุกครั้งที่เรียกใช้งาน APIs ดังนี้

```
Authorization: Basic {BASE64_ENCODED(client_id:client_secret)}
```

## HTTP Response Status Code

HTTP Response Status Code มาตรฐานของ masterTime มีดังนี้

#### 2xx (Success)

- สำเร็จ

#### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

#### 401 (Unauthorized)

- ยืนยันตัวตนไม่สำเร็จ
- อาจจะไม่มี HTTP Authorization header ในการ Request
- `Client ID` หรือ `Client Secret` อาจจะไม่ถูกต้อง

#### 403 (Forbidden)

- การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

## API ที่สามารถใช้งานได้

ระบบ masterTime ให้ Software ภายนอกที่ได้รับอนุญาต สามารถเรียกใช้งาน API ได้

โดยการเรียก API ทุกครั้ง Client จะต้องส่ง Basic Authentication มาใน `HTTP Authorization` header ด้วยเสมอ

API ที่สามารถใช้งานได้ มีดังนี้

## 1. การเพิ่มพนักงานใหม่

### Endpoint

```
/std/func/new-employee
```

### HTTP Method

`POST`

### Request Body

```json
{
    "employee": {
        "company_uuid": "string",
        "employee_code": "string",
        "firstname_th": "string",
        "lastname_th": "string",
        "firstname_en": "string",
        "lastname_en": "string",
        "identification_number": "string",
        "identification_type": "string (thai|foreign|passport)",
        "organization_code": "string",
        "position_code": "string",
        "employee_type_code": "string",
        "shift_code": "string",
        "role_code": null,
        "optional_information": {
            "string_field_1": "string",
            "string_field_2": "2023-03-01T11:08:34.318Z",
            "number_field_1": 0,
            "number_field_2": 2.3,
            "boolean_field_1": true,
            "boolean_field_2": false
        }
    },
    "options": {
        "unique_field": [
            "string"
        ]
    }
}
```

#### คำอธิบาย

- **`employee`:**
    - ข้อมูลพนักงานใหม่
    - ข้อมูลส่วนนี้จะต้องส่งมาเสมอ
- **`options`:**
    - ควบคุมการทำงานเพิ่มเติม
    - ข้อมูลส่วนนี้สามารถเป็น `null` ได้
    - ค่า default คือ `null`

##### ข้อมูลใน `employee` object

| ชื่อ Field            | คำอธิบาย                                                                                                                                                                                      |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| company_uuid          | UUID ของบริษัทที่ต้องการเพิ่มพนักงานใหม่                                                                                                                                                      |
| employee_code         | รหัสพนักงาน (จะต้องไม่ซ้ำกัน)                                                                                                                                                                 |
| firstname_th          | ชื่อจริง (ภาษาไทย)                                                                                                                                                                            |
| lastname_th           | นามสกุล (ภาษาไทย)                                                                                                                                                                             |
| firstname_en          | ชื่อจริง (ภาษาอังกฤษ)                                                                                                                                                                         |
| lastname_en           | นามสกุล (ภาษาอังกฤษ)                                                                                                                                                                          |
| identification_number | เลขประจำตัวที่ราชการออกให้, สามารถเป็นค่า `null` ได้                                                                                                                                          |
| identification_type   | ประเภทของ `identification_number` ("`thai`" = เลขบัตรประชาชน, "`foreign`" = เลขบัตรต่างด้าว, "`passport`" = เลข passport), สามารถเป็นค่า `null` ได้ถ้า `identification_number` เป็นค่า `null` |
| organization_code     | รหัสของหน่วยงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                       |
| position_code         | รหัสของตำแหน่ง (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                        |
| employee_type_code    | รหัสของประเภทพนักงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                  |
| shift_code            | รหัสของกะการทำงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                     |
| role_code             | รหัสของสิทธิ์การใช้งาน ในตอนนี้ต้องส่งค่า `NULL`                                                                                                                                              |
| optional_information  | ข้อมูลเพิ่มเติม (สามารถเป็น `null` ได้)<br/>- สามารถใช้เก็บข้อมูลอะไรก็ได้<br/>- สามารถใช้ชื่อ field อย่างไรก็ได้<br/>- field ต้องเป็นประเภท string, integer, float, boolean                  |

##### ข้อมูลใน `options` object

| ชื่อ Field   | คำอธิบาย                                                                                                             |
|--------------|----------------------------------------------------------------------------------------------------------------------|
| unique_field | ระบุเงื่อนไขการตรวจสอบ field ของข้อมูลพนักงานเพิ่มเติมเพื่อป้องการข้อมูลซ้ำซ้อน (default คือ `null` ไม่มีการตรวจสอบ) |

###### `unique_field` ที่สามารถใช้งานได้

- `identification_number`: เพื่อป้องกันไม่ให้ "**เลขประจำตัวที่ราชการออกให้**" ซ้ำกัน

### Response

#### 200 (Successful)

- การเพิ่มพนักงานใหม่สำเร็จ

#### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

#### 401 (Unauthorized)

- ยืนยันตัวตนไม่สำเร็จ
- อาจจะไม่มี HTTP Authorization header ในการ Request
- `Client ID` หรือ `Client Secret` อาจจะไม่ถูกต้อง

#### 403 (Forbidden)

- การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

#### 409 (Conflict)

- ในระบบ masterTime มีรหัสพนักงานนี้อยู่แล้ว (รหัสพนักงานซ้ำ) หรือมีข้อมูลพนักงานที่มี `unique_field` นี้อยู่แล้ว

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

## 2. แก้ไขข้อมูลพนักงาน

### Endpoint

```
/std/func/update-employee
```

### HTTP Method

`POST`

### Request Body

```json
{
    "employee": {
        "company_uuid": "string",
        "employee_code": "string",
        "firstname_th": "string",
        "lastname_th": "string",
        "firstname_en": "string",
        "lastname_en": "string",
        "identification_number": "string",
        "identification_type": "string (thai|foreign|passport)",
        "organization_code": "string",
        "position_code": "string",
        "employee_type_code": "string",
        "shift_code": "string",
        "role_code": null,
        "optional_information": {
            "string_field_1": "string",
            "string_field_2": "2023-03-01T11:08:34.318Z",
            "number_field_1": 0,
            "number_field_2": 2.3,
            "boolean_field_1": true,
            "boolean_field_2": false
        }
    },
    "options": {
        "unique_field": [
            "string"
        ]
    }
}
```

#### คำอธิบาย

- **`employee`:**
    - ข้อมูลพนักงานที่ต้องการแก้ไข
    - ข้อมูลส่วนนี้จะต้องส่งมาเสมอ

##### ข้อมูลใน `employee` object

| ชื่อ Field            | คำอธิบาย                                                                                                                                                                                      |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| company_uuid          | UUID ของบริษัทที่ต้องการเพิ่มพนักงานใหม่                                                                                                                                                      |
| employee_code         | รหัสพนักงานที่ต้องการแก้ไข                                                                                                                                                                    |
| firstname_th          | ชื่อจริง (ภาษาไทย)                                                                                                                                                                            |
| lastname_th           | นามสกุล (ภาษาไทย)                                                                                                                                                                             |
| firstname_en          | ชื่อจริง (ภาษาอังกฤษ)                                                                                                                                                                         |
| lastname_en           | นามสกุล (ภาษาอังกฤษ)                                                                                                                                                                          |
| identification_number | เลขประจำตัวที่ราชการออกให้, สามารถเป็นค่า `null` ได้                                                                                                                                          |
| identification_type   | ประเภทของ `identification_number` ("`thai`" = เลขบัตรประชาชน, "`foreign`" = เลขบัตรต่างด้าว, "`passport`" = เลข passport), สามารถเป็นค่า `null` ได้ถ้า `identification_number` เป็นค่า `null` |
| organization_code     | รหัสของหน่วยงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                       |
| position_code         | รหัสของตำแหน่ง (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                        |
| employee_type_code    | รหัสของประเภทพนักงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                  |
| shift_code            | รหัสของกะการทำงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                     |
| role_code             | รหัสของสิทธิ์การใช้งาน ในตอนนี้ต้องส่งค่า `NULL`                                                                                                                                              |
| optional_information  | ข้อมูลเพิ่มเติม (สามารถเป็น `null` ได้)<br/>- สามารถใช้เก็บข้อมูลอะไรก็ได้<br/>- สามารถใช้ชื่อ field อย่างไรก็ได้<br/>- field ต้องเป็นประเภท string, integer, float, boolean                  |

##### ข้อมูลใน `options` object

| ชื่อ Field   | คำอธิบาย                                                                                                             |
|--------------|----------------------------------------------------------------------------------------------------------------------|
| unique_field | ระบุเงื่อนไขการตรวจสอบ field ของข้อมูลพนักงานเพิ่มเติมเพื่อป้องการข้อมูลซ้ำซ้อน (default คือ `null` ไม่มีการตรวจสอบ) |

###### `unique_field` ที่สามารถใช้งานได้

- `identification_number`: เพื่อป้องกันไม่ให้ "**เลขประจำตัวที่ราชการออกให้**" ซ้ำกัน

### Response

#### 200 (Successful)

- การแก้ไขข้อมูลพนักงานสำเร็จ

#### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

#### 401 (Unauthorized)

- ยืนยันตัวตนไม่สำเร็จ
- อาจจะไม่มี HTTP Authorization header ในการ Request
- `Client ID` หรือ `Client Secret` อาจจะไม่ถูกต้อง

#### 403 (Forbidden)

- การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

#### 404 (Not Found)

- ไม่พบรหัสพนักงานนี้ในระบบ masterTime

#### 409 (Conflict)

- ในระบบ masterTime มี identification_number นี้ในข้อมูลพนักงานคนอื่นแล้ว

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

## 3. กำหนดสิทธิ์พนักงานลงเวลานอกสถานที่

### Endpoint

```
/std/func/set-employee-offsite-grant
```

### HTTP Method

`POST`

### Request Body

```json
{
    "employee": {
        "company_uuid": "string",
        "employee_code": "string"
    },
    "offsite_grant": {
        "location": [
            {
                "location_code": "string",
                "radius_meter": 100,
                "location_detail": {
                    "title_th": "string",
                    "title_en": "string",
                    "latitude": "string",
                    "longitude": "string",
                    "timezone": "string"
                }
            }
        ],
        "start_time": "2006-01-02T15:00Z",
        "end_time": "2026-01-02T15:00Z"
    }
}
```

#### คำอธิบาย

Request body ประกอบด้วย 2 ส่วน คือ `employee` และ `offsite_grant`

- **`employee`:**
    - ข้อมูลพนักงานที่ต้องการกำหนดสิทธิ์ลงเวลานอกสถานที่
    - ข้อมูลส่วนนี้จะต้องส่งมาเสมอ
- **`offsite_grant`:**
    - ข้อมูลการลงเวลานอกสถานที่
    - ข้อมูลส่วนนี้สามารถเป็น `null` ได้เมื่อไม่ต้องการให้สิทธิ์ในการลงเวลานอกสถานที่
    - ข้อมูลนี้จะแทนที่ข้อมูลเดิมทั้งหมด

##### ข้อมูลใน `employee` object

| ชื่อ Field    | คำอธิบาย                                         |
|---------------|--------------------------------------------------|
| company_uuid  | UUID ของบริษัทที่ต้องการเพิ่มพนักงานใหม่         |
| employee_code | รหัสพนักงานที่ต้องการกำหนดสิทธิ์ลงเวลานอกสถานที่ |

##### ข้อมูลใน `offsite_grant` object

| ชื่อ Field | คำอธิบาย                                         |
|------------|--------------------------------------------------|
| location   | รายการสถานที่ที่ให้สิทธิ์ลงเวลานอกสถานที่        |
| start_time | เวลาเริ่มต้นการให้สิทธิ์ (สามารถเป็น `null` ได้) |
| end_time   | เวลาสิ้นสุดการให้สิทธิ์ (สามารถเป็น `null` ได้)  |

##### ข้อมูลใน `location` object

- เป็น `array`
- มีได้มากกว่า 1 สถานที่
- ถ้าเป็น `empty array` หรือ `[]` หมายถึง การให้สิทธิ์ลงเวลานอกสถานที่โดยไม่ระบุสถานที่

| ชื่อ Field      | คำอธิบาย                                           |
|-----------------|----------------------------------------------------|
| location_code   | รหัสสถานที่ (ต้องระบุข้อมูลเสมอ)                   |
| location_detail | ข้อมูลรายละเอียดของสถานที่ (สามารถเป็น `null` ได้) |

##### ข้อมูลใน `location_detail` object

| ชื่อ Field   | คำอธิบาย                                          |
|--------------|---------------------------------------------------|
| title_th     | ชื่อสถานที่ (ภาษาไทย)                             |
| title_en     | ชื่อสถานที่ (ภาษาอังกฤษ)                          |
| latitude     | ละติจูด                                           |
| longitude    | ลองจิจูด                                          |
| timezone     | timezone ของสถานที่                               |
| radius_meter | รัศมีที่พนักงานสามารถลงเวลาในสถานที่นี้ได้ (เมตร) |

##### ตัวอย่างการใช้งาน `location`

###### (I) การระบุข้อมูลสถานที่ให้พนักงาน โดยการเพิ่ม location ใหม่

- ระบุ `location_code` ที่**ไม่เคยมี**ในระบบ
- ระบุ `location_detail` ให้ครบทุก fields

###### (II) การระบุข้อมูลสถานที่ให้พนักงาน โดยการใช้ location เดิมที่มีอยู่แล้วในระบบ

- ระบุ `location_code` ที่**มีอยู่แล้ว**ในระบบ
- ระบุ `location_detail` เป็น `null`

###### (III) การระบุข้อมูลสถานที่ให้พนักงาน โดยอัพเดตข้อมูล location เดิมที่มีอยู่แล้วในระบบ

**หมายเหตุ:** จะอัพเดตให้กับพนักงานทุกคนที่ใช้ `location` นี้ด้วย

- ระบุ `location_code` ที่**มีอยู่แล้ว**ในระบบ
- ระบุ `location_detail` ให้ครบทุก fields

### Response

#### 200 (Successful)

- การกำหนดสิทธิ์พนักงานลงเวลานอกสถานที่สำเร็จ

#### 400 (Bad Request)

- `Request` ไม่ถูกต้อง

#### 401 (Unauthorized)

- ยืนยันตัวตนไม่สำเร็จ
- อาจจะไม่มี HTTP Authorization header ในการ Request
- `Client ID` หรือ `Client Secret` อาจจะไม่ถูกต้อง

#### 403 (Forbidden)

- การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

#### 500 (Internal Server Error)

- เกิดข้อผิดพลาดในระบบ masterTime

