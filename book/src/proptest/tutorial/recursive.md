# การสร้างข้อมูลแบบเรียกซ้ำ

การสร้างโครงสร้างข้อมูลแบบเรียกซ้ำขึ้นมาแบบสุ่มนั้นยากกว่าที่คิด ตัวอย่างเช่น โค้ดด้านล่างเป็นความพยายามแบบไร้เดียงสาในการสร้าง JSON AST ด้วยการเรียกซ้ำ

```rust
# extern crate proptest;
use std::collections::HashMap;
use proptest::prelude::*;

#[derive(Clone, Debug)]
enum Json {
    Null,
    Bool(bool),
    Number(f64),
    String(String),
    Array(Vec<Json>),
    Map(HashMap<String, Json>),
}

fn arb_json() -> impl Strategy<Value = Json> {
    prop_oneof![
        Just(Json::Null),
        any::<bool>().prop_map(Json::Bool),
        any::<f64>().prop_map(Json::Number),
        ".*".prop_map(Json::String),
        prop::collection::vec(arb_json(), 0..10).prop_map(Json::Array),
        prop::collection::hash_map(
          ".*", arb_json(), 0..10).prop_map(Json::Map),
    ].boxed()
}
```

เมื่อพิจารณาดูดีๆ แล้ว เห็นได้ชัดว่าวิธีนี้ใช้ไม่ได้ เพราะ `arb_json()` เรียกซ้ำตัวเองแบบไม่มีเงื่อนไขหยุด

วิธีที่ซับซ้อนกว่านั้นคือนิยามกลยุทธ์หนึ่งตัวสำหรับแต่ละระดับของการซ้อนกันจนถึงระดับสูงสุดที่กำหนด วิธีนี้ไม่ทำให้สแตกโอเวอร์โฟลว์ แต่ตามที่นิยามไว้ตรงนี้ แค่การซ้อนสี่ระดับก็จะสร้างต้นไม้ที่มีโหนด _หลายพัน_ โหนดแล้ว และพอถึงแปดระดับ เราก็จะได้ถึงหลักสิบ_ล้าน_

Proptest มีทางออกที่เชื่อถือได้กว่าในรูปของคอมบิเนเตอร์ `prop_recursive` หากต้องการใช้ เราจะสร้างกลยุทธ์สำหรับกรณีที่ไม่เรียกซ้ำก่อน แล้วส่งกลยุทธ์นั้นพร้อมพารามิเตอร์ขนาดบางตัวและฟังก์ชันที่แปลงกลยุทธ์ที่ซ้อนกันให้เป็นกลยุทธ์แบบเรียกซ้ำให้กับคอมบิเนเตอร์

```rust
# extern crate proptest;
use std::collections::HashMap;
use proptest::prelude::*;

#[derive(Clone, Debug)]
enum Json {
    Null,
    Bool(bool),
    Number(f64),
    String(String),
    Array(Vec<Json>),
    Map(HashMap<String, Json>),
}

fn arb_json() -> impl Strategy<Value = Json> {
    let leaf = prop_oneof![
        Just(Json::Null),
        any::<bool>().prop_map(Json::Bool),
        any::<f64>().prop_map(Json::Number),
        ".*".prop_map(Json::String),
    ];
    leaf.prop_recursive(
      8, // 8 levels deep
      256, // Shoot for maximum size of 256 nodes
      10, // We put up to 10 items per collection
      |inner| prop_oneof![
          // Take the inner strategy and make the two recursive cases.
          prop::collection::vec(inner.clone(), 0..10)
              .prop_map(Json::Array),
          prop::collection::hash_map(".*", inner, 0..10)
              .prop_map(Json::Map),
      ])
}
```
