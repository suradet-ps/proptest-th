# กลยุทธ์อันดับสูง

_กลยุทธ์อันดับสูง_ คือกลยุทธ์ที่ถูกสร้างขึ้นโดยกลยุทธ์อีกตัวหนึ่ง ฟังดูน่ากลัวอยู่เหมือนกัน งั้นมาดูตัวอย่างกันก่อน

สมมติคุณมีฟังก์ชันที่ต้องการทดสอบ ซึ่งรับสไลซ์และดัชนีสำหรับเข้าถึงสไลซ์นั้น ถ้าเราใช้ขนาดคงที่สำหรับสไลซ์ก็ง่ายอยู่ แต่บางทีเราอาจต้องทดสอบกับขนาดสไลซ์ที่ต่างกันออกไป เราลองใช้ตัวกรองดูก็ได้:

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

วิธีนี้ใช้ไม่ได้ผลดีนัก อย่างแรก คุณจะเจอการปฏิเสธแบบโกลบอลจำนวนมาก เพราะ `index` จะอยู่นอกช่วงของ `stuff` ถึง 50% ของเวลา อย่างที่สอง มันจะหาเวกเตอร์ `stuff` ขนาดเล็กได้ยากมาก เพราะต้องสุ่มเลือก `index` ที่มีค่าน้อยในเวลาเดียวกันด้วย

ทางออกคือคอมบิเนเตอร์ `prop_flat_map` ซึ่งคล้ายกับ `prop_map` ต่างกันตรงที่ฟังก์ชันแปลงจะคืนค่าเป็น_กลยุทธ์_แทนที่จะเป็นค่า เข้าใจได้ง่ายขึ้นเมื่อลองทำตัวอย่างของเรา:

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

ใน `vec_and_index()` เราสร้างกลยุทธ์สำหรับสร้างเวกเตอร์แบบตามอำเภอใจ แต่แล้วเราก็สร้างกลยุทธ์ใหม่ขึ้นมาจาก_ค่า_ที่กลยุทธ์แรกสร้างขึ้น กลยุทธ์ใหม่จะสร้างเวกเตอร์ที่ได้มาโดยไม่เปลี่ยนแปลง แต่เพิ่มดัชนีที่ถูกต้องสำหรับเวกเตอร์นั้นเข้าไปด้วย ซึ่งทำได้โดยเลือกกลยุทธ์ของดัชนีนั้นตามขนาดของเวกเตอร์

ถึงแม้กลยุทธ์ใหม่จะระบุกลยุทธ์ `Just(vec)` แบบค่าเดียวสำหรับเวกเตอร์ proptest ก็ยังเข้าใจความเชื่อมโยงกับกลยุทธ์ต้นทาง และจะชริงก์ `vec` ด้วยเช่นกัน ตลอดเวลานั้น `index` ยังคงเป็นดัชนีที่ถูกต้องของ `vec` อยู่เสมอ

จริงๆ แล้ว `prop_compose!` ช่วยให้สร้างกลยุทธ์อันดับสองแบบนี้ได้ เพียงแค่ระบุรายการอาร์กิวเมนต์สามชุดแทนที่จะเป็นสองชุด โค้ดด้านล่างลดรูปซินแทกซ์ชูการ์ออกมาเป็นอะไรที่คล้ายกับที่เราเขียนด้วยมือข้างบนมาก ยกเว้นว่าตำแหน่งของ index กับ vector ถูกสลับกันภายในเนื่องจากข้อจำกัดด้านการยืม

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
