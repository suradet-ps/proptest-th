# การฟอร์กและไทม์เอาต์

โดยค่าเริ่มต้น เทสต์ของ proptest จะรันอยู่ภายในโปรเซสหลักตัวเดียวกัน (in-process) และได้รับอนุญาตให้รันนานเท่าใดก็ได้ วิธีนี้ประหยัดทรัพยากร ให้เอาต์พุตผลการทดสอบที่สะอาดตาที่สุด และเพียงพอสำหรับการใช้งานทั่วไปส่วนใหญ่ อย่างไรก็ตาม หากเกิดปัญหาขั้นวิกฤต เช่น สแตกโอเวอร์โฟลว์ (stack overflow), โปรเซสถูกสั่งหยุดกะทันหัน (abort) หรือโค้ดติดลูปไม่รู้จบ (infinite loop) ปัญหาเหล่านี้จะทำให้โปรเซสการทดสอบทั้งหมดตายลงทันที ส่งผลให้ Proptest ไม่สามารถดำเนินขั้นตอนการย่อขนาด (shrink) เพื่อค้นหาเคสทดสอบที่เล็กที่สุดมาจำลองปัญหาได้

ตั้งแต่เวอร์ชัน 0.7.1 เป็นต้นมา Proptest ได้เพิ่มฟีเจอร์ทางเลือกอย่าง "fork" (การแยกโปรเซสย่อย) และ "timeout" (การจำกัดเวลาทำงาน) เข้ามา (ซึ่งเปิดใช้งานเป็นค่าเริ่มต้นทั้งคู่) ทำให้สามารถแยกรันแต่ละกรณีทดสอบในโปรเซสย่อย (subprocess) และกำหนดขีดจำกัดเวลาทำงานได้ แม้ว่าวิธีนี้จะทำงานช้ากว่าเดิม ทำให้การใช้ดีบักเกอร์ยุ่งยากขึ้น และอ่านเอาต์พุตได้ยากขึ้นเล็กน้อย แต่ก็ช่วยให้ Proptest สามารถดักจับ ตลอดจนย่อขนาดเคสทดสอบสำหรับสถานการณ์ร้ายแรงเหล่านี้ได้อย่างมีประสิทธิภาพ

หากต้องการเปิดใช้งานฟีเจอร์เหล่านี้ เพียงกำหนดค่าที่ฟิลด์ `fork` และ/หรือ `timeout` บน `Config` (การกำหนดค่า `timeout` จะถือว่าเปิดใช้ `fork` ไปด้วยโดยอัตโนมัติ)

นี่คือตัวอย่างง่ายๆ ของการใช้งานทั้งสองฟีเจอร์ร่วมกัน:

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

ค่าอินพุตที่ทำให้เทสต์ล้มเหลวที่แท้จริงนั้นจะขึ้นอยู่กับประสิทธิภาพของเครื่องที่รัน เวอร์ชันของ Rust และแฟล็กของคอมไพเลอร์เป็นสำคัญ แต่บนระบบที่ผู้พัฒนาใช้ทดสอบต้นฉบับ พบว่าค่าสูงสุดที่ฟังก์ชัน `fib()` สามารถคำนวณไหวคือ 39 แม้ว่าระหว่างขั้นตอนการชริงก์จะมีโปรเซสย่อยนับสิบโปรเซสเกิด core dump จากสแตกโอเวอร์โฟลว์หรือหมดเวลา (timeout) ไปก็ตาม

หากต้องการรันการทดสอบในโปรเซสย่อยหรือกำหนดไทม์เอาต์เป็นครั้งคราวโดยไม่ต้องแก้โค้ด คุณสามารถตั้งค่าผ่านตัวแปรสภาพแวดล้อม (environment variables) อย่าง `PROPTEST_FORK` หรือ `PROPTEST_TIMEOUT` เพื่อแทนที่ค่าคอนฟิกเริ่มต้นได้ ตัวอย่างเช่น บนระบบ Unix:

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
