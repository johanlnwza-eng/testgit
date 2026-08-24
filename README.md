# Priority To-Do Manager

ระบบจัดการงาน/To-Do แบบมีลำดับความสำคัญ พัฒนาด้วย Python + Tkinter (GUI)

## วิธีรัน

```bash
python main.py
```

ต้องมี Python 3.8+ (มี Tkinter ติดตั้งมาพร้อม Python อยู่แล้วบน Windows/Mac
ส่วน Linux บางจอต้องลง `sudo apt install python3-tk`)

## โครงสร้างไฟล์

```
todo_priority_app/
├── task.py           # Task (abstract), DeadlineTask, RecurringTask
├── category.py       # Category
├── task_manager.py   # TaskManager - business logic
├── app.py            # TodoApp - GUI (Tkinter)
├── main.py           # entry point
└── README.md
```

## คลาสในระบบ (6 คลาส)

| คลาส | หน้าที่ |
|---|---|
| `Task` (abstract) | คลาสแม่ กำหนดโครงสร้าง/attribute พื้นฐานของงานทุกชนิด |
| `DeadlineTask` | งานที่มีกำหนดส่ง คะแนนความสำคัญเพิ่มเมื่อใกล้/เลยกำหนด |
| `RecurringTask` | งานที่ทำซ้ำเป็นรอบ (daily/weekly/monthly) |
| `Category` | หมวดหมู่ของงาน |
| `TaskManager` | จัดการ list ของ Task ทั้งหมด (CRUD, sort, search, filter, save/load) |
| `TodoApp` | ส่วน GUI เชื่อมผู้ใช้กับ TaskManager |

## ตำแหน่งหลักการ OOP (สำหรับ present)

### 1. Abstraction
**ไฟล์:** `task.py` คลาส `Task`
- ประกาศเป็น `class Task(ABC)` และมี `@abstractmethod calculate_priority_score()`
  ที่ไม่มี implementation ในคลาสแม่ — บังคับให้ subclass ต้องเขียนสูตรคำนวณเอง
  ห้ามสร้าง instance ของ `Task` ตรงๆ ได้เลย

### 2. Encapsulation
**ไฟล์:** `task.py`, `category.py` (ทุก class)
- attribute ทั้งหมดเป็น private เช่น `self.__id`, `self.__title`,
  `self.__is_completed`
- เข้าถึง/แก้ไขได้ผ่าน `@property` (getter) และ `@x.setter` เท่านั้น
- setter มีการ validate เช่น `title.setter` ห้ามตั้งชื่อว่าง,
  `priority.setter` ห้ามตั้งค่านอกเหนือ 1-3

### 3. Inheritance
**ไฟล์:** `task.py`
- `class DeadlineTask(Task):` และ `class RecurringTask(Task):`
  สืบทอด attribute/method ทั้งหมดจาก `Task` แล้วเพิ่มของเฉพาะตัว
  (`penalty_weight` / `recurrence_interval`)
- ใช้ `super().__init__(...)` เรียก constructor ของคลาสแม่

### 4. Polymorphism
**ไฟล์:** `task.py`, `task_manager.py`
- `calculate_priority_score()` ถูก override คนละสูตรใน `DeadlineTask`
  (คำนวณจากความใกล้ deadline) กับ `RecurringTask` (คำนวณจากรอบทำซ้ำ)
- `mark_complete()` ถูก override ใน `RecurringTask` ให้พฤติกรรมต่างจาก
  `Task` ต้นฉบับ (รีเซ็ตกลับเป็นรอบถัดไปแทนที่จะค้างสถานะเสร็จถาวร)
- `TaskManager.sort_by_priority()` เรียก `t.calculate_priority_score()`
  กับ task ทุกตัวแบบเดียวกัน โดยไม่สนใจว่า task ตัวนั้นเป็นชนิดไหน
  (runtime polymorphism ผ่าน method overriding)

## ฟีเจอร์หลักของระบบ

- เพิ่ม/ลบ/แก้ไขสถานะงาน ผ่าน GUI
- แสดงรายการงานเรียงตามคะแนนความสำคัญอัตโนมัติ
- ค้นหางานจากชื่อ/รายละเอียด
- กรองแสดง/ซ่อนงานที่เสร็จแล้ว
- บันทึก/โหลดข้อมูลเป็นไฟล์ JSON
- แจ้งเตือนจำนวนงานที่เลยกำหนดส่ง
