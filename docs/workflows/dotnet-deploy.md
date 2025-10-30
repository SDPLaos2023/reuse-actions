# Reusable Workflow: Deploy .NET Application from Tag

Path: `.github/workflows/dotnet-deploy.yaml`

Workflow สำหรับ deploy จาก tag ที่ระบุ โดยจะ checkout tag นั้น สร้างผลลัพธ์ publish และ deploy ไป IIS

## Inputs

- `run_on` (string, default: `dev`) — runner label เพิ่มเติม นอกเหนือจาก `self-hosted`
- `tag_version` (string, required) — tag ที่ต้องการ deploy เช่น `v1.0.0`
- `source_code_path` (string, default: `./`) — path ของซอร์สโค้ดใน repo ปลายทาง
- `publish_output_path` (string, default: `publish`) — โฟลเดอร์ output หลัง publish
- `exclude_files` (string, default: `appsettings.Development.json appsettings.json web.config`) — รายการไฟล์ที่ไม่ต้อง copy
- `app_pool` (string, default: `DCC-test`) — ชื่อ IIS Application Pool
- `deploy_path` (string, default: `C:\inetpub\wwwroot\Dcc-Dev`) — โฟลเดอร์ปลายทางบน IIS

## Permissions & Secrets

- เพียง `contents: read` ก็เพียงพอเพราะไม่ต้องเขียน tag เพิ่ม
- ใช้ `secrets: inherit` เพื่อดึง secrets (เช่น token) จาก repo ปลายทาง หากจำเป็น

## Runner ที่ใช้

- `runs-on: ["self-hosted", "${{ inputs.run_on }}"]`
  - ต้องมี self-hosted Windows runner ที่ติด label ตรงกับค่า `run_on` เช่น `dev`

## ตัวอย่างการเรียกใช้จาก repo อื่น

```yaml
name: DEV - Deploy

on:
  workflow_dispatch:
    inputs:
      tag_version:
        description: "Tag version to deploy (e.g., v1.0.0)"
        required: true
        type: string
      exclude_files:
        description: "Files to exclude (space separated)"
        required: false
        default: "appsettings.Development.json appsettings.json web.config"

jobs:
  call-deploy-from-tag:
    uses: SDPLaos2023/reuse-actions/.github/workflows/dotnet-deploy.yaml@main
    permissions:
      contents: read
    secrets: inherit
    with:
      run_on: "dev"
      tag_version: ${{ github.event.inputs.tag_version }}
      source_code_path: "Doccontrol"
      publish_output_path: "publish"
      exclude_files: ${{ github.event.inputs.exclude_files }}
      app_pool: "DCC-test"
      deploy_path: 'C:\\inetpub\\wwwroot\\Dcc-Dev'
```

## ขั้นตอนภายในแบบย่อ

1. Checkout โค้ดตาม tag ที่ระบุลงโฟลเดอร์ `project`
2. Checkout repo `reuse-actions` ลงโฟลเดอร์ `reuse-actions`
3. Build/Publish ด้วย Composite Action `actions/build/dotnet-build`
4. Deploy ไป IIS ด้วย Composite Action `actions/deploy/iis-dotnet-deploy`
