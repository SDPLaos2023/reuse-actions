# ขั้นตอนการติดตั้ง Self-Hosted Runners สำหรับ GitHub Actions เครื่อง windows แบบ Domain Join

- สร้าง user ใหม่บนเครื่องที่ต้องการตืดตั้ง self-hosted runner (กรณีที่เป็น windows server เพราะ user NT AUTHORITY\SYSTEM จะไม่มีสิทธิ์ในการรัน service บนเครื่องที่ join domain)
  - `net user github-runner P@ssw0rd! /add` user นี้จะไม่ใช่ local user แต่จะเป็น user ที่ join กับ domain
  - `Set-ADUser -Identity "github-runner" -PasswordNeverExpires $true` ตั้งค่า password ไม่ให้หมดอายุ
  - Add user github-runner เข้าไปใน Local policy ให้สิทธิ์ Log on as a service
    - เข้าไปที่ Local Security Policy > Local Policies > User Rights Assignment > Log on as a service > เพิ่ม user github-runner เข้าไป
- หลังจากนั้นตั้งติด Document ของ Github ได้เลย
  - ในขั้นตอนการรัน service ให้เลือกใช้ user github-runner ที่สร้างขึ้นมา
