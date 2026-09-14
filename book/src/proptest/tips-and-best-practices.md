# เคล็ดลับและแนวปฏิบัติที่ดีที่สุด

## ประสิทธิภาพ

### การตั้งค่า `opt-level`
ทั้งครีต proptest และตัวสร้างเลขสุ่มที่มันใช้อยู่อาจกิน CPU มาก หากคุณสร้างเคสจำนวนมาก คุณอาจเห็นประสิทธิภาพดีขึ้นอย่างชัดเจนโดยตั้งค่า `opt-level` เป็น `3` ในไฟล์ `Cargo.toml` ของคุณ:

```toml
[profile.test.package.proptest]
opt-level = 3

[profile.test.package.rand_chacha]
opt-level = 3
```

### การนำทรัพยากรที่แก้ไขได้กลับมาใช้ซ้ำ
บางครั้งคุณอาจต้องการนำทรัพยากรที่แก้ไขได้กลับมาใช้ซ้ำระหว่างแต่ละกรณี ตัวอย่างเช่น คุณอาจต้องการใช้การเชื่อมต่อฐานข้อมูลหรือ file handle ซ้ำ เพื่อหลีกเลี่ยงค่าใช้จ่ายในการเปิดและปิดสำหรับทุกกรณี เนื่องจากมาโคร `proptest!` (เมื่อใช้กับการเรียกแบบโคลเชอร์) ต้องการ `Fn` คุณจึงต้องห่อสถานะของคุณไว้ใน `RefCell`:

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
