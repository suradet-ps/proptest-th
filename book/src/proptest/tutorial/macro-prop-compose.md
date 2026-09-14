# ซินแทกซ์ชูการ์: `prop_compose!`

การนิยามฟังก์ชันที่คืนค่ากลยุทธ์แบบนี้มีประโยชน์มาก แต่โค้ดข้างบนยาวไปหน่อย แถมยังอ่านยากด้วยเหตุผลคล้ายๆ กับการเขียนฟังก์ชันทดสอบด้วยมือ

เพื่อให้งานนี้ง่ายขึ้น proptest จึงมีมาโคร [`prop_compose!`](https://docs.rs/proptest/latest/proptest/macro.prop_compose.html) ก่อนจะลงรายละเอียด นี่คือโค้ดจากข้างบนที่เขียนใหม่โดยใช้มัน

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
prop_compose! {
    fn arb_order_id()(id in any::<u32>()) -> String {
        id.to_string()
    }
}
prop_compose! {
    fn arb_order(max_quantity: u32)
                (id in arb_order_id(), item in "[a-z]*",
                 quantity in 1..max_quantity)
                -> Order {
        Order { id, item, quantity }
    }
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

เราต้องแยก `arb_order_id()` ออกมาเป็นฟังก์ชันของตัวเอง แต่ผลลัพธ์จากการลดรูปซินแทกซ์ชูการ์ก็เกือบจะเหมือนกับสิ่งที่เราเขียนไว้ในหัวข้อก่อนหน้าเลย ฟังก์ชันที่ถูกสร้างขึ้นจะรับรายการพารามิเตอร์ชุดแรกเป็นอาร์กิวเมนต์ อาร์กิวเมนต์เหล่านี้ใช้เลือกกลยุทธ์ในรายการอาร์กิวเมนต์ชุดที่สอง จากนั้นค่าจะถูกดึงออกมาจากกลยุทธ์เหล่านั้นและแปลงด้วยเนื้อความของฟังก์ชัน ฟังก์ชันจริงมีชนิดข้อมูลที่คืนค่าเป็น `impl Strategy<Value = T>` โดยที่ `T` คือชนิดข้อมูลที่ประกาศไว้
