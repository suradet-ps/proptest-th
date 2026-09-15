# การแปลงกลยุทธ์

สมมติว่าคุณมีฟังก์ชันที่รับสตริงซึ่งต้องเป็นข้อความแสดงผล (Display format) ของตัวเลข `u32` ค่าใดๆ (arbitrary u32) วิธีแรกที่คุณอาจลองทำคือการใช้นิพจน์เรกูลาร์เอ็กซ์เพรสชัน ดังนี้:

```rust
# extern crate proptest;
use proptest::prelude::*;

fn do_stuff(v: String) {
    let i: u32 = v.parse().unwrap();
    let s = i.to_string();
    assert_eq!(s, v);
}

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_do_stuff(v in "[1-9][0-9]{0,8}") {
        do_stuff(v);
    }
}
# fn main() { test_do_stuff(); }
```

วิธีนี้พอจะทำงานได้ แต่ก็มีข้อบกพร่องอยู่หลายจุด อย่างแรกคือ มันไม่ได้ครอบคลุมขอบเขตทั้งหมดของ `u32` (input space) แม้ว่าในทางทฤษฎีเราจะสามารถเขียนเรกูลาร์เอ็กซ์เพรสชันให้ครอบคลุมได้ แต่นิพจน์นั้นจะยาวเหยียดและซับซ้อนมาก ซ้ำยังส่งผลให้การแจกแจงความน่าจะเป็น (distribution) ของค่าสุ่มดูแปลกประหลาด และที่สำคัญที่สุดคือ อินพุตจะไม่สามารถย่อขนาด (shrink) ได้อย่างถูกต้อง เพราะ Proptest จะพยายามย่อขนาดในฐานะสตริงตัวอักษร แทนที่จะมองเป็นตัวเลขจำนวนเต็ม

สิ่งที่คุณต้องการจริงๆ คือการสร้างค่า `u32` ขึ้นมาก่อน แล้วค่อยแปลงเป็นสตริงส่งเข้าไปทดสอบ วิธีหนึ่งที่ทำได้ง่ายๆ คือรับ `u32` เข้ามาในฟังก์ชันทดสอบ แล้วค่อยเรียกแปลงเป็นสตริงภายในโค้ดเทสต์ วิธีนี้ทำงานได้ดี แต่ไม่สามารถนำกลับมาใช้ซ้ำ (reusable) หรือนำไปประกอบกับกลยุทธ์อื่น (composable) ได้ ในอุดมคติ เราจึงอยากได้ _กลยุทธ์ (strategy)_ ที่ทำหน้าที่แปลงค่านี้ได้โดยตรง

และสิ่งที่เรากำลังมองหาอยู่ก็คือ คอมบิเนเตอร์ของกลยุทธ์ตัวแรก: `prop_map` โดยเราต้องนำเทรต `Strategy` เข้ามาในสโคปเพื่อใช้งาน

```rust
# extern crate proptest;
// Grab `Strategy`, shorter namespace prefix, and the macros
use proptest::prelude::*;

fn do_stuff(v: String) {
    let i: u32 = v.parse().unwrap();
    let s = i.to_string();
    assert_eq!(s, v);
}

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_do_stuff(v in any::<u32>().prop_map(|v| v.to_string())) {
        do_stuff(v);
    }
}
# fn main() { test_do_stuff(); }
```

การเรียก `prop_map` บน `Strategy` จะสร้างกลยุทธ์ใหม่ขึ้นมา ซึ่งจะนำฟังก์ชันที่ส่งเข้าไปมาแปลงค่าทุกตัวที่ถูกสุ่มสร้างขึ้นมา และ Proptest จะยังคงรักษาความเชื่อมโยงระหว่าง `Strategy` ต้นทางกับกลยุทธ์ที่แปลงแล้วไว้เสมอ ส่งผลให้กระบวนการย่อขนาด (shrinking) ยังคงเกิดขึ้นตามกฎเกณฑ์ของ `u32` แม้ว่าค่าที่ส่งให้ฟังก์ชันทดสอบจะเป็น `String` ก็ตาม

`prop_map` ยังเป็นหัวใจหลักในการสร้างกลยุทธ์สำหรับชนิดข้อมูลใหม่ๆ (Custom types) เพราะชนิดข้อมูลส่วนใหญ่เกิดจากการประกอบฟิลด์หรือค่าพื้นฐานที่เรียบง่ายกว่าเข้าด้วยกัน

ลองมาปรับโค้ดของเราให้รับโครงสร้างข้อมูลที่ซับซ้อนและน่าสนใจยิ่งขึ้น:

```rust
# extern crate proptest;
use proptest::prelude::*;

#[derive(Clone, Debug)]
struct Order {
  id: String,
  // Some other fields, though the test doesn't do anything with them
  item: String,
  quantity: u32,
}

fn do_stuff(order: Order) {
    let i: u32 = order.id.parse().unwrap();
    let s = i.to_string();
    assert_eq!(s, order.id);
}

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_do_stuff(
        order in
        (any::<u32>().prop_map(|v| v.to_string()),
         "[a-z]*", 1..1000u32).prop_map(
             |(id, item, quantity)| Order { id, item, quantity })
    ) {
        do_stuff(order);
    }
}
# fn main() { test_do_stuff(); }
```

สังเกตว่าเราสามารถนำผลลัพธ์จาก `prop_map` ไปใส่ไว้ในทูเพิล แล้วเรียก `prop_map` ซ้ำบนทูเพิล _นั้น_ เพื่อแปลงเป็นค่าของ struct ปลายทางได้อีกทอดหนึ่ง

แต่การเขียนกลยุทธ์ซ้อนกันยาวๆ ในรายการพารามิเตอร์แบบนี้อาจทำให้อ่านโค้ดยาก โชคดีที่กลยุทธ์ใน Proptest เป็นค่าออบเจกต์ทั่วไป (first-class values) เราจึงสามารถแยกออกมาเขียนเป็นฟังก์ชันต่างหากได้:

```rust
# extern crate proptest;
use proptest::prelude::*;

// snip
#
# #[derive(Clone, Debug)]
# struct Order {
#   id: String,
#   // Some other fields, though the test doesn't do anything with them
#   item: String,
#   quantity: u32,
# }
#
# fn do_stuff(order: Order) {
#     let i: u32 = order.id.parse().unwrap();
#     let s = i.to_string();
#     assert_eq!(s, order.id);
# }
#
fn arb_order(max_quantity: u32) -> BoxedStrategy<Order> {
    (any::<u32>().prop_map(|v| v.to_string()),
     "[a-z]*", 1..max_quantity)
    .prop_map(|(id, item, quantity)| Order { id, item, quantity })
    .boxed()
}

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_do_stuff(order in arb_order(1000)) {
        do_stuff(order);
    }
}
# fn main() { test_do_stuff(); }
```

เราเรียก `.boxed()` ให้กับกลยุทธ์ในฟังก์ชัน เพราะหากไม่ทำเช่นนั้น ชนิดข้อมูลที่รีเทิร์นจะมีความซับซ้อนจนไม่สามารถระบุชื่อชนิดข้อมูล (unnameable type) ได้ หรือต่อให้เขียนได้ก็อ่านและทำความเข้าใจได้ยากมาก การทำ boxing ให้กับ `Strategy` จะแปลงทั้งตัวกลยุทธ์และ `ValueTree` ของมันให้กลายเป็นเทรตออบเจกต์ (trait object: `BoxedStrategy`) ซึ่งนอกจากจะช่วยให้ชนิดข้อมูลเรียบง่ายขึ้นแล้ว ยังช่วยให้สามารถรวมกลยุทธ์ต่างชนิดกัน (heterogeneous strategies) เข้าด้วยกันได้ ตราบใดที่พวกมันผลิตค่าชนิดเดียวกันออกมา

ฟังก์ชัน `arb_order()` ยังสามารถรับพารามิเตอร์ได้อีกด้วย (parameterised) ซึ่งเป็นข้อดีสำคัญอีกประการของการแยกกลยุทธ์ออกมาเป็นฟังก์ชัน ในกรณีนี้ หากเรามีเทสต์ที่ต้องการทดสอบ `Order` ที่มีจำนวนสินค้าไม่เกิน 12 ชิ้น เราก็เพียงแค่เรียก `arb_order(12)` ได้ทันที โดยไม่ต้องเขียนกลยุทธ์ใหม่ทั้งหมดขึ้นมาอีกรอบ

นอกจากนี้ เรายังสามารถใช้ `-> impl Strategy<Value = Order>` แทนเพื่อหลีกเลี่ยง overhead ของการจองหน่วยความจำบนฮีป (heap allocation) ได้ดังตัวอย่างด้านล่าง ซึ่งคุณควรเลือกใช้ `-> impl Strategy<..>` เป็นค่าเริ่มต้นเสมอ เว้นแต่กรณีที่คุณต้องการ dynamic dispatch จริงๆ:

```rust
# extern crate proptest;
use proptest::prelude::*;

// snip
#
# #[derive(Clone, Debug)]
# struct Order {
#   id: String,
#   // Some other fields, though the test doesn't do anything with them
#   item: String,
#   quantity: u32,
# }
#
# fn do_stuff(order: Order) {
#     let i: u32 = order.id.parse().unwrap();
#     let s = i.to_string();
#     assert_eq!(s, order.id);
# }
#
fn arb_order(max_quantity: u32) -> impl Strategy<Value = Order> {
    (any::<u32>().prop_map(|v| v.to_string()),
     "[a-z]*", 1..max_quantity)
    .prop_map(|(id, item, quantity)| Order { id, item, quantity })
}

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_do_stuff(order in arb_order(1000)) {
        do_stuff(order);
    }
}

# fn main() { test_do_stuff(); }
```
