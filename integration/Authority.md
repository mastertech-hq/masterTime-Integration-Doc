#เกี่ยวกับพนักงาน
ประกอบด้วย API ดังนี้
1. Update Employee Group Member
2. Delete Employee Group Member
3. Update Authority Employee
4. Delete Authority Employee

## Update Employee Group Member
```
POST https://api.mastertime.io/api/v1/integration/std/func/update-employee-member
```

```json
{
    "employee_group_member": {
        "employee_group_code": "0006",
        "employee_codes": [
            "5405017",
            "5208040",
            "123456789",
            "5405017000123"
        ]
    }
}
```

**คำอธิบาย**
|Field Name|Data type|Description|
|---|---|---|
|employee_group_code|string|รหัสกลุ่มพนักงาน|
|employee_codes|Array|รหัสพนักงานที่ต้องการ|

## Delete Employee Group Member
```
POST https://api.mastertime.io/api/v1/integration/std/func/delete-employee-member
```
```json
{
    "employee_group_member": {
        "employee_group_code": "0006",
        "employee_codes": [
            "5405017",
            "5208040",
            "123456789",
            "5405017000123"
        ]
    }
}
```
**คำอธิบาย**
|Field Name|Data type|Description|
|---|---|---|
|employee_group_code|string|รหัสกลุ่มพนักงาน|
|employee_codes|Array|รหัสพนักงานที่ต้องการ|

## Update Authority Employee
```
POST https://api.mastertime.io/api/v1/integration/std/func/update-authority-employee
```
```json
{
    "authority_employee": {
        "authority_code": "Z001",
        "employee_codes": [
            "5405017",
            "123456789",
            "uuid"
        ]
    }
}
```
**คำอธิบาย**
|Field Name|Data type|Description|
|---|---|---|
|authority_code|string|รหัสกลุ่มสิทธิ|
|employee_codes|Array|รหัสพนักงานที่ต้องการ|

## Delete Authority Employee
```
POST https://api.mastertime.io/api/v1/integration/std/func/delete-authority-employee
```
```json
{
    "authority_employee": {
        "authority_code": "Z001",
        "employee_codes": [
            "5405017",
            "123456789",
            "uuid"
        ]
    }
}
```
**คำอธิบาย**
|Field Name|Data type|Description|
|---|---|---|
|authority_code|string|รหัสกลุ่มสิทธิ|
|employee_codes|Array|รหัสพนักงานที่ต้องการ|
