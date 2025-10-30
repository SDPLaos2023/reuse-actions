# ขั้นตอนการติดตั้ง Self-Hosted Runners สำหรับ GitHub Actions เครื่อง windows แบบ Domain Join

- สร้าง user ใหม่บนเครื่องที่ต้องการตืดตั้ง self-hosted runner (กรณีที่เป็น windows server เพราะ user NT AUTHORITY\SYSTEM จะไม่มีสิทธิ์ในการรัน service บนเครื่องที่ join domain)
  - `net user github-runner P@ssw0rd! /add` user นี้จะไม่ใช่ local user แต่จะเป็น user ที่ join กับ domain
    - `runas /user:github-runner cmd` -> `whoami` เข้าไปเช็ค usergroup
  - Set password expire
    - `Set-ADUser -Identity "github-runner" -PasswordNeverExpires $true` ตั้งค่า password ไม่ให้หมดอายุ (domain user)
    - `Set-LocalUser -Name "github-runner" -PasswordNeverExpires $true` (local user)
  - Add user github-runner เข้าไปใน Local policy ให้สิทธิ์ Log on as a service
    - เข้าไปที่ Local Security Policy > Local Policies > User Rights Assignment > Log on as a service > เพิ่ม user github-runner เข้าไป
  - `net localgroup Administrators <user> /add`
- หลังจากนั้นตั้งติด Document ของ Github ได้เลย
  - ในขั้นตอนการรัน service ให้เลือกใช้ user <Domain>\github-runner ที่สร้างขึ้นมา (ชื่อจะสามารถดูได้จากผลลัพธ์หลังจากการ Add user เข้าไปใน Log on as a service)
