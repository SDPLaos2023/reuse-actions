# Action: Create Semantic Tag

Path: `actions/git/create-sematic-tag/action.yaml`

สร้างและ push semantic version tag (`vMAJOR.MINOR.PATCH`) ไปยัง remote โดยอ้างอิง tag ล่าสุดจาก `origin`

## Inputs

- `bump_type` (string, default: `patch`) — `major|minior|patch`
- `source_code_path` (string, default: `./`) — Path ของ repo (หากต้องการ cd ก่อนรัน git)

## ทำงานอย่างไร (สรุป)

1. ตั้งค่า git user
2. อ่าน tag ปัจจุบันจาก `origin` ที่รูปแบบ `vX.Y.Z`
3. คำนวณเวอร์ชันใหม่ตาม `bump_type` (major/minor/patch)
4. สร้าง annotated tag พร้อมข้อความสรุป และ `git push origin <tag>`

> Note: หาก push ล้มเหลวจะลอง `--force` อีกครั้ง

## Prerequisites

- ให้สิทธิ์ workflow ในการเขียน `contents: write` และ/หรือใช้ PAT ผ่าน `secrets` หากจำเป็น
- ตัวอย่างด้านล่างสืบทอด secrets จาก repo ที่เรียกใช้

## วิธีเรียกใช้

```yaml
- name: Create Semantic Version Tag
  uses: ./reuse-actions/actions/git/create-sematic-tag
  with:
    bump_type: patch # หรือ minor / major
    source_code_path: "project/Doccontrol"
```
