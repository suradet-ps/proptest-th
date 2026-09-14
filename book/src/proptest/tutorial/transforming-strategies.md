# การแปลงกลยุทธ์

สมมติว่าคุณมีฟังก์ชันที่รับสตริงซึ่งต้องเป็นรูปแบบ `Display` ของ `u32` แบบตามอำเภอใจ วิธีแรกที่อาจลองใช้ในการหาอาร์กิวเมนต์นี้คือการใช้นิพจน์ปรกติ ดังนี้:

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

วิธีนี้พอใช้ได้ แต่มีปัญหาอยู่ อย่างแรก มันไม่ได้สำรวจปริภูมิ `u32` ทั้งหมด เราสามารถเขียนนิพจน์ปรกติที่ทำได้ แต่นิพจน์แบบนั้นยาวพอสมควร แถมยังทำให้การกระจายของค่าดูแปลกๆ อีกด้วย อินพุตยังชริงก์ไม่ถูกต้องอีกด้วย เนื่องจาก proptest พยายามชริงก์มันในฐานะสตริง ไม่ใช่จำนวนเต็ม

สิ่งที่คุณต้องการจริงๆ คือสร้าง `u32` ขึ้นมา แล้วส่งตัวแทนในรูปสตริงของมันเข้าไป วิธีหนึ่งคือรับ `u32` เป็นอินพุตของเทสต์ แล้วแปลงเป็นสตริงภายในโค้ดเทสต์ วิธีนี้ใช้ได้ดี แต่ไม่สามารถนำกลับมาใช้ซ้ำหรือประกอบกับอย่างอื่นได้ ในอุดมคติ เราอยากได้_กลยุทธ์_ที่ทำสิ่งนี้ได้

สิ่งที่เราตามหาคือ_คอมบิเนเตอร์_ของกลยุทธ์ตัวแรกอย่าง `prop_map` เราต้องทำให้แน่ใจว่า `Strategy` อยู่ในสโคปจึงจะใช้มันได้

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

การเรียก `prop_map` บน `Strategy` จะสร้างกลยุทธ์ใหม่ที่แปลงทุกค่าที่สร้างขึ้นด้วยฟังก์ชันที่ให้มา Proptest ยังคงรักษาความสัมพันธ์ระหว่าง `Strategy` ต้นทางกับตัวที่ถูกแปลงไว้ ผลก็คือการชริงก์เกิดขึ้นในระดับ `u32` แม้ว่าเราจะสร้าง `String` ก็ตาม

`prop_map` ยังเป็นวิธีหลักในการนิยามกลยุทธ์สำหรับชนิดข้อมูลใหม่ เนื่องจากชนิดข้อมูลส่วนใหญ่เกิดจากการประกอบค่าอื่นที่ง่ายกว่าเข้าด้วยกัน

มาปรับโค้ดของเราให้รับโครงสร้างที่น่าสนใจมากขึ้นดีกว่า


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

จะเห็นว่าเราสามารถนำเอาต์พุตจาก `prop_map` ไปใส่ในทูเพิล แล้วเรียก `prop_map` บนทูเพิล_นั้น_ เพื่อสร้างค่าอีกตัวหนึ่งได้

แต่นั่นก็ยาวเฟื้อยในรายการอาร์กิวเมนต์ โชคดีที่กลยุทธ์เป็นค่าแบบปกติ เราจึงแยกมันออกมาเป็นฟังก์ชันได้

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

เราเรียก `boxed()` กับกลยุทธ์ในฟังก์ชัน เพราะไม่เช่นนั้นชนิดข้อมูลจะไม่สามารถตั้งชื่อได้ และต่อให้ตั้งชื่อได้ ก็จะอ่านหรือเขียนยากมาก การบ็อกซ์ `Strategy` จะเปลี่ยนทั้งตัวมันและ `ValueTree` ของมันให้เป็นเทรตออบเจกต์ ซึ่งทำให้ชนิดข้อมูลง่ายขึ้นและยังใช้ผสมกลยุทธ์ชนิดต่างๆ ที่ต่างกันได้ ตราบใดที่มันสร้างค่าชนิดเดียวกัน

ฟังก์ชัน `arb_order()` ยังเป็น_พารามิเตอร์ไรซ์_ด้วย ซึ่งเป็นข้อดีอีกอย่างของการแยกกลยุทธ์ออกมาเป็นฟังก์ชันต่างหาก ในกรณีนี้ ถ้าเรามีเทสต์ที่ต้องการ `Order` ที่มีไม่เกินสิบสองชิ้น เราก็เรียก `arb_order(12)` ได้เลย โดยไม่ต้องเขียนกลยุทธ์ใหม่ทั้งชุด

เรายังใช้ `-> impl Strategy<Value = Order>` แทนเพื่อหลีกเลี่ยงค่าใช้จ่ายส่วนเกินได้ ดังตัวอย่างต่อไปนี้ คุณควรใช้ `-> impl Strategy<..>` เว้นแต่คุณต้องการการดิสแพตช์แบบไดนามิก

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
