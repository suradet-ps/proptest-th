# เริ่มต้นใช้งาน

สมมติว่าเราต้องการเขียนฟังก์ชันสำหรับแยกวิเคราะห์วันที่ (date parser) ในรูปแบบ `YYYY-MM-DD` โดยในที่นี้เราจะไม่เน้นการ _ตรวจสอบความถูกต้องของปฏิทินจริง_ มากนัก ขอเพียงแยกออกมาเป็นจำนวนเต็มสามค่าได้ก็พอ งั้นเราลองมาเขียนโค้ดแบบเร็วๆ กันดู:

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

คอมไพล์ผ่านเรียบร้อย แสดงว่าโค้ดใช้ได้แล้วใช่ไหม? อาจจะยังไม่แน่ใจ มาลองเขียนเทสต์กันหน่อย:

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

เทสต์ทั้งหมดผ่านฉลุย นำขึ้นโปรดักชันได้เลย! แต่หลังจากนั้นไม่นาน แอปพลิเคชันของคุณก็เริ่มเกิดแครช และผู้ใช้งานก็เริ่มบ่นเมื่อพบว่าวันคริสต์มาสดันถูกเลื่อนไปอยู่วันอื่น เห็นได้ชัดว่าเราต้องตรวจสอบโค้ดให้รัดกุมกว่านี้

ในไฟล์ `Cargo.toml` ให้เพิ่มดีเพนเดนซี:

```toml
[dev-dependencies]
proptest = "1.11.0"
```

ตอนนี้เราพร้อมที่จะเพิ่มการทดสอบพร็อพเพอร์ตีให้กับตัวแยกวิเคราะห์วันที่แล้ว แต่คำถามคือ: เราจะทดสอบตัวแยกวิเคราะห์วันที่กับอินพุตค่าใดๆ (arbitrary inputs) ได้อย่างไรโดยไม่ต้องเขียน parser อีกตัวขึ้นมาเพื่อตรวจคำตอบ? คำตอบคือคุณไม่จำเป็นต้องทำเช่นนั้นเลย ตราบใดที่คุณเลือกอินพุตและกำหนดพร็อพเพอร์ตีได้อย่างเหมาะสม และก่อนที่จะไปถึงเรื่องความถูกต้องของผลลัพธ์ ยังมีพร็อพเพอร์ตีที่เรียบง่ายกว่านั้นให้ทดสอบก่อน นั่นคือ: _ฟังก์ชันต้องไม่แครช (panic)_ เรามาเริ่มจากตรงนี้กัน:

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

โค้ดนี้จะทำการสุ่มสร้างค่า `&String` ขึ้นมาจริงๆ (ตอนนี้ยังไม่ต้องกังวลกับ `\\PC*` เดี๋ยวเราจะอธิบายกันต่อไป — แต่ถ้าใครคุ้นเคยอยู่แล้วก็ขอให้อดใจรอสักครู่) จากนั้นส่งค่านั้นไปให้ `parse_date()` แล้วไม่สนใจผลลัพธ์ที่ได้

เมื่อสั่งรัน เราจะได้เอาต์พุตแจ้งข้อผิดพลาดขนาดยาว ซึ่งลงท้ายด้วย:

```text
thread 'main' panicked at 'Test failed: byte index 4 is not a char boundary; it is inside 'ௗ' (bytes 2..5) of `aAௗ0㌀0`; minimal failing input: s = "aAௗ0㌀0"
	successes: 102
	local rejects: 0
	global rejects: 0
'
```

หากลองดูที่ไดเรกทอรีราก (root directory) หลังการทดสอบล้มเหลว จะพบโฟลเดอร์ใหม่ชื่อ `proptest-regressions` ซึ่งบรรจุไฟล์ที่ตั้งชื่อตามไฟล์ซอร์สโค้ดที่มีการทดสอบล้มเหลว ไฟล์เหล่านี้คือกลไก[_การบันทึกกรณีล้มเหลวไว้ทดสอบซ้ำ (Failure Persistence)_](https://proptest-rs.github.io/proptest/proptest/failure-persistence.html) โดยสิ่งแรกที่เราควรทำคือเพิ่มไฟล์เหล่านี้เข้าสู่ระบบควบคุมเวอร์ชัน (Git):

```text
$ git add proptest-regressions
```

สิ่งถัดมาที่ควรทำคือ นำกรณีทดสอบที่ล้มเหลวนี้ไปเขียนเป็นยูนิตเทสต์แบบดั้งเดิมแยกไว้ต่างหาก เพราะเคสนี้ได้เผยให้เห็นบั๊กในรูปแบบที่เราไม่เคยนึกถึงมาก่อน:

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

ทีนี้มาดูกันว่าเกิดอะไรขึ้น... เราลืมคำนึงถึงเรื่อง UTF-8! ใน Rust เราไม่สามารถสไลซ์สตริง (string slicing) ตรงๆ ตาม byte index ได้ตามใจชอบ เพราะอาจไปตัดกึ่งกลางของตัวอักษรหลายไบต์ ซึ่งในเคสนี้คือเครื่องหมายสระในภาษาทมิฬที่ผสมอยู่กับอักขระอื่นในสตริง

เพื่อให้การแก้ไขโค้ดน้อยที่สุด เราจะตรวจสอบว่าสตริงเป็น ASCII หรือไม่ และปฏิเสธสตริงใดก็ตามที่ไม่ใช่:

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

ตอนนี้เทสต์ผ่านเรียบร้อยแล้ว! แต่เรารู้ว่ายังมีปัญหาอื่นอยู่อีก เรามาเพิ่มการทดสอบพร็อพเพอร์ตีกันต่อดีกว่า

พร็อพเพอร์ตีอีกข้อที่เราต้องการคือ ฟังก์ชันจะต้องสามารถแยกวิเคราะห์วันที่ที่ถูกต้องได้ทุกวัน เราสามารถเพิ่มเทสต์อีกตัวลงในบล็อก `proptest!` ได้ดังนี้:

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

สิ่งที่อยู่ทางขวามือของคีย์เวิร์ด `in` แท้จริงแล้วคือ _เรกูลาร์เอ็กซ์เพรสชัน (regular expression)_ โดย `s` จะถูกสุ่มขึ้นมาจากสตริงที่แมตช์กับแพตเทิร์นนั้น ดังนั้นในเทสต์ก่อนหน้านี้ นิพจน์ `"\\PC*"` จึงทำหน้าที่สุ่มสตริงค่าใดๆ ที่ประกอบด้วยอักขระที่ไม่ใช่อักขระควบคุม (control characters) ส่วนในเทสต์นี้ เรากำหนดให้สุ่มค่าสตริงที่ตรงตามรูปแบบ YYYY-MM-DD

เทสต์ใหม่นี้ผ่านฉลุย งั้นเรามาดูประเด็นถัดไปกัน

พร็อพเพอร์ตีสุดท้ายที่เราต้องการตรวจสอบ คือวันที่ต้องถูกแปลงออกมาได้อย่าง _ถูกต้อง_ จริงๆ ซึ่งเราไม่สามารถทำได้ด้วยการสุ่มสร้างสตริงขึ้นมาตรงๆ — เพราะไม่อย่างนั้นเราก็ต้องเขียนตัวแยกวิเคราะห์วันที่อีกตัวขึ้นมาเพื่อเทียบผลลัพธ์ในเทสต์! วิธีแก้คือ ให้เริ่มจากข้อมูลผลลัพธ์ที่ถูกต้อง (ปี, เดือน, วัน) นำมาจัดรูปแบบเป็นสตริง แล้วค่อยทดสอบว่าตัวแยกวิเคราะห์สามารถแปลงสตริงนั้นกลับมาเป็นค่าเดิมได้ถูกต้องหรือไม่:

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

จะเห็นได้ว่านอกจากเรกูลาร์เอ็กซ์เพรสชันแล้ว เรายังสามารถใช้นิพจน์ใดๆ ก็ได้ที่เป็นชนิดข้อมูลซึ่งอิมพลีเมนต์ `proptest::strategy::Strategy` ซึ่งในกรณีนี้คือช่วงตัวเลขจำนวนเต็ม (integer ranges)

พอสั่งรัน เทสต์ก็ล้มเหลวทันที ครั้งนี้เอาต์พุตสั้นกระชับกว่าเดิม:

```text
thread 'main' panicked at 'Test failed: assertion failed: `(left == right)` (left: `(0, 10, 1)`, right: `(0, 0, 1)`) at examples/dateparser_v2.rs:46; minimal failing input: y = 0, m = 10, d = 1
	successes: 2
	local rejects: 0
	global rejects: 0
', examples/dateparser_v2.rs:33
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

อินพุตที่ทำให้เกิดข้อผิดพลาดคือ `(y, m, d) = (0, 10, 1)` ซึ่งเป็นค่าที่ค่อนข้างเฉพาะเจาะจง ก่อนจะไปคิดหาสาเหตุว่าทำไมค่านี้ถึงทำให้โค้ดพัง มาดูกันก่อนว่า Proptest ทำงานอย่างไรจนได้ค่าน้อยที่สุดนี้มา โดยลองแทรกโค้ดพิมพ์ค่าไว้ที่บรรทัดแรกของฟังก์ชันทดสอบ:

```rust
    # let (y, m, d) = (0, 10, 1);
    println!("y = {}, m = {}, d = {}", y, m, d);
```

เมื่อรันเทสต์อีกครั้ง เราจะเห็นผลลัพธ์ขั้นตอนการชริงก์ดังนี้:

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

ข้อความแจ้งเตือนความล้มเหลวระบุว่ามีเคสที่รันผ่านไปแล้ว 2 ครั้ง ดังที่เห็นด้านบนสุดคือ `2497-08-27` และ `9641-08-18` จากนั้นเคสถัดมา `7360-12-20` เกิดล้มเหลวขึ้น ค่าวันที่นี้ดูเผินๆ เหมือนไม่มีอะไรพิเศษ แต่โชคดีที่ Proptest ช่วยย่อขนาด (shrink) ให้เหลือเคสที่วิเคราะห์ได้ง่ายกว่ามาก: ขั้นแรก มันปรับลดค่า `y` (ปี) ลงเหลือ `0` อย่างรวดเร็ว และปรับลดค่า `d` (วัน) ลงเหลือค่าต่ำสุดที่อนุญาตคือ `1` ส่วนระหว่างนั้น เราจะเห็นพฤติกรรมที่น่าสนใจ: Proptest ลองย่อค่า `12` (เดือน) ลงเหลือ `6` แต่ในที่สุดก็ปรับกลับขึ้นมาเป็น `10` นั่นเป็นเพราะเคส `0000-06-20` และ `0000-09-20` นั้น _ผ่าน_ (ไม่ทำให้เกิดบั๊ก)

ในท้ายที่สุด เราจึงได้อินพุตที่มีขนาดเล็กที่สุดคือ `0000-10-01` ซึ่งถูก parse ออกมาผิดพลาดกลายเป็น `(0, 0, 1)` เคสที่ล้มเหลวนี้จะถูกบันทึกไว้ในไฟล์ของกลไก Failure Persistence เช่นเดิม และเราควรนำเคสนี้ไปเขียนเป็นยูนิตเทสต์ไว้ด้วย:

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

คราวนี้มาหาสาเหตุกันว่าโค้ดผิดพลาดตรงไหน แม้จะไม่ได้ดูประวัติอินพุตระหว่างทาง แต่เราสามารถสรุปได้อย่างมั่นใจว่าส่วนของปีและวันไม่ได้เป็นสาเหตุของบั๊ก เพราะทั้งสองค่าถูกย่อขนาดลงจนถึงค่าต่ำสุดที่อนุญาตแล้ว แต่ค่าเดือน _ไม่ได้_ ถูกย่อลงเหลือค่าต่ำสุด กลับหยุดอยู่ที่ `10` นั่นหมายความว่าต้องมีอะไรบางอย่างที่พิเศษเกี่ยวกับเลข `10` ซึ่งต่างจากเลข `9` และในบริบทนี้ ความพิเศษนั้นก็คือมันเป็นตัวเลขสองหลักนั่นเอง เมื่อย้อนกลับไปดูโค้ดของเรา:

```rust,ignore
    let month = &s[6..7];
```

เรากำหนดช่วงดัชนีพลาดไปหนึ่งตำแหน่ง ที่ถูกต้องคือต้องใช้ช่วง `5..7` หลังจากแก้ไขเรียบร้อยแล้ว เทสต์ก็ผ่านทั้งหมด

มาโคร `proptest!` ยังมีไวยากรณ์เพิ่มเติมสำหรับการปรับแต่งค่าคอนฟิกต่างๆ เช่น จำนวนกรณีทดสอบที่ต้องการสุ่มสร้างขึ้นมา โดยสามารถดูรายละเอียดเพิ่มเติมได้ที่ [เอกสารประกอบ](https://docs.rs/proptest/latest/proptest/macro.proptest.html)
