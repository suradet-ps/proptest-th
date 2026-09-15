# การใช้ตัวรันทดสอบ

แทนที่เราจะต้องลงมือเขียนลูปย่อขนาดเคสทดสอบเองด้วยมือ โครงสร้าง [`TestRunner`](https://docs.rs/proptest/latest/proptest/test_runner/struct.TestRunner.html) ของ Proptest มีฟังก์ชันการทำงานนี้มาให้พร้อมใช้งาน และยังช่วยดักจับข้อผิดพลาดอย่างการแพนิก (panics) ให้อีกด้วย เมธอดสำคัญที่เราสนใจคือ `run` ซึ่งเราเพียงแค่ส่งกลยุทธ์ (strategy) และฟังก์ชันสำหรับทดสอบอินพุตเข้าไป แล้ว `TestRunner` จะดูแลขั้นตอนการสุ่มและการย่อขนาดที่เหลือให้ทั้งหมดโดยอัตโนมัติ:

```rust
# extern crate proptest;
use proptest::test_runner::{Config, FileFailurePersistence,
                            TestError, TestRunner};

fn some_function(v: i32) {
    // Do a bunch of stuff, but crash if v > 500.
    // We return to normal `assert!` here since `TestRunner` catches
    // panics.
    assert!(v <= 500);
}

// We know the function is broken, so use a purpose-built main function to
// find the breaking point.
fn main() {
    let mut runner = TestRunner::new(Config {
        // Turn failure persistence off for demonstration
        failure_persistence: Some(Box::new(FileFailurePersistence::Off)),
        .. Config::default()
    });
    let result = runner.run(&(0..10000i32), |v| {
        some_function(v);
        Ok(())
    });
    match result {
        Err(TestError::Fail(_, value)) => {
            println!("Found minimal failing case: {}", value);
            assert_eq!(501, value);
        },
        result => panic!("Unexpected result: {:?}", result),
    }
}
```

แบบนี้ดีขึ้นกว่าเดิมมาก! แม้ว่าจะยังคงมีโค้ดโครงสร้างที่ต้องเขียนซ้ำๆ (boilerplate) อยู่บ้าง ซึ่งมาโคร `proptest!` จะเข้ามาช่วยลดความซ้ำซ้อนตรงนี้ลง แต่เนื่องจากมาโครดังกล่าวยังมีความสามารถอื่นๆ ที่เรายังไม่ได้พูดถึง ในตอนนี้เราจึงจะใช้ `TestRunner` โดยตรงไปก่อน
