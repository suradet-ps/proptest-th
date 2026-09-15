# ซินแทกซ์ชูการ์: `prop_compose!`

การนิยามฟังก์ชันที่คืนค่ากลยุทธ์ (strategy-returning functions) ในลักษณะนี้มีประโยชน์อย่างยิ่ง แต่โค้ดในตัวอย่างก่อนหน้านี้ค่อนข้างยืดยาวและอ่านยาก ด้วยเหตุผลเดียวกับการเขียนฟังก์ชันทดสอบด้วยมือ

เพื่อช่วยให้การสร้างกลยุทธ์ทำได้ง่ายและกระชับขึ้น Proptest จึงเตรียมมาโคร [`prop_compose!`](https://docs.rs/proptest/latest/proptest/macro.prop_compose.html) เอาไว้ให้ ก่อนจะไปดูรายละเอียด มาลองดูโค้ดตัวอย่างเดิมที่ถูกเขียนใหม่ด้วยมาโครนี้กันก่อน:

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

แม้เราจะต้องแยก `arb_order_id()` ออกมาเป็นอีกฟังก์ชันหนึ่ง แต่กระบวนการคลี่ไวยากรณ์ (desugaring) จะได้โค้ดที่เกือบจะเหมือนกับสิ่งที่เราลงมือเขียนเองในหัวข้อที่แล้วทุกประการ: ฟังก์ชันที่สร้างขึ้นจะรับพารามิเตอร์ชุดแรกเป็นอาร์กิวเมนต์ ซึ่งจะถูกนำไปใช้ส่งต่อให้กลยุทธ์ในพารามิเตอร์ชุดที่สอง จากนั้นระบบจะสุ่มสร้างค่าจากกลยุทธ์เหล่านั้นแล้วแปลงด้วยตรรกะในบล็อกของฟังก์ชัน โดยฟังก์ชันที่ได้จะมีชนิดข้อมูลรีเทิร์นเป็น `impl Strategy<Value = T>` ซึ่ง `T` คือชนิดข้อมูลปลายทางที่ระบุไว้
