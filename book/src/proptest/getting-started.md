# เริ่มต้นใช้งาน

สมมติว่าเราต้องการเขียนฟังก์ชันสำหรับแยกวิเคราะห์วันที่ในรูปแบบ `YYYY-MM-DD` โดยที่เราจะไม่สนใจการ _ตรวจสอบความถูกต้อง_ ของวันที่มากนัก ขอให้เป็นจำนวนเต็มสามตัวอะไรก็ได้ งั้นเรามาเขียนแบบเร็วๆ กันเลย

```rust
fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
    if 10 != s.len() { return None; }
    if "-" != &s[4..5] || "-" != &s[7..8] { return None; }

    let year = &s[0..4];
    let month = &s[6..7];
    let day = &s[8..10];

    year.parse::<u32>().ok().and_then(
        |y| month.parse::<u32>().ok().and_then(
            |m| day.parse::<u32>().ok().map(
                |d| (y, m, d))))
}
```

คอมไพล์ผ่าน แสดงว่าใช้ได้แล้วใช่ไหม? อาจจะไม่นะ มาเพิ่มเทสต์กันหน่อย

```rust,ignore
# fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
#     if 10 != s.len() { return None; }
#     if "-" != &s[4..5] || "-" != &s[7..8] { return None; }
#
#     let year = &s[0..4];
#     let month = &s[6..7];
#     let day = &s[8..10];
#
#     year.parse::<u32>().ok().and_then(
#         |y| month.parse::<u32>().ok().and_then(
#             |m| day.parse::<u32>().ok().map(
#                 |d| (y, m, d))))
# }
#[test]
# fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
fn test_parse_date() {
    assert_eq!(None, parse_date("2017-06-1"));
    assert_eq!(None, parse_date("2017-06-170"));
    assert_eq!(None, parse_date("2017006-17"));
    assert_eq!(None, parse_date("2017-06017"));
    assert_eq!(Some((2017, 06, 17)), parse_date("2017-06-17"));
}
# fn main() { test_parse_date(); }
```

เทสต์ผ่านแล้ว ขึ้นโปรดักชันได้เลย! แต่แล้วแอปพลิเคชันของคุณก็เริ่มแครช และผู้คนก็ไม่พอใจที่คุณย้ายคริสต์มาสไปเป็นเดือนกุมภาพันธ์ เราคงต้องตรวจสอบให้ละเอียดกว่านี้สักหน่อย

ใน `Cargo.toml` ให้เพิ่ม

```toml
[dev-dependencies]
proptest = "1.11.0"
```

ตอนนี้เราสามารถเพิ่มการทดสอบพร็อพเพอร์ตีให้กับตัวแยกวิเคราะห์วันที่ของเราได้แล้ว แต่จะทดสอบตัวแยกวิเคราะห์วันที่กับอินพุตตามอำเภอใจได้อย่างไร โดยไม่ต้องเขียนตัวแยกวิเคราะห์วันที่อีกตัวขึ้นมาเพื่อตรวจสอบผลลัพธ์? เราไม่จำเป็นต้องทำเช่นนั้น ตราบใดที่เราเลือกอินพุตและพร็อพเพอร์ตีอย่างถูกต้อง แต่ก่อนที่จะถึงเรื่องความถูกต้อง จริงๆ แล้วมีพร็อพเพอร์ตีที่ง่ายกว่านั้นให้ทดสอบก่อน: _ฟังก์ชันไม่ควรแครช_ เริ่มจากตรงนั้นกันเลย

```rust,should_panic
# extern crate proptest;
// Bring the macros and other important things into scope.
use proptest::prelude::*;
# fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
#     if 10 != s.len() { return None; }
#     if "-" != &s[4..5] || "-" != &s[7..8] { return None; }
#
#     let year = &s[0..4];
#     let month = &s[6..7];
#     let day = &s[8..10];
#
#     year.parse::<u32>().ok().and_then(
#         |y| month.parse::<u32>().ok().and_then(
#             |m| day.parse::<u32>().ok().map(
#                 |d| (y, m, d))))
# }
proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn doesnt_crash(s in "\\PC*") {
        parse_date(&s);
    }
}
# fn main() { doesnt_crash(); }
```

โค้ดนี้จะสุ่ม `&String` ขึ้นมาจริงๆ (ยังไม่ต้องสนใจ `\\PC*` ตอนนี้ เดี๋ยวเราค่อยกลับมาพูดถึง — ถ้าคุณรู้คำตอบแล้ว อดใจรออีกนิด) แล้วส่งให้ `parse_date()` จากนั้นก็ทิ้งผลลัพธ์ไป

พอเรารัน จะได้เอาต์พุตหน้าตาน่ากลัวเต็มไปหมด และสุดท้ายลงท้ายด้วย

```text
thread 'main' panicked at 'Test failed: byte index 4 is not a char boundary; it is inside 'ௗ' (bytes 2..5) of `aAௗ0㌀0`; minimal failing input: s = "aAௗ0㌀0"
	successes: 102
	local rejects: 0
	global rejects: 0
'
```

ถ้าเราดูที่ไดเรกทอรีชั้นบนสุดหลังการทดสอบล้มเหลว เราจะเห็นไดเรกทอรีใหม่ชื่อ `proptest-regressions` ซึ่งมีไฟล์บางไฟล์ที่สอดคล้องกับไฟล์ซอร์สที่บรรจุกรณีทดสอบที่ล้มเหลว ไฟล์เหล่านี้คือ[_การคงอยู่ของกรณีล้มเหลว_](https://proptest-rs.github.io/proptest/proptest/failure-persistence.html) สิ่งแรกที่เราควรทำคือเพิ่มไฟล์เหล่านี้เข้าไปในระบบควบคุมเวอร์ชัน

```text
$ git add proptest-regressions
```

สิ่งต่อไปที่เราควรทำคือคัดลอกกรณีที่ล้มเหลวไปเป็นยูนิตเทสต์แบบดั้งเดิม เนื่องจากมันเผยบั๊กที่ไม่เหมือนกับสิ่งที่เราเคยทดสอบมาก่อน

```rust,should_panic
# fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
#     if 10 != s.len() { return None; }
#     if "-" != &s[4..5] || "-" != &s[7..8] { return None; }
#
#     let year = &s[0..4];
#     let month = &s[6..7];
#     let day = &s[8..10];
#
#     year.parse::<u32>().ok().and_then(
#         |y| month.parse::<u32>().ok().and_then(
#             |m| day.parse::<u32>().ok().map(
#                 |d| (y, m, d))))
# }
#[test]
# fn dummy() {} // Doctests don't build `#[test]` functions, so we need this
fn test_unicode_gibberish() {
    assert_eq!(None, parse_date("aAௗ0㌀0"));
}
# fn main() { test_unicode_gibberish(); }
```

เอาละ มาดูกันว่าเกิดอะไรขึ้น... เราลืมเรื่อง UTF-8 ไป! เราไม่สามารถตัดสไลซ์สตริงแบบไร้ความระวังได้ เพราะอาจตัดครึ่งตัวอักษร ในกรณีนี้คือเครื่องหมายวรรณยุกต์ภาษาทมิฬที่วางซ้อนอยู่บนตัวอักษรอื่นในสตริง

เพื่อให้การแก้โค้ดน้อยที่สุด เราจะตรวจสอบว่าสตริงเป็น ASCII แล้วปฏิเสธอะไรก็ตามที่ไม่ใช่

```rust
# use std::ascii::AsciiExt;
#
fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
    if 10 != s.len() { return None; }

    // NEW: Ignore non-ASCII strings so we don't need to deal with Unicode.
    if !s.is_ascii() { return None; }

    if "-" != &s[4..5] || "-" != &s[7..8] { return None; }

    let year = &s[0..4];
    let month = &s[6..7];
    let day = &s[8..10];

    year.parse::<u32>().ok().and_then(
        |y| month.parse::<u32>().ok().and_then(
            |m| day.parse::<u32>().ok().map(
                |d| (y, m, d))))
}
```

ตอนนี้เทสต์ผ่านแล้ว! แต่เรารู้ว่ายังมีปัญหาอื่นอีก เรามาทดสอบพร็อพเพอร์ตีเพิ่มกันดีกว่า

พร็อพเพอร์ตีอีกอย่างที่เราอยากได้จากโค้ดของเราคือ มันต้องแยกวิเคราะห์วันที่ที่ถูกต้องทุกวันได้ เราสามารถเพิ่มเทสต์อีกตัวในบล็อก `proptest!` ได้ดังนี้

```rust
# extern crate proptest;
# use proptest::prelude::*;
# fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
#     if 10 != s.len() { return None; }
#
#     // NEW: Ignore non-ASCII strings so we don't need to deal with Unicode.
#     if !s.is_ascii() { return None; }
#
#     if "-" != &s[4..5] || "-" != &s[7..8] { return None; }
#
#     let year = &s[0..4];
#     let month = &s[6..7];
#     let day = &s[8..10];
#
#     year.parse::<u32>().ok().and_then(
#         |y| month.parse::<u32>().ok().and_then(
#             |m| day.parse::<u32>().ok().map(
#                 |d| (y, m, d))))
# }
proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn parses_all_valid_dates(s in "[0-9]{4}-[0-9]{2}-[0-9]{2}") {
        parse_date(&s).unwrap();
    }
}
# fn main() { parses_all_valid_dates(); }
```

สิ่งที่อยู่ทางขวามือของ `in` จริงๆ แล้วคือ_นิพจน์ปรกติ_ และ `s` จะถูกเลือกจากสตริงที่ตรงกับนิพจน์นั้น ดังนั้นในเทสต์ก่อนหน้า `"\\PC*"` จึงสร้างสตริงตามอำเภอใจที่ประกอบด้วยอักขระที่ไม่ใช่ตัวควบคุมแบบใดก็ได้ ส่วนตอนนี้เราสร้างค่าที่มีรูปแบบ YYYY-MM-DD

เทสต์ใหม่ผ่านแล้ว งั้นไปดูเรื่องอื่นกันต่อ

พร็อพเพอร์ตีสุดท้ายที่เราอยากตรวจสอบคือ วันที่ถูกแยกวิเคราะห์ออกมา_อย่างถูกต้อง_จริงๆ ซึ่งเราทำแบบนี้ด้วยการสร้างสตริงไม่ได้ — เพราะสุดท้ายก็จะกลายเป็นการเขียนตัวแยกวิเคราะห์วันที่ซ้ำอีกตัวในเทสต์! แต่เราจะเริ่มจากผลลัพธ์ที่คาดหวัง สร้างสตริงขึ้นมา แล้วตรวจว่ามันถูกแยกวิเคราะห์กลับมาได้ถูกต้อง

```rust
# extern crate proptest;
# use proptest::prelude::*;
# fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
#     if 10 != s.len() { return None; }
#
#     // NEW: Ignore non-ASCII strings so we don't need to deal with Unicode.
#     if !s.is_ascii() { return None; }
#
#     if "-" != &s[4..5] || "-" != &s[7..8] { return None; }
#
#     let year = &s[0..4];
#     let month = &s[6..7];
#     let day = &s[8..10];
#
#     year.parse::<u32>().ok().and_then(
#         |y| month.parse::<u32>().ok().and_then(
#             |m| day.parse::<u32>().ok().map(
#                 |d| (y, m, d))))
# }
proptest! {
    #[test]
    # fn dummy(0..1) {} // Doctests don't build `#[test]` functions, so we need this
    fn parses_date_back_to_original(y in 0u32..10000,
                                    m in 1u32..13, d in 1u32..32) {
        let (y2, m2, d2) = parse_date(
            &format!("{:04}-{:02}-{:02}", y, m, d)).unwrap();
        // prop_assert_eq! is basically the same as assert_eq!, but doesn't
        // cause a bunch of panic messages to be printed on intermediate
        // test failures. Which one to use is largely a matter of taste.
        prop_assert_eq!((y, m, d), (y2, m2, d2));
    }
}
```

จะเห็นได้ว่านอกจากนิพจน์ปรกติแล้ว เรายังใช้นิพจน์ใดก็ได้ที่เป็น `proptest::strategy::Strategy` ซึ่งในกรณีนี้คือช่วงของจำนวนเต็ม

พอรัน เทสต์ก็ล้มเหลว ครั้งนี้เอาต์พุตไม่ค่อยมีอะไรมาก

```text
thread 'main' panicked at 'Test failed: assertion failed: `(left == right)` (left: `(0, 10, 1)`, right: `(0, 0, 1)`) at examples/dateparser_v2.rs:46; minimal failing input: y = 0, m = 10, d = 1
	successes: 2
	local rejects: 0
	global rejects: 0
', examples/dateparser_v2.rs:33
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

อินพุตที่ล้มเหลวคือ `(y, m, d) = (0, 10, 1)` ซึ่งเป็นค่าที่ค่อนข้างเฉพาะเจาะจง ก่อนจะคิดว่าทำไมถึงทำให้โค้ดพัง มาดูกันก่อนว่า proptest ทำอะไรบ้างกว่าจะได้ค่านี้มา ให้แทรกโค้ดนี้ตอนต้นของฟังก์ชันทดสอบ

```rust
    # let (y, m, d) = (0, 10, 1);
    println!("y = {}, m = {}, d = {}", y, m, d);
```

รันเทสต์อีกครั้ง เราจะได้อะไรแบบนี้

```text
y = 2497, m = 8, d = 27
y = 9641, m = 8, d = 18
y = 7360, m = 12, d = 20
y = 3680, m = 12, d = 20
y = 1840, m = 12, d = 20
y = 920, m = 12, d = 20
y = 460, m = 12, d = 20
y = 230, m = 12, d = 20
y = 115, m = 12, d = 20
y = 57, m = 12, d = 20
y = 28, m = 12, d = 20
y = 14, m = 12, d = 20
y = 7, m = 12, d = 20
y = 3, m = 12, d = 20
y = 1, m = 12, d = 20
y = 0, m = 12, d = 20
y = 0, m = 6, d = 20
y = 0, m = 9, d = 20
y = 0, m = 11, d = 20
y = 0, m = 10, d = 20
y = 0, m = 10, d = 10
y = 0, m = 10, d = 5
y = 0, m = 10, d = 3
y = 0, m = 10, d = 2
y = 0, m = 10, d = 1
```

ข้อความแจ้งการล้มเหลวบอกว่ามีเคสที่สำเร็จสองเคส ซึ่งเราเห็นได้ที่ด้านบนสุด คือ `2497-08-27` และ `9641-08-18` ส่วนเคสถัดไป `7360-12-20` ล้มเหลว วันที่นี้ไม่ได้ดูพิเศษอะไรในทันที แต่โชคดีที่ proptest ย่อมันลงเหลือเคสที่ง่ายกว่ามาก ขั้นแรก มันลดอินพุต `y` ลงเหลือ `0` อย่างรวดเร็วตั้งแต่ต้น และลดอินพุต `d` ลงเหลือค่าต่ำสุดที่อนุญาตคือ `1` ในตอนท้าย ระหว่างนั้น เรากลับเห็นอะไรบางอย่างที่ต่างออกไป: มันลองชริงก์ `12` ลงเป็น `6` แต่สุดท้ายก็ดันกลับขึ้นไปเป็น `10` นั่นเป็นเพราะกรณีทดสอบ `0000-06-20` และ `0000-09-20` นั้น _ผ่าน_

สุดท้าย เราได้วันที่ `0000-10-01` ซึ่งดูเหมือนจะถูกแยกวิเคราะห์เป็น `0000-00-01` อีกครั้งที่กรณีที่ล้มเหลวนี้ถูกเพิ่มเข้าไปในไฟล์การคงอยู่ของกรณีล้มเหลว และเราควรเพิ่มมันเป็นยูนิตเทสต์ของตัวเองด้วย

```text
$ git add proptest-regressions
```

```rust,should_panic
# fn parse_date(s: &str) -> Option<(u32, u32, u32)> {
#     if 10 != s.len() { return None; }
#
#     // NEW: Ignore non-ASCII strings so we don't need to deal with Unicode.
#     if !s.is_ascii() { return None; }
#
#     if "-" != &s[4..5] || "-" != &s[7..8] { return None; }
#
#     let year = &s[0..4];
#     let month = &s[6..7];
#     let day = &s[8..10];
#
#     year.parse::<u32>().ok().and_then(
#         |y| month.parse::<u32>().ok().and_then(
#             |m| day.parse::<u32>().ok().map(
#                 |d| (y, m, d))))
# }
#[test]
# fn dummy() {} // Doctests don't build `#[test]` functions, so we need this
fn test_october_first() {
    assert_eq!(Some((0, 10, 1)), parse_date("0000-10-01"));
}
# fn main() { test_october_first(); }
```

ทีนี้มาหากันว่าโค้ดพังตรงไหน แม้ไม่มีอินพุตระหว่างทาง เราก็พอพูดได้อย่างมั่นใจว่าส่วนของปีและวันไม่ได้เข้ามาเกี่ยวข้อง เพราะทั้งคู่ถูกลดลงเหลือค่าอินพุตต่ำสุดที่อนุญาตแล้ว ส่วนอินพุตเดือน_ไม่_ได้ถูกลดลงเหลือค่าต่ำสุด แต่ถูกลดลงเหลือ `10` นั่นแปลว่าเราสามารถอนุมานได้ว่ามีอะไรบางอย่างพิเศษกับ `10` ที่ไม่เหมือนกับ `9` ในกรณีนี้ "ความพิเศษ" นั้นก็คือการเป็นเลขสองหลัก ในโค้ดของเรา:

```rust,ignore
    let month = &s[6..7];
```

เราพลาดไปหนึ่งตำแหน่ง และต้องใช้ช่วง `5..7` หลังจากแก้แล้ว เทสต์ก็ผ่าน

มาโคร `proptest!` ยังมีไวยากรณ์เพิ่มเติมอีก เช่น การกำหนดค่าต่างๆ อย่างจำนวนกรณีทดสอบที่ต้องการสร้าง ดูรายละเอียดเพิ่มเติมได้ที่[เอกสารประกอบ](https://docs.rs/proptest/latest/proptest/macro.proptest.html)
