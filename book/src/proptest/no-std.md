# การรองรับ `no_std`

Proptest รองรับการใช้งานในบริบท `no_std` แบบบางส่วน

คุณจะต้องใช้คอมไพเลอร์เวอร์ชัน nightly ใน `Cargo.toml` ของคุณ ให้ปรับดีเพนเดนซีของ Proptest ให้มีหน้าตาประมาณนี้:

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

API บางตัวไม่พร้อมใช้งานในบิลด์แบบ `no_std` ซึ่งรวมถึงฟังก์ชันการทำงานที่จำเป็นต้องใช้ `std` อย่างเช่นการคงอยู่ของกรณีล้มเหลวและการฟอร์ก รวมถึงฟีเจอร์ที่พึ่งพาครีตอื่นซึ่งไม่รองรับการใช้งานแบบ `no_std` เช่นการรองรับ regex

บิลด์แบบ `no_std` อาจไม่สามารถเข้าถึงแหล่งเอนโทรปีได้ (ข้อยกเว้นหนึ่งคือเครื่อง x86-64 ที่รองรับ rdrand ในกรณีนี้สามารถคอมไพล์ไลบรารีด้วยฟีเจอร์ `hardware-rng` เพื่อให้ได้ตัวเลขสุ่ม) หากไม่มีแหล่งเอนโทรปี ทุก `TestRunner` (นั่นคือทุก `#[test]` เมื่อใช้มาโคร `proptest!`) จะใช้ซีดแบบฮาร์ดโค้ดเพียงค่าเดียว สำหรับอินพุตที่ซับซ้อน อาจเป็นความคิดที่ดีที่จะเพิ่มจำนวนกรณีทดสอบเพื่อชดเชย ทั้งนี้ ซีดแบบฮาร์ดโค้ดไม่ได้รับประกันตามสัญญา และอาจเปลี่ยนแปลงได้ระหว่างเวอร์ชัน Proptest โดยไม่แจ้งให้ทราบล่วงหน้า
