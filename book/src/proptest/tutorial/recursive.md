# การสร้างข้อมูลแบบเรียกซ้ำ

การสุ่มสร้างโครงสร้างข้อมูลแบบเรียกซ้ำ (Recursive Data Structures) เช่น โครงสร้างต้นไม้ (Trees) หรือไวยากรณ์ AST มีความซับซ้อนกว่าที่คิด ตัวอย่างด้านล่างคือความพยายามเบื้องต้นในการสร้างโครงสร้าง JSON AST โดยการเรียกฟังก์ชันซ้ำตรงๆ:

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

เมื่อพิจารณาดูให้ดี จะพบว่าโค้ดด้านบนไม่สามารถทำงานได้เลย เพราะฟังก์ชัน `arb_json()` มีการเรียกซ้ำตัวเองแบบไม่มีเงื่อนไขหยุด (infinite recursion)

อีกวิธีหนึ่งที่ดูรัดกุมขึ้น คือการนิยามกลยุทธ์แยกตามแต่ละระดับความลึกของการซ้อนกันจนถึงขีดจำกัดสูงสุด แม้วิธีนี้จะไม่ทำให้เกิด stack overflow แต่การเติบโตของโครงสร้างข้อมูลจะทวีคูณอย่างรวดเร็วมาก โดยเพียงแค่ความลึก 4 ระดับก็อาจสร้างต้นไม้ที่มีโหนดมากถึง _หลายพัน_ โหนดแล้ว และเมื่อลึกถึง 8 ระดับ จำนวนโหนดอาจพุ่งสูงขึ้นเป็นหลัก _สิบล้าน_ โหนด

Proptest ได้เตรียมทางออกที่รัดกุมและเชื่อถือได้กว่า ผ่านคอมบิเนเตอร์ `prop_recursive` โดยเราจะเริ่มต้นด้วยการสร้างกลยุทธ์สำหรับกรณีฐานที่ไม่เรียกซ้ำ (leaf nodes / non-recursive cases) จากนั้นส่งกลยุทธ์ฐานดังกล่าว พร้อมระบุพารามิเตอร์ควบคุมขนาด และฟังก์ชันโคลเชอร์ที่ใช้สร้างกิ่งก้านสาขา (recursive branches) ให้กับคอมบิเนเตอร์:

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
