# การสร้าง enum

ในปัจจุบัน ซินแทกซ์ชูการ์สำหรับช่วยนิยามกลยุทธ์ให้กับ `enum` ยังค่อนข้างมีจำกัด แม้ว่าจะสามารถเขียนสร้างกลยุทธ์ประเภทนี้ด้วย `prop_compose!` ได้ แต่มักจะอ่านยากและไม่ค่อยสวยงาม ในกรณีส่วนใหญ่ การเขียนนิยามฟังก์ชันขึ้นมาด้วยตนเองจึงเป็นทางเลือกที่เหมาะสมกว่า

หัวใจสำคัญในการสร้างค่าให้กับ enum คือมาโคร [`prop_oneof!`](https://docs.rs/proptest/latest/proptest/macro.prop_oneof.html) ซึ่งเปิดโอกาสให้คุณระบุกลยุทธ์สำหรับแต่ละวาเรียนต์ (variant) ของ `enum`: สำหรับวาเรียนต์ที่ไม่มีข้อมูลแนบ (unit variant) กลยุทธ์ที่ใช้จะเป็นเพียง `Just(YourEnum::TheCase)` ส่วนวาเรียนต์ที่มีข้อมูลแนบมาด้วย โดยทั่วไปจะต้องสร้างข้อมูลนั้นไว้ในทูเพิล แล้วใช้ `prop_map` เพื่อนำไปแปลงเป็นวาเรียนต์ของ enum นั้นๆ

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

ตามแนวปฏิบัติที่ดีที่สุด ควรจัดเรียงลำดับวาเรียนต์ของ enum จาก "เรียบง่ายที่สุด" ไปหา "ซับซ้อนที่สุด" เสมอ เพราะกระบวนการย่อขนาด (shrinking) จะพยายามย่อค่ากลับเข้าหาวาเรียนต์ที่อยู่ลำดับต้นๆ ในรายการ

สำหรับวาเรียนต์ที่มีฟิลด์ซับซ้อนเป็นพิเศษ การแยกกลยุทธ์สำหรับสร้างวาเรียนต์นั้นออกมาเป็นฟังก์ชันต่างหากจะช่วยให้อ่านโค้ดได้ง่ายขึ้น ซึ่งในจุดนี้ มาโคร [`prop_compose!`](https://docs.rs/proptest/latest/proptest/macro.prop_compose.html) จะมีประโยชน์อย่างยิ่ง:

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
