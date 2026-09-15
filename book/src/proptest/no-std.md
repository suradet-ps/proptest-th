# การรองรับ `no_std`

Proptest มีการรองรับการทำงานในสภาพแวดล้อม `no_std` แบบจำกัด (partial support)

คุณจำเป็นต้องใช้คอมไพเลอร์ Rust เวอร์ชัน nightly โดยในไฟล์ `Cargo.toml` ให้กำหนดค่าดีเพนเดนซีของ Proptest ดังนี้:

```toml
[dev-dependencies.proptest]
version = "proptestVersion"

# Opt out of the `std` feature
default-features = false

# alloc: Use the `alloc` crate directly. Proptest has a hard requirement on
# memory allocation, so either this or `std` is needed.
# unstable: Enable use of nightly-only compiler features.
features = ["no_std", "alloc", "unstable"]
```

API บางตัวจะไม่สามารถใช้งานได้ในบิลด์แบบ `no_std` ซึ่งรวมถึงฟังก์ชันการทำงานที่จำเป็นต้องพึ่งพา `std` โดยตรง เช่น การบันทึกกรณีล้มเหลวไว้ทดสอบซ้ำ (failure persistence) และการฟอร์กโปรเซส (forking) ตลอดจนฟีเจอร์ที่ขึ้นกับเครตภายนอกซึ่งไม่รองรับ `no_std` เช่น การสร้างสตริงจากเรกูลาร์เอ็กซ์เพรสชัน (regex)

นอกจากนี้ บิลด์แบบ `no_std` อาจไม่สามารถเข้าถึงแหล่งความสุ่ม (entropy source) ได้ (ยกเว้นบนสถาปัตยกรรม x86-64 ที่รองรับคำสั่ง `rdrand` ซึ่งสามารถคอมไพล์ไลบรารีพร้อมเปิดฟีเจอร์ `hardware-rng` เพื่อดึงเลขสุ่มจากฮาร์ดแวร์ได้) หากไม่มีแหล่งความสุ่มให้ใช้งาน ตัวรันทดสอบ `TestRunner` ทุกตัว (กล่าวคือ ทุกฟังก์ชัน `#[test]` ที่ครอบด้วยมาโคร `proptest!`) จะใช้ซีดตั้งต้นค่าคงที่ (hard-coded seed) เพียงค่าเดียวเสมอ สำหรับอินพุตที่มีความซับซ้อน จึงควรเพิ่มจำนวนเคสทดสอบ (test cases) ให้มากขึ้นเพื่อชดเชย ทั้งนี้ ค่าซีดแบบฮาร์ดโค้ดดังกล่าวไม่ได้มีการการันตีว่าจะคงเดิมตลอดไป และอาจเปลี่ยนแปลงได้ระหว่างเวอร์ชันของ Proptest โดยไม่ต้องแจ้งให้ทราบล่วงหน้า
