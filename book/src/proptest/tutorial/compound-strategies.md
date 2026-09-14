# กลยุทธ์เชิงประกอบ

การทดสอบฟังก์ชันที่รับอาร์กิวเมนต์เดียวชนิดพื้นฐานก็ดีไปอย่าง แต่ก็ค่อนข้างธรรมดาไปหน่อย ตอนที่เราเขียนทุกอย่างด้วยมือ การขยายเทคนิคไปใช้กับจำนวนเต็ม _สองตัว_ ก็ตรงไปตรงมา ถึงจะยาวหน่อย แต่ `TestRunner` รับ `Strategy` ได้เพียงตัวเดียว แล้วเราจะทดสอบฟังก์ชันที่ต้องการอินพุตมากกว่าหนึ่งตัวได้อย่างไร?

```rust,ignore
# extern crate proptest;
use proptest::test_runner::TestRunner;

fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[test]
# fn dummy() {} // Doctests don't build `#[test]` functions, so we need this
fn test_add() {
    let mut runner = TestRunner::default();
    runner.run(/* uhhm... */).unwrap();
}
# fn main() { test_add(); }
```

หัวใจคือกลยุทธ์นั้น_ประกอบเข้าด้วยกันได้_ รูปแบบการประกอบที่ง่ายที่สุดคือ "กลยุทธ์เชิงประกอบ" ซึ่งเรานำกลยุทธ์หลายตัวมารวมค่าของมันเข้าเป็นค่าเดียวที่เก็บอินพุตแต่ละตัวแยกกัน กลยุทธ์แบบนี้มีอยู่หลายตัว ที่ง่ายที่สุดคือทูเพิล ทูเพิลของกลยุทธ์ก็เป็นกลยุทธ์สำหรับทูเพิลของค่าที่กลยุทธ์เหล่านั้นสร้างขึ้น ตัวอย่างเช่น `(0..100i32,100..1000i32)` เป็นกลยุทธ์สำหรับคู่จำนวนเต็มที่ค่าแรกอยู่ระหว่าง 0 กับ 100 และค่าที่สองอยู่ระหว่าง 100 กับ 1000

ดังนั้นสำหรับฟังก์ชันสองอาร์กิวเมนต์ของเรา กลยุทธ์ก็คือทูเพิลของช่วงตัวเลข

```rust
# extern crate proptest;
use proptest::test_runner::TestRunner;

fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[test]
# fn dummy() {} // Doctests don't build `#[test]` functions, so we need this
fn test_add() {
    let mut runner = TestRunner::default();
    // Combine our two inputs into a strategy for one tuple. Our test
    // function then destructures the generated tuples back into separate
    // `a` and `b` variables to be passed in to `add()`.
    runner.run(&(0..1000i32, 0..1000i32), |(a, b)| {
        let sum = add(a, b);
        assert!(sum >= a);
        assert!(sum >= b);
        Ok(())
    }).unwrap();
}
# fn main() { test_add(); }
```

กลยุทธ์เชิงประกอบแบบอื่นๆ ได้แก่ อาเรย์ขนาดคงที่ของกลยุทธ์และ `Vec` ของกลยุทธ์ (ซึ่งสร้างอาเรย์หรือ `Vec` ของค่าที่ขนานกับคอลเลกชันกลยุทธ์) รวมถึงกลยุทธ์ต่างๆ ที่โมดูล [collection](https://docs.rs/proptest/latest/proptest/collection/index.html) มีให้
