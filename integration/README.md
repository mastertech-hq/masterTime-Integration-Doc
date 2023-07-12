# masterTime's Integration APIs

![version: v0.6.0](https://img.shields.io/badge/version-v0.6.0-brightgreen)
![last update: 12 July 2023](https://img.shields.io/badge/last%20update-12%20July%202023-blue)

## หัวข้อ

- [ข้อมูลเบื้องต้น](#ข้อมูลเบื้องต้น)
- [Root Endpoint](#root-endpoint)
- [Client](#client)
- [Basic Authentication](#basic-authentication)
- [HTTP Response Status Code](#http-response-status-code)
- [Error](#error)
- [Postman](#postman)
- [API ที่สามารถใช้งานได้](#API-ที่สามารถใช้งานได้)
- [Request Body สำหรับการ List ข้อมูล](#request-body-สำหรับการ-list-ข้อมูล)
- [Request Body สำหรับการลบข้อมูล](#request-body-สำหรับการลบข้อมูล)
- [Request Body สำหรับการเพิ่มข้อมูล](#request-body-สำหรับการเพิ่มข้อมูล)
- [Request Body สำหรับการแก้ไขข้อมูล](#request-body-สำหรับการแก้ไขข้อมูล)
- [Object ของข้อมูลในระบบ masterTime](#object-ของข้อมูลในระบบ-mastertime)
    - [1. Object ของข้อมูล "พนักงาน" (Employee)](#1-object-ของข้อมูล-พนักงาน-employee)
    - [2. Object ของข้อมูล "สถานที่สำหรับลงเวลานอกสถานที่" (Off-Site Location)](#2-object-ของข้อมูล-สถานทีสำหรับลงเวลานอกสถานที-off-site-location)
    - [3. Object ของข้อมูล "หน่วยงาน" (Organization)](#3-object-ของข้อมูล-หน่วยงาน-organization)
    - [4. Object ของข้อมูล "ตำแหน่งงาน" (Position)](#4-object-ของข้อมูล-ตำแหน่งงาน-position)
    - [5. Object ของข้อมูล "กะเวลาการทำงาน" (Shift)](#5-object-ของข้อมูล-กะเวลาการทำงาน-shift)
    - [6. Object ของข้อมูล "ประเภทพนักงาน" (Employee Type)](#6-object-ของข้อมูล-ประเภทพนักงาน-employee-type)

## ข้อมูลเบื้องต้น

- API ของ masterTime เป็น `HTTP Request`
- ทุก `Request` จะใช้ `POST` method
- ทุก `Request` จะใช้ `Basic Authentication` ในการยืนยันตัวตน
- `Request Body` และ `Response Body` เป็น `JSON`
- เราใช้ `HTTP Response Status Code` เพื่อบอกว่าการ `Request` นั้น `Success` หรือ `Error` ตามความหมายของโค้ดนั้น ๆ
- API ของ masterTime ยอมรับ `https` เท่านั้น

## Root Endpoint

```
https://api.mastertime.io/api/v1/integration/
```

## Client

- `Client` คือ Software ที่ต้องการเชื่อมต่อกับระบบ masterTime
- ทาง developer ของ masterTime จะสร้าง Account ให้สำหรับ Software 3rd-Party ที่ต้องการเชื่อมต่อกับ masterTime
- ผู้พัฒนา Software 3rd-Party จะได้รับ `Username` และ `Password` เพื่อให้ Client ใช้ในการ Login

## Basic Authentication

- ผู้ที่ต้องการใช้งาน Integration APIs จาก masterTime จะได้รับ `Username` และ `Password` จาก masterTime
- Client จะต้องส่ง Basic Authentication มาใน `HTTP Authorization` header ทุกครั้งที่เรียกใช้งาน APIs ดังนี้

```
Authorization: Basic {BASE64_ENCODED(username:password)}
```

## HTTP Response Status Code

HTTP Response Status Code มาตรฐานของ masterTime มีดังนี้

**2xx (Success)**

- สำเร็จ

**400 (Bad Request)**

- `Request` ไม่ถูกต้อง

**401 (Unauthorized)**

- ยืนยันตัวตนไม่สำเร็จ
- อาจจะไม่มี HTTP Authorization header ในการ Request
- `Client ID` หรือ `Client Secret` อาจจะไม่ถูกต้อง

**403 (Forbidden)**

- การยืนยันตัวตนสำเร็จ แต่ระบบ masterTime ไม่ให้ `Client ID` นี้ใช้งาน

**404 (Not Found)**

- ไม่พบข้อมูลที่ต้องการเรียกใช้งานหรือแก้ไข

**409 (Conflict)**

- มีข้อมูลซ้ำกันในระบบ masterTime

**500 (Internal Server Error)**

- เกิดข้อผิดพลาดในระบบ masterTime

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

**คำอธิบาย**

| ชื่อ Field | คำอธิบาย                                                                                 |
|------------|------------------------------------------------------------------------------------------|
| status     | `Number` ที่เป็น `Integer` ตรงกับ `HTTP Response Status Code` ที่ return กลับมา          |
| message    | ข้อความที่อธิบายสาเหตุของ error                                                          |
| time       | เวลาที่เกิด error (ใช้มาตรฐาน [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339)) |
| log_id     | Log ID ในระบบ masterTime สำหรับแต่ละ Request                                             |
| error_code | รหัสสำหรับแต่ละ Error ที่เกิดขึ้นในระบบ masterTime                                       |
| error_args | ยังไม่ใช้งาน (ตอนนี้จะส่งกลับมาเป็น `NULL` เสมอ)                                         |

## Postman

- [ดาวน์โหลด Postman](https://www.postman.com/downloads/)
- masterTime มีไฟล์ `Postman Collection` และ `Postman Environment` สำหรับทดสอบการเชื่อมต่อกับระบบ masterTime ให้
  Developer ทดสอบ
- เอกสารในส่วนที่เหลือต้องใช้ควบคู่กับ Postman Collection และ Postman Environment ที่ masterTime จัดเตรียมไว้
- แสดงตัวอย่างและรูปแบบของข้อมูลที่ใช้ในการเรียกใช้งาน APIs ของ masterTime
- แสดงตัวอย่างการใช้งาน APIs ของ masterTime ในรูปแบบของ `Request` และ `Response`
- [ดาวน์โหลด Postman Collection](postman/masterTime%20Integration%20API.postman_collection.json)
- [ดาวน์โหลด Postman Environment](postman/masterTime%20Integration.postman_environment.json)

## API ที่สามารถใช้งานได้

- **พนักงาน (Employee)**
    - การลิสต์ข้อมูลพนักงาน (List Employee)
    - การลบพนักงาน (Delete Employee)
    - การเพิ่มพนักงานใหม่ (New Employee)
    - การแก้ไขข้อมูลพนักงาน (Update Employee)
    - การแก้ไขข้อมูลพนักงานบางฟิลด์ (Update Employee Fields)
    - การขอข้อมูลพนักงานรายคน (Get Employee)


- **สิทธิ์การลงเวลานอกสถานที่ของพนักงาน (Employee Off-Site Grant)**
    - กำหนดสิทธิ์พนักงานลงเวลานอกสถานที่ (Set Employee Off-Site Grant)


- **สถานที่สำหรับลงเวลานอกสถานที่ (Off-Site Location)**
    - การลิสต์ข้อมูลสถานที่สำหรับลงเวลานอกสถานที่ (List Off-Site Location)
    - การลบสถานที่สำหรับลงเวลานอกสถานที่ (Delete Off-Site Location)
    - การเพิ่มสถานที่สำหรับลงเวลานอกสถานที่ (New Off-Site Location)
    - การแก้ไขข้อมูลสถานที่สำหรับลงเวลานอกสถานที่ (Update Off-Site Location)
    - การขอข้อมูลสถานที่สำหรับลงเวลานอกสถานที่ (Get Off-Site Location)


- **หน่วยงาน (Organization)**
    - การลิสต์ข้อมูลหน่วยงาน (List Organization)
    - การลบหน่วยงาน (Delete Organization)
    - การเพิ่มหน่วยงานใหม่ (New Organization)
    - การแก้ไขข้อมูลหน่วยงาน (Update Organization)
    - การขอข้อมูลหน่วยงานรายคน (Get Organization)


- **ตำแหน่งงาน (Position)**
    - การลิสต์ข้อมูลตำแหน่งงาน (List Position)
    - การลบตำแหน่งงาน (Delete Position)
    - การเพิ่มตำแหน่งงานใหม่ (New Position)
    - การแก้ไขข้อมูลตำแหน่งงาน (Update Position)
    - การขอข้อมูลตำแหน่งงานรายคน (Get Position)


- **กะเวลาการทำงาน (Shift)**
    - การลิสต์ข้อมูลกะเวลาการทำงาน (List Shift)
    - การลบกะเวลาการทำงาน (Delete Shift)
    - การเพิ่มกะเวลาการทำงานใหม่ (New Shift)
    - การแก้ไขข้อมูลกะเวลาการทำงาน (Update Shift)
    - การขอข้อมูลกะเวลาการทำงานรายคน (Get Shift)


- **ประเภทพนักงาน (Employee Type)**
    - การลิสต์ข้อมูลประเภทพนักงาน (List Employee Type)
    - การลบประเภทพนักงาน (Delete Employee Type)
    - การเพิ่มประเภทพนักงานใหม่ (New Employee Type)
    - การแก้ไขข้อมูลประเภทพนักงาน (Update Employee Type)
    - การขอข้อมูลประเภทพนักงานรายคน (Get Employee Type)

## Request Body สำหรับการ List ข้อมูล

ทุก API ที่เป็นการ List ข้อมูล จะต้องส่ง Request Body ดังนี้

```json
{
  "conditions": {
    "search": "string",
    "sort": "field_name",
    "order_by": "asc หรือ desc",
    "limit": 5,
    "offset": 0,
    "created_at_gte": "2023-03-01T00:00:00Z",
    "updated_at_gte": "2023-03-01T00:00:00Z"
  }
}
```

**คำอธิบาย**

| ชื่อ Field     | คำอธิบาย                                                                                              | ค่า Default |
|----------------|-------------------------------------------------------------------------------------------------------|-------------|
| search         | string ที่ต้องการใช้ filter ผลของการลิสต์ข้อมูล                                                       | ""          |   
| sort           | field ที่ต้องการ sort ข้อมูลผลลัพธ์                                                                   | ""          |
| order_by       | asc = เรียงจากน้อยไปมาก, desc = เรียงจากมากไปน้อย                                                     | ""          |
| limit          | การจำกัดจำนวนของผลลัพธ์ (0 = ไม่จำกัด)                                                                | 0           |
| offset         | Offset ของข้อมูลเริ่มต้น                                                                              | 0           |
| created_at_gte | filter เฉพาะข้อมูลที่ Create ตั้งแต่เวลานี้เป็นต้นไป (รูปแบบ `RFC3339` เช่น `"2023-03-01T00:00:00Z"`) | ""          |
| updated_at_gte | filter เฉพาะข้อมูลที่ Update ตั้งแต่เวลานี้เป็นต้นไป (รูปแบบ `RFC3339` เช่น `"2023-03-01T00:00:00Z"`) | ""          |

## Request Body สำหรับการลบข้อมูล

ทุก API ที่เป็นการลบข้อมูล จะต้องระบุ "ประเภทของข้อมูล" และ "Code ของข้อมูลที่ต้องการลบ"

**ตัวอย่างเช่น**

```json
{
  "organization": {
    "organization_code": "5000"
  }
}
```

จากตัวอย่างข้างต้น คือการลบข้อมูลประเภทหน่วยงาน `"organization"`
โดยระบุ Code ของหน่วยงานที่ต้องการลบ `"organization_code"`
ที่มีค่าเป็น `"5000"`

## Request Body สำหรับการเพิ่มข้อมูล

ทุก API ที่เป็นการเพิ่มข้อมูล จะต้องระบุ "ประเภทของข้อมูล" และ "ข้อมูลที่ต้องการเพิ่ม"

**ตัวอย่างเช่น**

```json
{
  "organization": {
    "organization_code": "5000",
    "title_th": "หน่วยงานตัวอย่าง",
    "title_en": "Example Organization",
    "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod",
    "root_organization_code": "900000"
  }
}
```

จากตัวอย่างข้างต้น คือการเพิ่มข้อมูลประเภทหน่วยงาน `"organization"`
และ Object ของข้อมูลที่ต้องการเพิ่ม ซึ่งมีค่าเป็น

```json
{
  "organization_code": "5000",
  "title_th": "หน่วยงานตัวอย่าง",
  "title_en": "Example Organization",
  "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod",
  "root_organization_code": "900000"
}
```

## Request Body สำหรับการแก้ไขข้อมูล

ทุก API ที่เป็นการแก้ไขข้อมูล จะต้องระบุ "ประเภทของข้อมูล" และ "ข้อมูลที่ต้องการแก้ไข"

**ตัวอย่างเช่น**

```json
{
  "organization": {
    "organization_code": "5000",
    "title_th": "หน่วยงานตัวอย่าง",
    "title_en": "Example Organization",
    "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod",
    "root_organization_code": "900000"
  }
}
```

จากตัวอย่างข้างต้น คือการแก้ไขข้อมูลประเภทหน่วยงาน `"organization"`
และ Object ของข้อมูลที่ต้องการแก้ไข ซึ่งมีค่าเป็น

```json
{
  "organization_code": "5000",
  "title_th": "หน่วยงานตัวอย่าง",
  "title_en": "Example Organization",
  "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod",
  "root_organization_code": "900000"
}
```

**หมายเหตุ**

- หากเป็น API สำหรับแก้ไขข้อมูลโดยทั่วไปแล้ว จะต้องระบุ Object ของข้อมูลให้ครบทุก Field
- แต่หากเป็น API สำหรับแก้ไขข้อมูลบางฟิลด์ ไม่จำเป็นต้องระบุข้อมูลครบทุกฟิลด์ก็ได้ เช่น
    - การแก้ไขข้อมูลพนักงานบางฟิลด์ (Update Employee Fields)

## Object ของข้อมูลในระบบ masterTime

ในระบบ masterTime มี Object ของข้อมูลที่ใช้ในการจัดเก็บข้อมูลต่างๆ ดังนี้

1. พนักงาน (Employee)
2. สถานที่สำหรับลงเวลานอกสถานที่ (Off-Site Location)
3. หน่วยงาน (Organization)
4. ตำแหน่งงาน (Position)
5. กะเวลาการทำงาน (Shift)
6. ประเภทพนักงาน (Employee Type)

### 1. Object ของข้อมูล "พนักงาน" (Employee)

```json
{
  "employee": {
    "employee_code": "string",
    "firstname_th": "string",
    "lastname_th": "string",
    "firstname_en": "string",
    "lastname_en": "string",
    "identification_type": "string",
    "identification_number": "string",
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

**คำอธิบาย**

- `employee`:
    - ข้อมูลพนักงาน
    - ข้อมูลส่วนนี้จะต้องส่งมาเสมอ
- `options`:
    - ควบคุมการทำงานเพิ่มเติม
    - ข้อมูลส่วนนี้สามารถเป็น `null` ได้
    - ค่า default คือ `null`

**ข้อมูลใน `employee` object**

| ชื่อ Field            | คำอธิบาย                                                                                                                                                                                                          |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| employee_code         | รหัสพนักงาน (จะต้องไม่ซ้ำกัน)                                                                                                                                                                                     |
| firstname_th          | ชื่อจริง (ภาษาไทย)                                                                                                                                                                                                |
| lastname_th           | นามสกุล (ภาษาไทย)                                                                                                                                                                                                 |
| firstname_en          | ชื่อจริง (ภาษาอังกฤษ)                                                                                                                                                                                             |
| lastname_en           | นามสกุล (ภาษาอังกฤษ)                                                                                                                                                                                              |
| identification_type   | ประเภทของ `identification_number`<br/>- `"thai"` = เลขบัตรประชาชน<br/>- `"foreign"` = เลขบัตรต่างด้าว<br/>- `"passport"` = เลข passport)<br/>- สามารถเป็นค่า `null` ได้ถ้า `identification_number` เป็นค่า `null` |
| identification_number | เลขประจำตัวที่ราชการออกให้, สามารถเป็นค่า `null` ได้                                                                                                                                                              |
| organization_code     | รหัสของหน่วยงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                                           |
| position_code         | รหัสของตำแหน่ง (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                                            |
| employee_type_code    | รหัสของประเภทพนักงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                                      |
| shift_code            | รหัสของกะการทำงาน (ต้องตรงกับที่กรอกไว้ในระบบ masterTime)                                                                                                                                                         |
| role_code             | รหัสของสิทธิ์การใช้งาน ในตอนนี้ต้องส่งค่า `NULL`                                                                                                                                                                  |
| optional_information  | ข้อมูลเพิ่มเติม (สามารถเป็น `null` ได้)<br/>- สามารถใช้เก็บข้อมูลอะไรก็ได้<br/>- สามารถใช้ชื่อ field อย่างไรก็ได้<br/>- field ต้องเป็นประเภท string, integer, float, boolean                                      |

**ข้อมูลใน `options` object**

| ชื่อ Field   | คำอธิบาย                                                                                                             |
|--------------|----------------------------------------------------------------------------------------------------------------------|
| unique_field | ระบุเงื่อนไขการตรวจสอบ field ของข้อมูลพนักงานเพิ่มเติมเพื่อป้องการข้อมูลซ้ำซ้อน (default คือ `null` ไม่มีการตรวจสอบ) |

**`unique_field` ที่สามารถใช้งานได้**

- `identification_number`: เพื่อป้องกันไม่ให้ "**เลขประจำตัวที่ราชการออกให้**" ซ้ำกัน

### 2. Object ของข้อมูล "สถานที่สำหรับลงเวลานอกสถานที่" (Off-Site Location)

```json
{
  "offsite_location": {
    "offsite_location_code": "string",
    "title_th": "string",
    "title_en": "string",
    "latitude": "string",
    "longitude": "string",
    "timezone": "string"
  }
}
```

**คำอธิบาย**

| ชื่อ Field            | คำอธิบาย                 |
|-----------------------|--------------------------|
| offsite_location_code | Code ของสถานที่          |
| title_th              | ชื่อสถานที่ (ภาษาไทย)    |
| title_en              | ชื่อสถานที่ (ภาษาอังกฤษ) |
| latitude              | ละติจูด                  |
| longitude             | ลองจิจูด                 |
| timezone              | timezone ของสถานที่      |

### 3. Object ของข้อมูล "หน่วยงาน" (Organization)

```json
{
  "organization": {
    "organization_code": "string",
    "title_th": "string",
    "title_en": "string",
    "description": "string",
    "root_organization_code": "string"
  }
}
```

**คำอธิบาย**

| ชื่อ Field             | คำอธิบาย                                                  |
|------------------------|-----------------------------------------------------------|
| organization_code      | Code ของหน่วยงาน                                          |
| title_th               | ชื่อหน่วยงาน (ภาษาไทย)                                    |
| title_en               | ชื่อหน่วยงาน (ภาษาอังกฤษ)                                 |
| description            | คำอธิบายหน่วยงาน                                          |
| root_organization_code | Code ของหน่วยงานระดับที่สูงกว่า ซึ่งหน่วยงานนี้สังกัดอยู่ |

### 4. Object ของข้อมูล "ตำแหน่งงาน" (Position)

```json
{
  "position": {
    "position_code": "string",
    "title_th": "string",
    "title_en": "string"
  }
}
```

**คำอธิบาย**

| ชื่อ Field    | คำอธิบาย                    |
|---------------|-----------------------------|
| position_code | Code ของตำแหน่งงาน          |
| title_th      | ชื่อตำแหน่งงาน (ภาษาไทย)    |
| title_en      | ชื่อตำแหน่งงาน (ภาษาอังกฤษ) |

### 5. Object ของข้อมูล "กะเวลาการทำงาน" (Shift)

```json
{
  "shift": {
    "shift_code": "string",
    "title_th": "string",
    "title_en": "string",
    "time_in": "string",
    "time_out": "string"
  }
}
```

**คำอธิบาย**

| ชื่อ Field | คำอธิบาย                        |
|------------|---------------------------------|
| shift_code | Code ของกะเวลาการทำงาน          |
| title_th   | ชื่อกะเวลาการทำงาน (ภาษาไทย)    |
| title_en   | ชื่อกะเวลาการทำงาน (ภาษาอังกฤษ) |
| time_in    | เวลาเข้างาน (HH:MM)             |
| time_out   | เวลาเลิกงาน (HH:MM)             |

### 6. Object ของข้อมูล "ประเภทพนักงาน" (Employee Type)

```json
{
  "employee_type": {
    "employee_type_code": "string",
    "title_th": "string",
    "title_en": "string"
  }
}
```

**คำอธิบาย**

| ชื่อ Field | คำอธิบาย                       |
|------------|--------------------------------|
| shift_code | Code ของประเภทพนักงาน          |
| title_th   | ชื่อประเภทพนักงาน (ภาษาไทย)    |
| title_en   | ชื่อประเภทพนักงาน (ภาษาอังกฤษ) |

