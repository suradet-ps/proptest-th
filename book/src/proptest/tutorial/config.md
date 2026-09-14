# การตั้งค่าจำนวนกรณีทดสอบที่ต้องการ

จำนวนเริ่มต้นของกรณีทดสอบที่สำเร็จซึ่งต้องรันเพื่อให้เทสต์ทั้งเทสต์ผ่าน ปัจจุบันคือ 256 หากคุณไม่พอใจกับค่านี้และต้องการรันมากกว่าหรือน้อยกว่า มีหลายวิธีที่ทำได้

วิธีแรกคือตั้งค่าตัวแปรสภาพแวดล้อม `PROPTEST_CASES` เป็นค่าที่แยกวิเคราะห์เป็น `u32` ได้สำเร็จ ค่าที่คุณตั้งให้ตัวแปรนี้จะกลายเป็นค่าเริ่มต้นใหม่ (วิธีนี้ใช้ได้เฉพาะเมื่อเปิดฟีเจอร์ `std` ของ proptest ซึ่งเปิดอยู่โดยค่าเริ่มต้น)

อีกวิธีหนึ่งคือใช้ `#![proptest_config(expr)]` ภายใน `proptest!` โดยที่ `expr : Config` หากต้องการเปลี่ยนแค่จำนวนกรณีทดสอบ ก็เขียนได้ง่ายๆ ว่า:

```rust
# extern crate proptest;
use proptest::prelude::*;

fn add(a: i32, b: i32) -> i32 { a + b }

proptest! {
    // The next line modifies the number of tests.
    #![proptest_config(ProptestConfig::with_cases(1000))]
    #[test]
    # fn dummy(a in 0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_add(a in 0..1000i32, b in 0..1000i32) {
        let sum = add(a, b);
        assert!(sum >= a);
        assert!(sum >= b);
    }
}
# fn main() {
#     test_add();
# }
```

ผ่านกลไก `proptest_config` เดียวกันนี้ คุณยังปรับแต่งการตั้งค่าได้อย่างละเอียดผ่านชนิดข้อมูล `Config` ดูข้อมูลเพิ่มเติมได้ที่เอกสารประกอบของมัน
