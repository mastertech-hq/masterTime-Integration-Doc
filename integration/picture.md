# รูปพนักงาน
ประกอบด้วย API ดังนี้
1. New File Jpeg
2. Delete File Jpeg

## New File Jpeg

```
POST https://api.mastertime.io/api/v1/integration/std/func/new-file-jpeg
```
```json
{
    "file_jpeg": {
        "employee_code": "5405017",
        "is_photo_identifier": true,
        "is_face": true,
        "file_jpeg_base64":""
    }
}
```
**คำอธิบาย**
|Field Name|Data type|Description|
|---|---|---|
|employee_code|string|รหัสพนักงาน|
|is_photo_identifier|bool|true ใช้เป็นรูปโปรไฟล์|
|is_face|bool|true ใช้ในการสแกนใบหน้า|
|file_jpeg_base64|string|เป็นรูปที่แปลงเป็น base64|


## Delete File Jpeg

```
POST https://api.mastertime.io/api/v1/integration/std/func/delete-file-jpeg
```
```json
{
    "file_jpeg": {
        "employee_code": "5405017",
        "is_photo_identifier": true,
        "is_face": true
    }
}
```
**คำอธิบาย**
|Field Name|Data type|Description|
|---|---|---|
|employee_code|string|รหัสพนักงาน|
|is_photo_identifier|bool|true ลบรูปที่เป็นโปรไฟล์|
|is_face|bool|true ลบรูปที่ใช้ในการสแกนใบหน้า|
