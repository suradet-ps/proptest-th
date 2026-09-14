# พื้นฐานการชริงก์

การค้นหาอินพุตที่ "ง่ายที่สุด" ซึ่งทำให้เทสต์ล้มเหลวเรียกว่า_การชริงก์_ นี่คือจุดที่ชนิดข้อมูลตัวกลางอย่าง `ValueTree` เข้ามามีบทบาท นอกจาก `current()` แล้ว มันยังมีเมธอดอีกสองตัว ได้แก่ `simplify()` และ `complicate()` ซึ่งเมื่อใช้ร่วมกันจะช่วยให้ค้นหาแบบไบนารีในปริภูมิอินพุตได้ ตัวอย่าง `tutorial-simplify-play.rs` แสดงให้เห็นว่าการเรียก `simplify()` ซ้ำๆ ให้เอาต์พุตที่ "ง่ายขึ้น" ทีละน้อยทั้งในแง่ขนาดและจำนวนอักขระที่ใช้อย่างไร

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

โปรดทราบว่าการชริงก์จะไม่ย่อค่าไปนอกช่วงที่กลยุทธ์อธิบายไว้เลย สังเกตว่าสตริงในตัวอย่างข้างต้นยังคงตรงกับนิพจน์ปรกติแม้ในตอนสุดท้าย จำนวนเต็มที่สุ่มมาจาก `100..1000i32` จะชริงก์เข้าหาศูนย์ แต่จะหยุดที่ 100 เนื่องจากเป็นค่าต่ำสุด

`simplify()` และ `complicate()` ใช้ดัดแปลงการทดสอบฟัซพื้นฐานของเราให้ค้นหาเงื่อนไขขอบเขตได้จริง

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

โค้ดนี้จะค้นหาขอบเขตของความล้มเหลวคือ 501 ได้อย่างแน่นอน
