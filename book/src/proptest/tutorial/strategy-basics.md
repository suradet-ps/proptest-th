# พื้นฐานของกลยุทธ์

โปรดตรวจสอบให้แน่ใจว่าได้อ่าน[บทนำของบทเรียนนี้](index.md)ก่อนเริ่มหัวข้อนี้

[_Strategy_](https://docs.rs/proptest/latest/proptest/strategy/trait.Strategy.html) เป็นแนวคิดพื้นฐานที่สุดใน Proptest โดยกลยุทธ์ (Strategy) ทำหน้าที่กำหนดสิ่งสำคัญ 2 ประการ:

- วิธีกำเนิดค่าสุ่มของชนิดข้อมูลหนึ่งๆ จากตัวสร้างเลขสุ่ม (Random Number Generator)
- วิธี "ย่อขนาด (shrink)" ค่านั้นลงให้อยู่ในรูปแบบที่ "เรียบง่ายขึ้น"

Proptest มาพร้อมคลังกลยุทธ์สำเร็จรูปจำนวนมาก ซึ่งบางส่วนถูกนิยามไว้สำหรับชนิดข้อมูลพื้นฐานของภาษา Rust อยู่แล้ว เช่น `0..100i32` เป็นกลยุทธ์สำหรับสร้างค่า `i32` ตั้งแต่ 0 (รวม) ไปจนถึง 100 (ไม่รวม) และดังที่เราได้เห็นไปแล้ว สตริงก็สามารถทำหน้าที่เป็นกลยุทธ์ในตัวเองสำหรับการสุ่มสตริงที่ตรงตามแพตเทิร์นเรกูลาร์เอ็กซ์เพรสชันนั้นๆ ได้เช่นกัน

การสร้างค่าประกอบด้วย 2 ขั้นตอนหลัก: ขั้นแรก ส่ง `TestRunner` ให้กับเมธอด `new_tree()` ของ `Strategy` ซึ่งจะได้ `ValueTree` กลับมา (เราจะมาเจาะลึกกันในหัวข้อถัดไป) จากนั้นเรียกเมธอด `current()` บน `ValueTree` เพื่อดึงค่าที่ถูกสร้างขึ้นมาใช้งานจริง เมื่อเข้าใจกลไกนี้แล้ว เราสามารถนำชิ้นส่วนต่างๆ มาประกอบกันเพื่อสร้างค่าได้ ดังตัวอย่างใน `tutorial-strategy-play.rs`:

```rust
# extern crate proptest;
use proptest::test_runner::TestRunner;
use proptest::strategy::{Strategy, ValueTree};

fn main() {
    let mut runner = TestRunner::default();
    let int_val = (0..100i32).new_tree(&mut runner).unwrap();
    let str_val = "[a-z]{1,4}\\p{Cyrillic}{1,4}\\p{Greek}{1,4}"
        .new_tree(&mut runner).unwrap();
    println!("int_val = {}, str_val = {}",
             int_val.current(), str_val.current());
}
```

หากคุณลองสั่งรันโค้ดนี้สักสองสามครั้ง จะได้เอาต์พุตหน้าตาคล้ายๆ กันดังนี้:

```text
$ target/debug/examples/tutorial-strategy-play
int_val = 99, str_val = vѨͿἕΌ
$ target/debug/examples/tutorial-strategy-play
int_val = 25, str_val = cwᵸійΉ
$ target/debug/examples/tutorial-strategy-play
int_val = 5, str_val = oegiᴫᵸӈᵸὛΉ
```

ความรู้เพียงเท่านี้ก็เพียงพอแล้วสำหรับการสร้างการทดสอบแบบฟัซซิง (primitive fuzzing test) ขั้นพื้นฐาน:

```rust
# extern crate proptest;
use proptest::test_runner::TestRunner;
use proptest::strategy::{Strategy, ValueTree};

fn some_function(v: i32) {
    // Do a bunch of stuff, but crash if v > 500
    assert!(v <= 500);
}

#[test]
fn some_function_doesnt_crash() {
    let mut runner = TestRunner::default();
    for _ in 0..256 {
        let val = (0..10000i32).new_tree(&mut runner).unwrap();
        some_function(val.current());
    }
}
```

วิธีนี้ _ใช้งานได้จริง_ แต่เมื่อเทสต์ล้มเหลว เราแทบจะไม่ได้ข้อมูลบริบทอะไรเลย และต่อให้เรากู้ค่าอินพุตกลับมาดูได้ เราก็จะเห็นค่าที่ดูเหมือนสุ่มมาลอยๆ อย่าง 1771 แทนที่จะเป็นค่าขอบเขตที่ทำให้เกิดปัญหาจริงๆ เช่น 501 สำหรับฟังก์ชันที่รับแค่ตัวเลขจำนวนเต็มตัวเดียว การได้ค่าแบบนี้อาจยังพอเดาสาเหตุได้บ้าง แต่เมื่อโครงสร้างของอินพุตมีความซับซ้อนยิ่งขึ้น การตีความค่าที่ถูกสุ่มมาแบบกระจัดกระจายก็แทบจะเป็นไปไม่ได้เลย
