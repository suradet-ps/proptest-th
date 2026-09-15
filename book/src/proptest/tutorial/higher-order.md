# กลยุทธ์อันดับสูง

_กลยุทธ์อันดับสูง (Higher-order strategy)_ คือกลยุทธ์ที่ถูกสร้างขึ้นมาจากกลยุทธ์อีกตัวหนึ่ง ฟังดูอาจจะเหมือนซับซ้อนน่ากลัว งั้นเราลองมาดูตัวอย่างเพื่อให้เห็นภาพกันก่อน

สมมติคุณมีฟังก์ชันที่ต้องการทดสอบ ซึ่งรับสไลซ์และดัชนีสำหรับเข้าถึงข้อมูลในสไลซ์นั้น หากเรากำหนดขนาดของสไลซ์ไว้คงที่ก็คงไม่มีปัญหาอะไร แต่ในความเป็นจริงเรามักต้องการทดสอบกับสไลซ์ที่มีความยาวหลากหลายขนาด เราอาจลองใช้ตัวกรองดูแบบนี้:

```rust
# extern crate proptest;
use proptest::prelude::*;
fn some_function(stuff: &[String], index: usize) { /* do stuff */ }

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_some_function(
        stuff in prop::collection::vec(".*", 1..100),
        index in 0..100usize
    ) {
        prop_assume!(index < stuff.len());
        some_function(&stuff, index);
    }
}
```

วิธีนี้ทำงานได้ไม่ดีนัก: อย่างแรกคือ คุณจะเจอกรณีการปฏิเสธแบบโกลบอล (global rejections) มากเกินไป เพราะค่า `index` มีโอกาสหลุดออกนอกขอบเขตของ `stuff` ถึง 50% และอย่างที่สองคือ มันแทบจะไม่มีโอกาสได้ทดสอบกับเวกเตอร์ `stuff` ที่มีขนาดสั้นๆ เลย เพราะการจะเกิดขึ้นได้ ระบบต้องสุ่มเลือก `index` ที่มีค่าน้อยมากๆ มาพร้อมกันในรอบเดียวกันเท่านั้น

ทางออกสำหรับปัญหานี้คือคอมบิเนเตอร์ `prop_flat_map` ซึ่งทำงานคล้ายกับ `prop_map` แต่แตกต่างกันตรงที่ฟังก์ชันแปลงค่าจะส่งคืน _กลยุทธ์ (strategy)_ ตัวใหม่กลับมาแทนที่จะเป็นค่าเดี่ยวๆ เพื่อให้เข้าใจได้ง่ายขึ้น ลองดูโค้ดตัวอย่างของเรา:

```rust
# extern crate proptest;
use proptest::prelude::*;

fn some_function(stuff: Vec<String>, index: usize) {
    let _ = &stuff[index];
    // Do stuff
}

fn vec_and_index() -> impl Strategy<Value = (Vec<String>, usize)> {
    prop::collection::vec(".*", 1..100)
        .prop_flat_map(|vec| {
            let len = vec.len();
            (Just(vec), 0..len)
        })
}

proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_some_function((vec, index) in vec_and_index()) {
        some_function(vec, index);
    }
}
# fn main() { test_some_function(); }
```

ในฟังก์ชัน `vec_and_index()` เราเริ่มต้นด้วยการสร้างกลยุทธ์สุ่มเวกเตอร์ค่าใดๆ (arbitrary vector) ขึ้นมาก่อน จากนั้นเราสร้างกลยุทธ์ชุดใหม่ขึ้นมาโดยอิงจาก _ค่า_ ที่กลยุทธ์แรกผลิตออกมา โดยกลยุทธ์ใหม่นี้จะส่งต่อเวกเตอร์เดิมออกไปโดยไม่เปลี่ยนแปลง พร้อมทั้งสร้างดัชนี (index) ที่ถูกต้องสัมพันธ์กับเวกเตอร์นั้นออกมาควบคู่กัน ซึ่งทำได้โดยการกำหนดช่วงของดัชนีตามขนาดความยาวจริงของเวกเตอร์

แม้ว่ากลยุทธ์ใหม่จะใช้ `Just(vec)` เพื่อส่งต่อเวกเตอร์ออกไป Proptest ก็ยังคงเข้าใจความสัมพันธ์ย้อนกลับไปยังกลยุทธ์ตั้งต้น และสามารถย่อขนาด (shrink) ทั้งตัวเวกเตอร์ `vec` และดัชนี `index` ให้สัมพันธ์กันได้อย่างถูกต้อง โดยที่ `index` จะยังคงเป็นดัชนีที่ถูกต้องของ `vec` เสมอตลอดกระบวนการชริงก์

นอกจากนี้ มาโคร `prop_compose!` ยังรองรับการสร้างกลยุทธ์อันดับสอง (second-order strategies) ในลักษณะนี้ได้ง่ายๆ เพียงแค่ระบุรายการพารามิเตอร์ 3 ชุดแทนที่จะเป็น 2 ชุด ซึ่งจะคลี่โค้ดออกมาเหมือนกับสิ่งที่เราเขียนด้วยมือด้านบน (มีเพียงการสลับตำแหน่งของ index และ vec เล็กน้อยภายในเนื่องจากข้อจำกัดเรื่องการยืมตัวแปรของ Rust):

```rust
# extern crate proptest;
# use proptest::prelude::*;
prop_compose! {
    fn vec_and_index()(vec in prop::collection::vec(".*", 1..100))
                    (index in 0..vec.len(), vec in Just(vec))
                    -> (Vec<String>, usize) {
       (vec, index)
   }
}
```
