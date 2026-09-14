# การสร้าง enum

ปัจจุบันซินแทกซ์ชูการ์สำหรับนิยามกลยุทธ์ให้ `enum` ยังค่อนข้างจำกัด เราสามารถสร้างกลยุทธ์แบบนี้ด้วย `prop_compose!` ได้ แต่โดยทั่วไปแล้วไม่ค่อยอ่านง่าย ดังนั้นในกรณีส่วนใหญ่ การนิยามฟังก์ชันด้วยมือจึงดีกว่า

ส่วนประกอบหลักคือมาโคร [`prop_oneof!`](https://docs.rs/proptest/latest/proptest/macro.prop_oneof.html) ซึ่งคุณจะระบุหนึ่งกรณีสำหรับแต่ละกรณีใน `enum` ของคุณ สำหรับ `enum` ที่ไม่มีข้อมูล กลยุทธ์ของแต่ละกรณีคือ `Just(YourEnum::TheCase)` ส่วนกรณีของ enum ที่มีข้อมูล โดยทั่วไปต้องนำข้อมูลไปใส่ในทูเพิล แล้วใช้ `prop_map` แมปมันเข้าสู่กรณีของ enum

นี่คือตัวอย่างง่ายๆ:

```rust
# extern crate proptest;
use proptest::prelude::*;

#[derive(Debug, Clone)]
enum MyEnum {
    SimpleCase,
    CaseWithSingleDatum(u32),
    CaseWithMultipleData(u32, String),
}

fn my_enum_strategy() -> impl Strategy<Value = MyEnum> {
  prop_oneof![
    // For cases without data, `Just` is all you need
    Just(MyEnum::SimpleCase),

    // For cases with data, write a strategy for the interior data, then
    // map into the actual enum case.
    any::<u32>().prop_map(MyEnum::CaseWithSingleDatum),

    (any::<u32>(), ".*").prop_map(
      |(a, b)| MyEnum::CaseWithMultipleData(a, b)),
  ]
}
```

โดยทั่วไป ควรเรียงกรณีของ enum จาก "ง่ายที่สุด" ไปจนถึง "ซับซ้อนที่สุด" เนื่องจากการชริงก์จะย่อเข้าหารายการที่อยู่ต้นๆ ของรายการ

สำหรับกรณีของ enum ที่ซับซ้อนเป็นพิเศษ การแยกกลยุทธ์ของกรณีนั้นออกมาเป็นกลยุทธ์ต่างหากอาจช่วยได้ ในกรณีนี้ [`prop_compose!`](https://docs.rs/proptest/latest/proptest/macro.prop_compose.html) มีประโยชน์

```rust
# extern crate proptest;
use proptest::prelude::*;

#[derive(Debug, Clone)]
enum MyComplexEnum {
    SimpleCase,
    AnotherSimpleCase,
    ComplexCase {
        product_code: String,
        id: u64,
        chapter: String,
    },
}

prop_compose! {
  fn my_complex_enum_complex_case()(
      product_code in "[0-9A-Z]{10,20}",
      id in 1u64..10000u64,
      chapter in "X{0,2}(V?I{1,3}|IV|IX)",
  ) -> MyComplexEnum {
      MyComplexEnum::ComplexCase { product_code, id, chapter }
  }
}

fn my_enum_strategy() -> BoxedStrategy<MyComplexEnum> {
  prop_oneof![
    Just(MyComplexEnum::SimpleCase),
    Just(MyComplexEnum::AnotherSimpleCase),
    my_complex_enum_complex_case(),
  ].boxed()
}
```
