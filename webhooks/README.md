# Webhooks

## หัวข้อ

- [Webhooks](#webhooks)
    - [Basic Authentication](#basic-authentication)
    - [Events](#events)
        - [New Transaction Event](#new-transaction-event)


## Webhooks

**หมายเหตุ :** หลังจากนี้ masterTime จะมีรายละเอียดเกี่ยวกับ Webhooks ใน `Developer Portal` ด้วย

ระบบ masterTime ให้บริการ `Event` ผ่านกลไกของ `Webhooks` เพื่อให้ผู้ที่เชื่อมต่อกับระบบ masterTime ได้รับข้อมูลแบบ
`Real-time` โดยมีข้อตกลงเบื่องต้นดังนี้

- ผู้ที่ต้องการรับ Event จาก masterTime จะต้องเป็น Server (`Webhooks receiver`)
- masterTime จะเป็น Client
- `Webhooks receiver` ต้องรองรับ `https` เท่านั้น
- masterTime จะ request ไปด้วย `POST` method
- masterTime จะส่งข้อมูลของ Event ไปใน `Request Body`
- ผู้ที่ต้องการรับ Event จาก masterTime จะต้องเตรียม `Endpoint` ไว้รับแต่ละ `Event` ที่ลงทะเบียน (Subscription) ไว้กับ
  masterTime
- ทุกครั้งที่ masterTime ส่ง Event ไปให้, masterTime จะส่ง `Basic Authentication` ไปใน `HTTP Authorization` header
  โดยใช้เป็น `Basic authentication` ทุกครั้ง
- Event Receiver ต้องตรวจสอบ `IP Whitelist` ของ masterTime เสมอ
- Event Receiver ต้องเป็น `Idempotent` คือ ระบบ masterTime อาจส่ง Event เดิมซ้ำมาหลายครั้ง, Event Receiver
  ต้องสามารถจัดการได้โดยไม่เกิดปัญหา

## Basic Authentication

- ผู้ที่ต้องการรับ Event จาก masterTime จะต้องกำหนด `Username` และ `Password` ให้กับ masterTime
- masterTime จะส่ง Basic Authentication ไปใน `HTTP Authorization` header ทุกครั้งที่ส่ง Event ดังนี้

```
Authorization: Basic {BASE64_ENCODED(Username:Password)}
```

## Timeout 5 วินาที

เมื่อ masterTime ส่ง Event ให้ Event Receiver จะมี `timeout 5 วินาที` ถ้าหาก masterTime ไม่ได้รับ response
ในเวลาที่กำหนดจะถือว่าเกิด Timeout

## Retry Mechanism

เมื่อเกิด Timeout ขึ้น masterTime จะพยายามส่ง Event เดิมทั้งหมด 10 ครั้ง โดยการส่งแต่ละครั้งจะนานขึ้นเรื่อย ๆ
ซึ่งจะห่างกัน
$2^N$ milliseconds (Back-off algorithm)

โดยที่

- N = delivery_attempt - 1

delivery_attempt คือ จำนวนครั้งที่พยายามส่ง Event นี้

## การหยุดส่ง Event

masterTime จะหยุดส่ง Event เมื่อ

- masterTime ได้รับ response 4XX หรือ 5XX
- masterTime พยายามส่ง Event เดิมครบ 10 ครั้ง และไม่ได้รับ response `202 Accepted`

## Events

**Event Header**

- masterTime จะส่ง Basic Authentication ไปใน `HTTP Authorization` header ทุกครั้งที่ส่ง Event ดังนี้

```
Authorization: Basic {BASE64_ENCODED(Username:Password)}
```

**Event body**

```json
{
  "event_uuid": "4f90f1ee-6c54-4b01-90e6-d701748f08534",
  "topic": "employee.transaction.mobile",
  "latest_delivery": "2017-08-29T09:12:33.001Z",
  "first_delivery": "2017-08-29T09:12:33.001Z",
  "delivery_attempt": 2,
  "delivery_state": "open",
  "subscription_uuid": "4f90f1ee-6c54-4b01-90e6-d701748f08534",
  "affected_resource": "api.mastertime.io/v1/resources/transaction/4f90f1ee-6c54-4b01-90e6-d701748f08534",
  "payload": {}
}
```

#### คำอธิบาย

| ชื่อ Field        | คำอธิบาย                                   |
|-------------------|--------------------------------------------|
| event_uuid        | UUID ของ Event                             |
| topic             | Topic ของ Event                            |
| latest_delivery   | วันที่ - เวลา ล่าสุดที่พยายามส่ง Event นี้ |
| first_delivery    | วันที่ - เวลา ครั้งแรกที่ส่ง Event นี้     |
| delivery_attempt  | จำนวนครั้งที่พยายามส่ง Event นี้           |
| delivery_state    | สถานะของ Event นี้                         |
| subscription_uuid | UUID ของผู้ Subscribe Event นี้            |
| affected_resource | URI ของ Resource ที่ส่งมาใน Payload        |
| payload           | ข้อมูลของ Event นี้                        |

##### อธิบาย `delivery_state` เพิ่มเติม

| delivery_state | คำอธิบาย                                                                                                 |
|----------------|----------------------------------------------------------------------------------------------------------|
| open           | Event ที่ masterTime ยังพยายามส่งให้ Event Receiver อยู่ โดยที่ยังไม่สำเร็จหรือยกเลิกการส่งแล้ว          |
| successfully   | Event ที่สำเร็จแล้ว                                                                                      |
| error          | Event ที่ Event Receiver ตอบ Error กลับมา                                                                |
| timeout        | Event ที่ Event Receiver ไม่ตอบกลับมาในเวลาที่ masterTime กำหนด                                          |
| not-delivered  | Event ที่ masterTime ไม่ส่งให้ Event Receiver เช่น การ Subscribe ของ Event Receiver ถูก Disable ชั่วคราว |

**หมายเหตุ :** Events ที่ masterTime จะส่งไปที่ Event Receiver โดยทั่วไปจะเป็น `open` เสมอ

## Topics (Event Types) ทีมีให้บริการ

ระบบ masterTime มี Event types สำหรับ Web Hooks ไว้ให้บริการ ดังนี้

1. New Transaction from mobile app

## 1. Event : New Transaction from mobile app

Event นี้จะส่งเมื่อเกิด Transaction ใหม่ในระบบ masterTime จาก mobile application.

### Topic

```
"topic": "employee.transaction.mobile",
```

### Payload

```json
{
  "company_uuid": "string",
  "transaction_uuid": "string",
  "transaction_time": "2022-03-10T02:29:42Z",
  "timezone": "+07:00",
  "transaction_source": {
    "transaction_source_uuid": "string",
    "transaction_source_code": "string",
    "title_th": "string",
    "title_en": "string",
    "title_short": "string"
  },
  "company_event": null,
  "company_duty": {
    "company_duty_uuid": "string",
    "company_duty_code": "string",
    "title_th": "string",
    "title_en": "string",
    "duty_code": "string",
    "is_office": false,
    "is_display_on_mobile": true,
    "is_tracking_only": false,
    "duty_type": {
      "duty_type_id": 1,
      "title_th": "string",
      "title_en": "string"
    }
  },
  "company_location": {
    "company_location_uuid": "string",
    "company_location_code": "string",
    "title_th": "string",
    "title_en": "string",
    "latitude": "string",
    "longitude": "string",
    "radius_meter": 100,
    "timezone": "+07:00"
  },
  "time_attendance_offsite_grant": {
    "time_attendance_offsite_grant_uuid": "string",
    "time_attendance_offsite_grant_code": "string",
    "start_time": "2021-02-28T23:59:00Z",
    "end_time": "2021-02-28T23:59:00Z",
    "offsite_location": {
      "offsite_location_uuid": "string",
      "offsite_location_code": "string",
      "title_th": "string",
      "title_en": "string",
      "latitude": "string",
      "longitude": "string",
      "timezone": "+07:00"
    }
  },
  "hardware": {
    "hardware_uuid": "string",
    "hardware_code": "string",
    "title_th": "string",
    "title_en": "string",
    "hardware_serial_number": "string",
    "hardware_model": {
      "hardware_model_code": "string",
      "title": "string",
      "hardware_manufacturer": {
        "hardware_manufacturer_code": "string",
        "title": "string"
      }
    }
  },
  "employee": {
    "employee_uuid": "string",
    "employee_code": "string",
    "firstname_th": "string",
    "lastname_th": "string",
    "firstname_en": "string",
    "lastname_en": "string",
    "organization": {
      "organization_uuid": "string",
      "organization_code": "string",
      "title_th": "string",
      "title_en": "string"
    },
    "position": {
      "position_uuid": "string",
      "position_code": "string",
      "title_th": "string",
      "title_en": "string"
    },
    "employee_type": {
      "employee_type_uuid": "string",
      "employee_type_code": "string",
      "title_th": "string",
      "title_en": "string"
    },
    "shift": {
      "shift_uuid": "string",
      "shift_code": "string",
      "title_th": "string",
      "title_en": "string",
      "time_in": "08:00",
      "time_out": "17:00"
    },
    "role": {
      "role_uuid": "string",
      "role_code": "string",
      "title_th": "string",
      "title_en": "string"
    },
    "gender": {
      "gender_id": 1,
      "title_th": "string",
      "title_en": "string"
    },
    "company_prefix": {
      "company_prefix_uuid": "string",
      "company_prefix_code": "string",
      "title_th": "string",
      "title_en": "string"
    }
  },
  "card_serial_number": "string",
  "mobile_unique_code": "string",
  "latitude": "string",
  "longitude": "string",
  "license_plate": "string",
  "qr_code_3rdparty": "string",
  "url_photo": "string",
  "created": "2022-03-10T02:29:55.906267Z"
}
```

### Response

#### 202 (Accepted)

- ได้รับ Event สำเร็จ
- Event Receiver ควรเก็บ Event ลงใน Incoming Event Queue ก่อน แล้วตอบ `202 Accepted` กลับมาที่ masterTime
  เพื่อไม่ให้เกิน Timeout ที่กำหนด
- หลังจากนั้น Event Receiver ค่อยประมวนผล Event ใน Incoming Event Queue แบบ Asynchronous

#### 400 (Bad Request)

- `Request Body` ของ `Event` ที่ masterTime ส่งไป รูปแบบไม่ถูกต้อง
- ผู้รับ Event ต้องตรวจสอบว่า Implement ตามเอกสารล่าสุดของ masterTime หรือไม่

#### 401 (Unauthorized)

- `Username` และ `Password` ใน Basic Authentication ที่ masterTime ส่งไปใน `HTTP Authorization` header ไม่ถูกต้อง
- ผู้รับ Event ต้องตรวจสอบว่าระบุ `Username` และ `Password` ไว้ถูกต้องหรือไม่

#### 404 (Webhook not found / Topic not processed here)

- ไม่พบ Endpoint ของ Event Receiver
- หรือ Event Receiver Endpoint นี้ไม่รองรับ Topic ที่ส่งมา

#### 500 (Server Error)

- เกิดข้อผิดพลาดในระบบของผู้รับ Event

#### 503 (Server Error, Server is overloaded, try again later)

- Server is overloaded

## แนะนำการออกแบบ Architecture ของ Event Receiver Endpoint

### Event Receiver Architecture :

![Event Receiver Architecture](img/event-receiver-architecture.png)

### Event Receiver Behavior :

![Event Receiver Behavior](img/event-receiver-behavior.png)

## Webhooks History API

บางกรณี Webhook Receiver Endpoint อาจไม่ได้รับ Event จาก masterTime วึ่งอาจเกิดได้จากหลายสาเหตุ

อย่างไรก็ตาม masterTime ได้เตรียม Webhooks History API ไว้ให้

## การทดสอบ Webhooks

### 1. ทดสอบผ่าน Postman

ผู้ที่พัฒนา Event Receiver สามารถให้ [Postman](https://www.postman.com/) เพื่อทดสอบ Webhooks ได้ตาม spec ของ masterTime
ตามเอกสารก่อนหน้านี้

### 2. ทดสอบผ่าน Webhooks Ping API

**หมายเหตุ :** รายละเอียดในส่วนนี้อยู่ใน masterTime's Developer Portal

# Developer Portal

masterTime พัฒนา Developer Portal ไว้เพื่ออำนวยความสะดวกแก่ผู้ที่ต้องการใช้ Webhooks ของ masterTime ซึ่ง Developer Portal สามารถ
1. Documents
    - Topic / Events
    - IP Whitelist ของ masterTime
    - เอกสาร API
2. จัดการ Webhooks
    - Enable / Disable Webhook Events
3. Dashboard
    - ดู Event Log
    - filter ข้อมูล Event Log
4. Testing Webhooks ด้วย `Webhooks Ping API`

