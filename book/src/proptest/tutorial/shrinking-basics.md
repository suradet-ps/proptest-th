# พื้นฐานการชริงก์

กระบวนการค้นหาอินพุตที่ "เรียบง่ายที่สุด" ซึ่งยังคงทำให้การทดสอบล้มเหลว เรียกว่า _การย่อขนาด (Shrinking)_ นี่คือจุดที่ชนิดข้อมูลตัวกลางอย่าง `ValueTree` เข้ามามีบทบาทสำคัญ เพราะนอกจากเมธอด `current()` แล้ว มันยังมีอีก 2 เมธอดหลักคือ `simplify()` และ `complicate()` ซึ่งเมื่อทำงานร่วมกันจะทำหน้าที่คล้ายการค้นหาแบบทวิภาค (Binary Search) บนขอบเขตอินพุต ตัวอย่างโค้ดใน `tutorial-simplify-play.rs` แสดงให้เห็นว่า การเรียก `simplify()` ซ้ำๆ จะทำให้ได้ผลลัพธ์ที่ "เรียบง่ายขึ้น" ทีละขั้น ทั้งในแง่ของขนาดความยาวและช่วงตัวอักษรที่ใช้:

```rust
# extern crate proptest;
use proptest::test_runner::TestRunner;
use proptest::strategy::{Strategy, ValueTree};

fn main() {
    let mut runner = TestRunner::default();
    let mut str_val = "[a-z]{1,4}\\p{Cyrillic}{1,4}\\p{Greek}{1,4}"
        .new_tree(&mut runner).unwrap();
    println!("str_val = {}", str_val.current());
    while str_val.simplify() {
        println!("        = {}", str_val.current());
    }
}
```

ลองรันสักสองครั้ง:

```text
$ target/debug/examples/tutorial-simplify-play
str_val = vy꙲ꙈᴫѱΆῨῨ
        = y꙲ꙈᴫѱΆῨῨ
        = y꙲ꙈᴫѱΆῨῨ
        = m꙲ꙈᴫѱΆῨῨ
        = g꙲ꙈᴫѱΆῨῨ
        = d꙲ꙈᴫѱΆῨῨ
        = b꙲ꙈᴫѱΆῨῨ
        = a꙲ꙈᴫѱΆῨῨ
        = aꙈᴫѱΆῨῨ
        = aᴫѱΆῨῨ
        = aѱΆῨῨ
        = aѱΆῨῨ
        = aѱΆῨῨ
        = aиΆῨῨ
        = aМΆῨῨ
        = aЎΆῨῨ
        = aЇΆῨῨ
        = aЃΆῨῨ
        = aЁΆῨῨ
        = aЀΆῨῨ
        = aЀῨῨ
        = aЀῨ
        = aЀῨ
        = aЀῢ
        = aЀ῟
        = aЀ῞
        = aЀ῝
$ target/debug/examples/tutorial-simplify-play
str_val = dyiꙭᾪῇΊ
        = yiꙭᾪῇΊ
        = iꙭᾪῇΊ
        = iꙭᾪῇΊ
        = iꙭᾪῇΊ
        = eꙭᾪῇΊ
        = cꙭᾪῇΊ
        = bꙭᾪῇΊ
        = aꙭᾪῇΊ
        = aꙖᾪῇΊ
        = aꙋᾪῇΊ
        = aꙅᾪῇΊ
        = aꙂᾪῇΊ
        = aꙁᾪῇΊ
        = aꙀᾪῇΊ
        = aꙀῇΊ
        = aꙀΊ
        = aꙀΊ
        = aꙀΊ
        = aꙀΉ
        = aꙀΈ
```

ข้อควรทราบคือ กระบวนการชริงก์จะไม่ย่อขนาดค่าให้หลุดออกไปนอกขอบเขตที่กลยุทธ์กำหนดไว้โดยเด็ดขาด สังเกตได้ว่าสตริงในตัวอย่างข้างต้นจะยังคงสอดคล้องกับเรกูลาร์เอ็กซ์เพรสชันจนถึงขั้นตอนสุดท้าย หรือหากสุ่มตัวเลขจำนวนเต็มจากช่วง `100..1000i32` ระบบจะพยายามย่อขนาดเข้าหาศูนย์ แต่จะหยุดอยู่ที่ 100 เสมอเนื่องจากเป็นค่าต่ำสุดของช่วงที่กำหนด

เราสามารถนำ `simplify()` และ `complicate()` มาประยุกต์ใช้เพื่อปรับปรุงการทดสอบแบบฟัซซิงของเรา ให้สามารถค้นพบค่าขอบเขต (boundary condition) ที่แท้จริงได้อย่างแม่นยำ:

```rust
# extern crate proptest;
use proptest::test_runner::TestRunner;
use proptest::strategy::{Strategy, ValueTree};

fn some_function(v: i32) -> bool {
    // Do a bunch of stuff, but crash if v > 500
    // assert!(v <= 500);
    // But return a boolean instead of panicking for simplicity
    v <= 500
}

// We know the function is broken, so use a purpose-built main function to
// find the breaking point.
fn main() {
    let mut runner = TestRunner::default();
    for _ in 0..256 {
        let mut val = (0..10000i32).new_tree(&mut runner).unwrap();
        if some_function(val.current()) {
            // Test case passed
            continue;
        }

        // We found our failing test case, simplify it as much as possible.
        loop {
            if !some_function(val.current()) {
                // Still failing, find a simpler case
                if !val.simplify() {
                    // No more simplification possible; we're done
                    break;
                }
            } else {
                // Passed this input, back up a bit
                if !val.complicate() {
                    break;
                }
            }
        }

        println!("The minimal failing case is {}", val.current());
        assert_eq!(501, val.current());
        return;
    }
    panic!("Didn't find a failing test case");
}
```

โค้ดชุดนี้จะสามารถค้นพบค่าขอบเขตต่ำสุดที่ทำให้เกิดข้อผิดพลาด คือ 501 ได้อย่างแม่นยำและสม่ำเสมอ
