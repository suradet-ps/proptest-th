# การใช้ตัวรันทดสอบ

แทนที่จะชริงก์เองด้วยมือ [`TestRunner`](https://docs.rs/proptest/latest/proptest/test_runner/struct.TestRunner.html) ของ proptest มีฟังก์ชันนี้ให้เราใช้ และยังจัดการเรื่องอย่างการแพนิกให้ด้วย เมธอดที่เราสนใจคือ `run` เราเพียงส่งกลยุทธ์กับฟังก์ชันสำหรับทดสอบอินพุตให้มัน แล้วมันจะจัดการส่วนที่เหลือให้เอง

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

ดีขึ้นเยอะเลย! แต่ก็ยังมีโค้ดซ้ำซากอยู่บ้าง มาโคร `proptest!` จะช่วยเรื่องนี้ได้ แต่มันยังทำอย่างอื่นที่เรายังไม่ได้พูดถึงอีก ดังนั้นตอนนี้เราจะใช้ `TestRunner` ตรงๆ กันไปก่อน
