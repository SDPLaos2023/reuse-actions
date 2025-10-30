# Reusable Workflow: Build test and Deploy .NET Application to IIS

Path: `.github/workflows/dotnet-build-test-deploy.yaml`

Workflow สำหรับ repo อื่นเรียกใช้เพื่อ: Checkout โค้ด → Build → Test → Publish → Deploy ไป IIS → สร้าง semantic tag

## Inputs

- `run_on` (string, default: `dev`) — runner label เพิ่มเติม นอกเหนือจาก `self-hosted`
- `source_code_path` (string, default: `./`) — path ของซอร์สโค้ดใน repo ปลายทาง
- `publish_output_path` (string, default: `publish`) — โฟลเดอร์ output หลัง publish
- `exclude_files` (string, default: `appsettings.Development.json appsettings.json web.config`) — รายการไฟล์ที่ไม่ต้อง copy
- `app_pool` (string, default: `DCC-test`) — ชื่อ IIS Application Pool
- `deploy_path` (string, default: `C:\inetpub\wwwroot\Dcc-Dev`) — โฟลเดอร์ปลายทางบน IIS
- `bump_type` (string, default: `patch`) — ประเภทการเพิ่มเวอร์ชัน tag: `patch|minor|major`

## Permissions

- แนะนำให้เซ็ต `permissions: contents: write` เมื่อเรียกใช้ เพื่อให้สร้างและ push tag ได้
- สามารถ `secrets: inherit` เพื่อรับ secrets จาก repo ปลายทาง

## Runner ที่ใช้

- `runs-on: ["self-hosted", "${{ inputs.run_on }}"]`
  - ต้องมี self-hosted Windows runner ที่ติด label ตรงกับค่า `run_on` เช่น `dev`

## ตัวอย่างการเรียกใช้จาก repo อื่น

```yaml
name: DEV - Build and Deploy

on:
  workflow_dispatch:
    inputs:
      exclude_files:
        description: "ระบุไฟล์หรือโฟลเดอร์ที่ไม่ต้องการ copy (คั่นด้วย space เช่น web.config appsettings.json)"
        required: false
        default: "appsettings.Development.json appsettings.json web.config"
      bump_version:
        description: "Version bump type (patch|minor|major)"
        required: false
        default: "patch"
        type: choice
        options: [patch, minor, major]

jobs:
  call-dotnet-build-deploy:
    uses: SDPLaos2023/reuse-actions/.github/workflows/dotnet-build-test-deploy.yaml@main
    permissions:
      contents: write
    secrets: inherit
    with:
      run_on: "dev"
      source_code_path: "Doccontrol"
      publish_output_path: "publish"
      exclude_files: ${{ github.event.inputs.exclude_files }}
      app_pool: "DCC-test"
      deploy_path: 'C:\\inetpub\\wwwroot\\Dcc-Dev'
      bump_type: ${{ github.event.inputs.bump_version }}
```

## สิ่งที่ workflow ทำภายใน

1. Checkout repo ปลายทางไว้ในโฟลเดอร์ `project`
2. Checkout repo `reuse-actions` ไว้ในโฟลเดอร์ `reuse-actions`
3. เรียก Composite Action: Build (`actions/build/dotnet-build`)
4. เรียก Composite Action: Deploy (`actions/deploy/iis-dotnet-deploy`)
5. เรียก Composite Action: Create Semantic Tag (`actions/git/create-sematic-tag`)

## หมายเหตุ

- ตรวจสอบ prerequisites ของแต่ละ action (เช่น .NET SDK, IIS, สิทธิ์ของ runner)
- `deploy_path` และ `app_pool` ควรตรงกับสภาพแวดล้อมจริงของ IIS
