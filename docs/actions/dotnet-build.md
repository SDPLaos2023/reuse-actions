# Action: Build and Test .NET Application

Path: `actions/build/dotnet-build/action.yaml`

Builds, tests, and publishes a .NET application. เหมาะสำหรับใช้งานร่วมกับขั้นตอน deploy ต่อไป

## Inputs

- `source_code_path` (string, default: `./`)
  - Path ไปยังโฟลเดอร์ของ .NET solution หรือ project ที่ต้องการ build/test/publish
- `configuration` (string, default: `Release`)
  - ค่า configuration สำหรับ build เช่น `Release` หรือ `Debug`
- `publish_output_path` (string, default: `publish`)
  - โฟลเดอร์ output สำหรับไฟล์ที่ publish แล้ว (relative ต่อ `source_code_path`)

## ทำอะไรบ้าง

1. `dotnet restore`
2. `dotnet build --configuration <configuration> --no-restore`
3. `dotnet test --no-build`
4. `dotnet publish -c <configuration> -o <publish_output_path>`

## Prerequisites

- Self-hosted Windows runner พร้อม .NET SDK ตรงกับเวอร์ชันของโปรเจกต์
- ใช้ shell: PowerShell ตามที่กำหนดใน action

## วิธีเรียกใช้ (เมื่อ checkout repo นี้เป็น subfolder ชื่อ `reuse-actions`)

```yaml
- name: Build .NET Application
  uses: ./reuse-actions/actions/build/dotnet-build
  with:
    source_code_path: "project/Doccontrol"
    configuration: "Release"
    publish_output_path: "publish"
```

ผลลัพธ์ publish จะอยู่ที่ `project/Doccontrol/publish`
