# การฟอร์กและไทม์เอาต์

โดยค่าเริ่มต้น เทสต์ของ proptest จะรันภายในโปรเซสเดียวกัน และได้รับอนุญาตให้รันนานเท่าที่ต้องการ วิธีนี้ประหยัดทรัพยากรและให้เอาต์พุตการทดสอบที่อ่านง่ายที่สุด และเพียงพอสำหรับการใช้งานหลายกรณี อย่างไรก็ตาม ปัญหาอย่างสแตกโอเวอร์โฟลว์ การทำให้โปรเซสยุติการทำงานทันที หรือการติดอยู่ในลูปไม่รู้จบ จะทำให้ทั้งโปรเซสการทดสอบพังไปเลย และทำให้ proptest ไม่สามารถหากรณีที่เล็กที่สุดที่ทำซ้ำได้

ตั้งแต่เวอร์ชัน 0.7.1 เป็นต้นมา proptest มีฟีเจอร์ "fork" และ "timeout" แบบเลือกเปิดใช้ได้ (เปิดใช้งานทั้งคู่โดยค่าเริ่มต้น) ซึ่งทำให้สามารถรันกรณีทดสอบในโปรเซสย่อยและจำกัดระยะเวลาที่อนุญาตให้รันได้ วิธีนี้โดยทั่วไปจะช้ากว่า อาจทำให้การใช้ดีบักเกอร์ยากขึ้น และทำให้เอาต์พุตการทดสอบตีความได้ยากขึ้น แต่ก็ช่วยให้ proptest ค้นหาและย่อกรณีทดสอบสำหรับสถานการณ์เหล่านี้ได้เช่นกัน

หากต้องการใช้ฟีเจอร์เหล่านี้ เพียงตั้งค่าฟิลด์ `fork` และ/หรือ `timeout` บน `Config` (การตั้งค่า `timeout` ย่อมหมายถึงการเปิด `fork` ด้วย)

นี่คือตัวอย่างง่ายๆ ของการใช้ทั้งสองฟีเจอร์:

```rust,should_panic
# extern crate proptest;
use proptest::prelude::*;

// The worst possible way to calculate Fibonacci numbers
fn fib(n: u64) -> u64 {
    if n <= 1 {
        n
    } else {
        fib(n - 1) + fib(n - 2)
    }
}

proptest! {
    #![proptest_config(ProptestConfig {
        // Setting both fork and timeout is redundant since timeout implies
        // fork, but both are shown for clarity.
        fork: true,
        timeout: 100,
        # cases: 1, // Need to set this to 1 to avoid doctest running forever
        .. ProptestConfig::default()
    })]
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn test_fib(n: u64) {
        // For large n, this will variously run for an extremely long time,
        // overflow the stack, or panic due to integer overflow.
        assert!(fib(n) >= n);
    }
}
# fn main() { test_fib(); }
```

ค่าที่ทำให้เทสต์ล้มเหลวอย่างแน่ชัดนั้นแตกต่างกันไปตามประสิทธิภาพของระบบโฮสต์ เวอร์ชันของ Rust และแฟล็กของคอมไพเลอร์ แต่บนระบบที่ใช้ทดสอบครั้งแรก พบว่าค่าสูงสุดที่ `fib()` รับไหวคือ 39 แม้ระหว่างทางจะมีโปรเซสหลายสิบตัวที่ต้องดัมป์คอร์เพราะสแตกโอเวอร์โฟลว์หรือหมดเวลาไปตามทาง

ถ้าคุณแค่อยากรันเทสต์ในโปรเซสย่อยหรือรันแบบมีไทม์เอาต์เป็นครั้งคราว ก็ทำได้โดยตั้งค่าตัวแปรสภาพแวดล้อม `PROPTEST_FORK` หรือ `PROPTEST_TIMEOUT` เพื่อเปลี่ยนการตั้งค่าเริ่มต้น ตัวอย่างเช่น บน Unix

```sh
# Run all the proptest tests in subprocesses with no timeout.
# Individual tests can still opt out by setting `fork: false` in their
# own configuration.
PROPTEST_FORK=true cargo test
# Run all the proptest tests in subprocesses with a 1 second timeout.
# Tests can still opt out or use a different timeout by setting `timeout: 0`
# or another timeout in their own configuration.
PROPTEST_TIMEOUT=1000 cargo test
```
