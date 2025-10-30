# reuse-actions Documentation

เอกสารรวมวิธีใช้งาน shared GitHub Actions และ Reusable Workflows ที่อยู่ใน repository นี้ โดยแยกเป็นหมวดหมู่ชัดเจน พร้อมตัวอย่างเรียกใช้จาก repository อื่น

- Actions (Composite Actions)
  - [Build: .NET Build & Test](./actions/dotnet-build.md)
  - [Deploy: IIS .NET Deploy](./actions/iis-dotnet-deploy.md)
  - [Git: Create Semantic Tag](./actions/create-semantic-tag.md)
- Reusable Workflows
  - [Build, Test, Deploy (.NET → IIS)](./workflows/dotnet-build-test-deploy.md)
  - [Deploy from Tag](./workflows/dotnet-deploy.md)
- Runner Setup (Windows Self-hosted)
  - ดูขั้นตอนการติดตั้ง Runner ที่ไฟล์เดิม: [../readme.md](../readme.md) (เนื้อหาเดิม ไม่เปลี่ยนแปลง)

## Quickstart (เรียกใช้จากโปรเจกต์อื่น)

1. สร้าง workflow ใน repo ปลายทาง เพื่อเรียก Reusable Workflow จาก repo นี้

ตัวอย่าง: เรียกใช้ workflow build + test + deploy

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

ตัวอย่าง: เรียกใช้ workflow deploy จาก tag

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

2. ตั้งค่า runner labels ให้ตรงกับ `run_on` (เช่น `dev`) บน self-hosted runner ฝั่ง Windows
3. ตรวจสอบ prerequisites ของ action ที่ใช้ (ดูในเอกสารของแต่ละ action)

## โครงสร้างที่เกี่ยวข้องใน repo นี้

- `.github/workflows/`
  - `dotnet-build-test-deploy.yaml` — Reusable Workflow สำหรับ build → test → deploy และสร้าง semantic tag
  - `dotnet-deploy.yaml` — Reusable Workflow สำหรับ deploy จาก tag ที่ระบุ
- `actions/`
  - `build/dotnet-build/action.yaml` — Composite Action: build + test + publish
  - `deploy/iis-dotnet-deploy/action.yaml` — Composite Action: deploy ไป IIS ด้วย robocopy, stop/start app pool
  - `git/create-sematic-tag/action.yaml` — Composite Action: สร้าง semantic version tag และ push ไปที่ remote

อ้างอิงการติดตั้ง runner: [../readme.md](../readme.md)
