# กลยุทธ์เชิงประกอบ

การทดสอบฟังก์ชันที่รับอาร์กิวเมนต์ชนิดพื้นฐานเพียงตัวเดียวอาจดูสะดวกดี แต่ก็ยังไม่ตอบโจทย์การใช้งานจริงส่วนใหญ่ ย้อนกลับไปตอนที่เราเขียนโครงสร้างการทดสอบด้วยมือ การขยายไปทดสอบอาร์กิวเมนต์ตัวเลข _สองตัว_ นั้นอาจพอเขียนตรงๆ ได้แม้จะค่อนข้างยาว แต่เมธอด `TestRunner::run` รับ `Strategy` ได้เพียงตัวเดียวเท่านั้น แล้วเราจะทดสอบฟังก์ชันที่ต้องการอินพุตมากกว่าหนึ่งค่าพร้อมกันได้อย่างไร?

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

กุญแจสำคัญคือ กลยุทธ์ใน Proptest นั้นสามารถ _นำมาประกอบกันได้ (Composable)_ รูปแบบการประกอบที่เรียบง่ายที่สุดคือ "กลยุทธ์เชิงประกอบ (Compound Strategies)" ซึ่งเป็นการรวมหลายๆ กลยุทธ์เข้าด้วยกัน เพื่อให้สร้างค่าผลลัพธ์เป็นโครงสร้างเดียวที่บรรจุแต่ละอินพุตแยกจากกัน ซึ่งมีอยู่หลายรูปแบบ โดยรูปแบบที่ง่ายที่สุดคือ ทูเพิล (Tuple): ทูเพิลของกลยุทธ์จะทำหน้าที่เป็นกลยุทธ์สำหรับสร้างทูเพิลของค่าที่กลยุทธ์ย่อยเหล่านั้นผลิตออกมา เช่น `(0..100i32, 100..1000i32)` คือกลยุทธ์สำหรับสร้างคู่ตัวเลขจำนวนเต็ม โดยค่าแรกอยู่ในช่วง 0 ถึง 100 และค่าที่สองอยู่ในช่วง 100 ถึง 1000

ดังนั้น สำหรับฟังก์ชันที่รับ 2 อาร์กิวเมนต์ของเรา กลยุทธ์ที่ใช้จึงเป็นเพียงทูเพิลของช่วงตัวเลขเท่านั้น:

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

นอกจากนี้ กลยุทธ์เชิงประกอบรูปแบบอื่นๆ ยังรวมถึง อาร์เรย์ของกลยุทธ์ขนาดคงที่ (fixed-size arrays), เวกเตอร์ `Vec` ของกลยุทธ์ (ซึ่งจะสร้างอาร์เรย์หรือ `Vec` ของค่าออกมาตามลำดับ) ตลอดจนกลยุทธ์สร้างคอลเลกชันรูปแบบต่างๆ ที่มีให้ใช้งานในโมดูล [collection](https://docs.rs/proptest/latest/proptest/collection/index.html)
