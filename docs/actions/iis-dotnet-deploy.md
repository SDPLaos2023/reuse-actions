# Action: Deploy to IIS

Path: `actions/deploy/iis-dotnet-deploy/action.yaml`

Deploy ไฟล์ที่ publish แล้วไปยัง IIS ด้วย `robocopy` พร้อมหยุดและเริ่ม App Pool อัตโนมัติ และล็อกผลลัพธ์

## Inputs

- `exclude_files` (string, default: `appsettings.Development.json appsettings.json web.config`)
  - รายการไฟล์ที่ไม่ต้องการ copy ไปปลายทาง คั่นด้วย space
- `app_pool` (string, default: `DCC-test`)
  - ชื่อ IIS Application Pool
- `deploy_path` (string, default: `C:\inetpub\wwwroot\Dcc-Dev`)
  - โฟลเดอร์ปลายทางที่จะ deploy
- `source_path` (string, default: `Doccontrol\publish`)
  - ตำแหน่งไฟล์ต้นทางที่จะ deploy (เช่น ผลลัพธ์จากขั้นตอน publish)

## พฤติกรรมสำคัญ

- พยายาม Stop App Pool ก่อน copy และ Start App Pool เสมอใน `finally`
- ใช้ `robocopy` แบบ `/E /XO` เพื่อไม่ mirror ลบไฟล์ปลายทาง (รักษาไฟล์ที่ไม่มีในต้นทาง) และไม่ทับไฟล์ใหม่กว่า
- เขียน log ไว้ที่ `%TEMP%/robocopy_yyyyMMdd_HHmmss.log` และแสดงสถานะ exit code อย่างชัดเจน
- หาก exit code > 7 จะถือว่า fail และ exit ด้วย code นั้น

## Prerequisites

- Self-hosted Windows runner
- IIS และ PowerShell module `WebAdministration` พร้อมใช้งาน
- Runner account ต้องมีสิทธิ์จัดการ App Pool และเขียนไฟล์ลง `deploy_path`

## วิธีเรียกใช้ (ร่วมกับ step Build ด้านบน)

```yaml
- name: Deploy to IIS
  uses: ./reuse-actions/actions/deploy/iis-dotnet-deploy
  with:
    exclude_files: "appsettings.Development.json appsettings.json web.config"
    app_pool: "DCC-test"
    deploy_path: 'C:\\inetpub\\wwwroot\\Dcc-Dev'
    source_path: "project/Doccontrol/publish"
```
