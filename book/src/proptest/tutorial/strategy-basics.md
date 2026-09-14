# พื้นฐานของกลยุทธ์

โปรดอ่าน[บทนำของบทช่วยสอนนี้](index.md)ก่อนเริ่มหัวข้อนี้

[_Strategy_](https://docs.rs/proptest/latest/proptest/strategy/trait.Strategy.html) เป็นแนวคิดพื้นฐานที่สุดใน proptest กลยุทธ์นิยามสองสิ่ง:

- วิธีสร้างค่าสุ่มของชนิดข้อมูลหนึ่งๆ จากตัวสร้างเลขสุ่ม

- วิธี "ชริงก์" ค่าเหล่านั้นให้อยู่ในรูปแบบที่ "ง่ายกว่า"

Proptest มาพร้อมไลบรารีกลยุทธ์จำนวนมาก บางส่วนนิยามจากชนิดข้อมูลที่มีมาให้ในภาษา ตัวอย่างเช่น `0..100i32` เป็นกลยุทธ์สำหรับสร้าง `i32` ตั้งแต่ 0 (รวม) ไปจนถึง 100 (ไม่รวม) ดังที่เราเห็นมาแล้วว่าสตริงเองก็เป็นกลยุทธ์สำหรับสร้างสตริงที่ตรงกับสตริงก่อนหน้าในฐานะนิพจน์ปรกติ

การสร้างค่าเป็นกระบวนการสองขั้นตอน ขั้นแรก `TestRunner` จะถูกส่งให้เมธอด `new_tree()` ของ `Strategy` ซึ่งจะคืนค่า `ValueTree` ที่เราจะดูรายละเอียดกันอีกสักครู่ การเรียกเมธอด `current()` บน `ValueTree` จะได้ค่าจริงออกมา เมื่อรู้อย่างนี้แล้ว เราก็ประกอบชิ้นส่วนเข้าด้วยกันแล้วสร้างค่าออกมาได้ ด้านล่างคือตัวอย่าง `tutorial-strategy-play.rs`:

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

ถ้าคุณรันสักสองสามครั้ง จะได้เอาต์พุตคล้ายๆ แบบนี้

```text
$ target/debug/examples/tutorial-strategy-play
int_val = 99, str_val = vѨͿἕΌ
$ target/debug/examples/tutorial-strategy-play
int_val = 25, str_val = cwᵸійΉ
$ target/debug/examples/tutorial-strategy-play
int_val = 5, str_val = oegiᴫᵸӈᵸὛΉ
```

ความรู้เพียงเท่านี้ก็เพียงพอจะสร้างการทดสอบแบบฟัซขั้นพื้นฐานที่สุดได้

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

วิธีนี้_ได้ผล_ แต่เมื่อเทสต์ล้มเหลว เราแทบไม่ได้บริบทอะไรเลย และต่อให้เรากู้ค่าอินพุตกลับมาได้ ก็จะเห็นค่าที่ดูเหมือนสุ่มไปหมดอย่าง 1771 แทนที่จะเป็นเงื่อนไขขอบเขตอย่าง 501 สำหรับฟังก์ชันที่รับแค่จำนวนเต็มตัวเดียว วิธีนี้อาจยังพอใช้ได้ แต่เมื่ออินพุตซับซ้อนขึ้น การตีความค่าที่สุ่มแบบล้วนๆ ก็จะยิ่งยากขึ้นเรื่อยๆ
