# เคล็ดลับและแนวปฏิบัติที่ดีที่สุด

## ประสิทธิภาพ

### การตั้งค่า `opt-level`
ทั้งเครต `proptest` และตัวกำเนิดตัวเลขสุ่มที่ใช้นั้นอาจใช้พลังการประมวลผลของ CPU ค่อนข้างสูง หากคุณกำลังทดสอบด้วยการสุ่มสร้างกรณีทดสอบจำนวนมาก คุณอาจเห็นประสิทธิภาพและความเร็วเพิ่มขึ้นอย่างเห็นได้ชัด ด้วยการตั้งค่า `opt-level` เป็น `3` สำหรับแพ็กเกจเหล่านี้ในไฟล์ `Cargo.toml`:

```toml
[profile.test.package.proptest]
opt-level = 3

[profile.test.package.rand_chacha]
opt-level = 3
```

### การนำทรัพยากรที่เปลี่ยนแปลงค่าได้กลับมาใช้ซ้ำ
ในบางสถานการณ์ คุณอาจต้องการนำทรัพยากรที่เปลี่ยนแปลงค่าได้ (mutable resources) กลับมาใช้ซ้ำระหว่างการทดสอบแต่ละเคส เช่น การใช้ connection ของฐานข้อมูลเดิม หรือ file handle เดิมซ้ำ เพื่อลดโอเวอร์เฮด (overhead) จากการเปิดและปิดใหม่ทุกครั้งในแต่ละกรณีทดสอบ และเนื่องจากมาโคร `proptest!` (เมื่อเรียกใช้ในรูปแบบโคลเชอร์) กำหนดว่าฟังก์ชันทดสอบต้องเป็นเทรต `Fn` คุณจึงจำเป็นต้องห่อสถานะดังกล่าวไว้ภายใน `RefCell`:

```rust
# extern crate proptest;
use std::cell::RefCell;
use proptest::proptest;

# struct ConnectionPool {};
# struct MyConnection {};
# impl ConnectionPool {
#    fn new() -> Self { Self {} }
#    fn connect(&mut self) -> MyConnection { MyConnection {} }
# }
#[test]
# fn dummy() {}; // This is here to make the doctest work
fn test_with_shared_connection() {
    let mut my_conn = RefCell::new(ConnectionPool::new().connect());
    proptest!(|(x in 0..42)| {
        let mut conn = my_conn.borrow_mut();
        // Use state
    });
}
```
