# 🧰 WSL2 Bridged Network Setup with Hyper-V

เอกสารนี้อธิบายวิธีการตรวจสอบและตั้งค่า Hyper-V รวมถึงการเชื่อมต่อเครือข่ายแบบ Bridged ให้กับ WSL2 บน Windows

---

## 📌 ข้อกำหนดเบื้องต้น

- Windows 10/11 Pro, Enterprise หรือ Education
- รองรับ Hyper-V
- มี Ethernet adapter สำหรับ bridged network

---

## ⚠️ คำเตือน: ต้องเปิด PowerShell แบบ Run as Administrator

ก่อนรันคำสั่งใด ๆ ให้ทำตามนี้:

1. กดปุ่ม `Windows` แล้วพิมพ์ “PowerShell”
2. คลิกขวาที่ **Windows PowerShell**
3. เลือก **Run as administrator**

---

## ✅ ขั้นตอนที่ 1: ตรวจสอบว่าเครื่องรองรับ Hyper-V หรือไม่

รันคำสั่ง:

```powershell
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```

หากระบบรองรับ Hyper-V จะพบข้อความคล้าย:

```
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22631 N/A Build 22631
```

---

## 🔧 ขั้นตอนที่ 2: เปิดใช้ Hyper-V (ถ้ายังไม่ได้เปิด)

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

> ถ้าเครื่องไม่รู้จักคำสั่งข้างต้น ให้ลอง:
```powershell
Get-Command -Module Hyper-V
```

หลังจากเปิดใช้งานแล้ว ให้รีสตาร์ทเครื่อง

---

## 🌐 ขั้นตอนที่ 3: สร้าง External Virtual Switch

1. ตรวจสอบชื่อ Network Adapter จริง:

```powershell
Get-NetAdapter
```

ตัวอย่างผลลัพธ์:

```
Name           InterfaceDescription
----           --------------------
Ethernet       Intel(R) Ethernet Controller
Wi-Fi          Intel(R) Wi-Fi 6 AX201
```

2. สร้าง Virtual Switch:

```powershell
New-VMSwitch -Name "External-Switch" -NetAdapterName "Ethernet" -AllowManagementOS $true
```

> 📝 **คำอธิบายเพิ่มเติม:**  
> ชื่อ `"External-Switch"` คือชื่อที่คุณสามารถตั้งเองได้ตามต้องการ เช่น `"WSL-Bridge"` หรือ `"BridgedNet"`  
> แต่ **ต้องใช้ชื่อเดียวกันนี้ในไฟล์ `.wslconfig`** ในบรรทัด `vmSwitch=<ชื่อที่ตั้งไว้>`  
> เช่น ถ้าตั้งชื่อสวิตช์ว่า `"WSL-Bridge"` ก็ต้องเขียนใน `.wslconfig` แบบนี้:
> ```ini
> vmSwitch=WSL-Bridge
> ```

---

## 🧾 ขั้นตอนที่ 4: ตั้งค่า `.wslconfig` ให้ WSL2 ใช้ Bridged Network

1. เปิด Notepad หรือ Text Editor
2. สร้างไฟล์ชื่อ `.wslconfig` ที่โฟลเดอร์ `%USERPROFILE%`  
   เช่น `C:\Users\<ชื่อคุณ>\.wslconfig`
3. ใส่เนื้อหาต่อไปนี้:

```ini
[wsl2]
networkingMode=bridged
vmSwitch=External-Switch
ipv6=true
```

> 📌 อย่าลืมว่า `vmSwitch=` ต้องตรงกับชื่อที่คุณตั้งไว้ตอน `New-VMSwitch`

4. บันทึกไฟล์

---

## 🔁 ขั้นตอนที่ 5: รีสตาร์ท WSL2

เปิด PowerShell แล้วรัน:

```powershell
wsl --shutdown
```

เปิด WSL ใหม่อีกครั้งเพื่อให้ค่ามีผล

---

## 🧪 ขั้นตอนที่ 6: ตรวจสอบว่า WSL2 ได้รับ IP แบบ Bridged

ใน WSL shell รันคำสั่ง:

```bash
ip addr
```

คุณควรเห็น IP ที่อยู่ในช่วงเดียวกับ LAN (ที่ Router แจก)

---

## 🛠 ตรวจสอบ Virtual Switch ที่มีอยู่แล้ว

รันคำสั่ง:

```powershell
Get-VMSwitch
```

หากมี `External-Switch` และ `SwitchType` เป็น `External` แสดงว่าสร้างสำเร็จ

ดูรายละเอียดเพิ่มเติม:

```powershell
Get-VMSwitch -Name "External-Switch" | Format-List *
```

---

## 🧹 ลบ Virtual Switch (ถ้าต้องการเริ่มใหม่)

```powershell
Remove-VMSwitch -Name "External-Switch"
```

---

## 🎉 เสร็จสิ้น

ตอนนี้ WSL2 ของคุณเชื่อมต่อแบบ Bridged ผ่าน Hyper-V แล้ว พร้อมใช้งานในเครือข่ายเหมือนเป็นเครื่องอิสระในระบบ LAN!
