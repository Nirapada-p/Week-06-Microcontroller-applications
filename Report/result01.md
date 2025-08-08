
# บันทึกภาพตาราง ใส่ในไฟล์ส่งงาน
<img width="1629" height="758" alt="image" src="https://github.com/user-attachments/assets/e94d40f8-533c-40ea-891c-2bc9bff68efa" />

# แนบไฟล์ size-components.txt ในโฟลเดอร์ส่งงาน
<img width="1563" height="571" alt="image" src="https://github.com/user-attachments/assets/8c9f35eb-c198-4a34-b90e-0e72b1debd7e" />

# แนบไฟล์ size-files.txt ในโฟลเดอร์ส่งงาน
<img width="1637" height="756" alt="image" src="https://github.com/user-attachments/assets/b434e576-b64b-4744-9194-a0b085cb8e01" />

<img width="1630" height="759" alt="image" src="https://github.com/user-attachments/assets/573f171e-ff7e-40f0-ad25-8de0544dfef3" />

<img width="1645" height="760" alt="image" src="https://github.com/user-attachments/assets/d7fe31e6-691a-4e21-812d-d37ed8cec215" />

<img width="1622" height="745" alt="image" src="https://github.com/user-attachments/assets/68bbe369-17c8-4c6c-9274-92a331c9916a" />

# การ Flash และทดสอบ
<img width="1073" height="726" alt="image" src="https://github.com/user-attachments/assets/580675c7-dc72-40db-a442-b4a67d3fe115" />

# บันทึกผลการ simulate ในโฟลเดอร์ส่งงาน
<img width="885" height="832" alt="image" src="https://github.com/user-attachments/assets/f11583f8-45af-43f0-a359-9cab8f9c8657" />

# 🔍 คำถามทบทวน
### Docker vs Native Setup: อธิบายข้อดีของการใช้ Docker เปรียบเทียบกับการติดตั้ง ESP-IDF บน host system?
  - Docker  ดึง image มา แล้วก็ใช้ได้เลย ไม่ต้องไปไล่ลง Python 
  - Native  เตรียมลงไลบรารีครบชุด ถ้าผิดเวอร์ชันคือจบ 
  - ถ้าอยาก เร็วสุด และไม่กลัวเครื่องเลอะ → Native  ถ้าอยาก ไม่ปวดหัว และให้ทีมใช้เหมือนกันหมด → Docker
### Build Process: อธิบายขั้นตอนการ build ของ ESP-IDF ใน Docker container ตั้งแต่ source code จนได้ binary?
- Source Code Mount เข้า Container
- Docker container มี ESP-IDF ติดตั้งพร้อมแล้ว
- โค้ดโปรเจกต์ของเราถูก mount จาก host → container เพื่อให้ build ได้
- CMake Configure
- รัน idf.py set-target และ idf.py configure
- CMake สร้างโครงสร้าง build (build directory + config เช่น sdkconfig)
- Build with Ninja
- รัน idf.py build → CMake เรียก Ninja เพื่อ compile source (.c/.cpp)
- Compiler (xtensa-esp32-elf-gcc) แปลงเป็น object files (.o)
- Linking
- Linker รวม object files + libraries ของ ESP-IDF เป็นไฟล์ .elf
- Generate Binary
- เครื่องมือ esptool.py แปลง .elf เป็น .bin หลายไฟล์ (bootloader.bin, partition-table.bin, firmware.bin)
- Output Ready
- Binary พร้อม flash อยู่ใน build/ ของโค้ดเรา (ยังอยู่บน host เพราะ mount ไว้)

### CMake Files: บทบาทของไฟล์ CMakeLists.txt แต่ละไฟล์คืออะไร และทำงานอย่างไรใน Docker environment?
  บทบาทของ CMakeLists.txt ไม่เปลี่ยนเลยไม่ว่ารันใน Docker หรือ Native ความต่างคือ Docker คุมเวอร์ชันของ CMake, compiler, และ ESP-IDF ให้ตายตัว → ลดปัญหา “เครื่องนึง build ผ่าน อีกเครื่องพัง”

### Git Ignore: ไฟล์ .gitignore มีความสำคัญอย่างไรสำหรับ ESP32 project development?
 - .gitignore ใน ESP32 project = ด่านหน้า กันไฟล์ที่ไม่จำเป็น, ไฟล์เฉพาะเครื่อง, และไฟล์ชั่วคราว ไม่ให้เข้ามาใน Git → ทำให้โปรเจกต์เบา, สะอาด, และทำงานร่วมกับคนอื่นได้ราบรื่น
  
### Container Persistence: ข้อมูลใดบ้างที่จะหายไปเมื่อ restart container และข้อมูลใดที่จะอยู่ต่อ?
  - หากข้อมูลถูกจัดเก็บใน volume หรือ bind mount ข้อมูลจะคงอยู่แม้ container ถูกหยุดหรือเริ่มใหม่ เนื่องจากข้อมูลถูกเก็บภายนอกตัว container
  - หากข้อมูลถูกจัดเก็บภายใน filesystem ของ container โดยตรง ข้อมูลจะสูญหายเมื่อ container ถูกลบ หรือสร้างใหม่ เพราะเป็นพื้นที่เก็บข้อมูลชั่วคราวของ container

### Development Workflow: เปรียบเทียบ workflow การพัฒนาระหว่างการใช้ Docker กับการทำงานบน native system?
  - Docker → เหมือนทำงานใน “ห้องแล็บที่จัดทุกอย่างไว้พร้อม” แค่เข้าไปก็เริ่มได้ ไม่ต้องเตรียมเอง
  - Native → เหมือนทำงานที่บ้าน จะทำอะไรก็สะดวก แต่ต้องดูแลอุปกรณ์เองทั้งหมด

