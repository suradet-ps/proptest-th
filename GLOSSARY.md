# พจนานุกรมศัพท์ (Glossary) — proptest ฉบับภาษาไทย

ตารางนี้เป็นคำศัพท์ที่ใช้อย่างสม่ำเสมอตลอดทั้งเล่ม เพื่อให้การแปลทุกบทใช้คำเดียวกัน

| ศัพท์ต้นฉบับ | คำแปลไทย | หมายเหตุ |
|---|---|---|
| property testing | การทดสอบพร็อพเพอร์ตี | กล่าวถึงครั้งแรกด้วย "การทดสอบพร็อพเพอร์ตี (property testing)" |
| property | พร็อพเพอร์ตี | คุณสมบัติที่ต้องเป็นจริงกับอินพุตทุกค่า |
| strategy | กลยุทธ์ (strategy) | คง `Strategy` เมื่อหมายถึงเทรตหรือชนิดข้อมูลในโค้ด |
| test case | กรณีทดสอบ | "case" เดี่ยวใช้ "กรณี" |
| test runner | ตัวรันทดสอบ | คง `TestRunner` เมื่อหมายถึงชนิดข้อมูล |
| shrinking / shrink | การชริงก์ / ชริงก์ | การย่ออินพุตที่ล้มเหลวให้เล็กลงที่สุด |
| failure persistence | การคงอยู่ของกรณีล้มเหลว | ไฟล์ `proptest-regressions` |
| seed | ซีด (seed) | ค่าตั้งต้นของตัวสุ่ม |
| fork / forking | การฟอร์ก (fork) | รันกรณีทดสอบในโปรเซสย่อย |
| timeout | ไทม์เอาต์ | |
| trait | เทรต (trait) | |
| crate | เครต | |
| macro | มาโคร | |
| combinator | คอมบิเนเตอร์ (combinator) | เช่น `prop_map`, `prop_filter` |
| arbitrary | ตามอำเภอใจ (arbitrary) | คง `Arbitrary` เมื่อหมายถึงเทรต |
| enum | enum (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| variant | วาเรียนต์ (variant) | |
| struct | struct (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| field | ฟิลด์ (field) | |
| derive | derive (คงชื่อเดิม) | |
| regular expression | นิพจน์ปรกติ (regular expression) | |
| filter / filtering | ตัวกรอง / การกรอง | คง `prop_filter` ในโค้ด |
| rejection sampling | การสุ่มแบบปฏิเสธ (rejection sampling) | |
| state machine | สเตตแมชชีน (state machine) | |
| transition | ทรานซิชัน (transition) | |
| invariant | อินแวเรียนต์ (invariant) | |
| pre-condition / post-condition | เงื่อนไขก่อน / เงื่อนไขหลัง | |
| system under test (SUT) | ระบบที่ทดสอบ (SUT) | |
| counter-example | ตัวอย่างค้าน | |
| assertion | แอสเซอร์ชัน | |
| panic | การแพนิก (panic) | |
| fuzz / fuzzing | ฟัซ / การฟัซ | |
| bound | บาวด์ (bound) | เช่น `T: Arbitrary` |
| modifier | ตัวปรับแต่ง (modifier) | ใช้กับ `#[proptest(..)]` |
| attribute | แอตทริบิวต์ (attribute) | |
| canonical | แบบมาตรฐาน (canonical) | |
| trait object | เทรตออบเจกต์ (trait object) | |
| dynamic dispatch | การดิสแพตช์แบบไดนามิก (dynamic dispatch) | |
| uninhabited | ไม่มีค่าใดๆ (uninhabited) | ชนิดที่สร้างค่าไม่ได้ |
| lifetime | ไลฟ์ไทม์ (lifetime) | |
| entropy | เอนโทรปี (entropy) | แหล่งสุ่ม |
| heap | ฮีป | |
| distribution | การกระจาย (distribution) | |
| unit variant / unit struct | variant / struct แบบยูนิต | |
| closure | โคลเชอร์ (closure) | |

## หลักการทั่วไป

- ชื่อเครื่องมือ คำสั่ง CLI ตัวเลือก (flag) ชื่อเครต/เทรต/ชนิดข้อมูล และ URL **ไม่แปล** เช่น `proptest`, `proptest-derive`, `Strategy`, `Config`, `#[derive(Arbitrary)]`
- โค้ดทุกบล็อก (``` ... ```) เก็บไว้ตามต้นฉบับทุกตัวอักษร รวมถึงคอมเมนต์ภายในโค้ด
- ลิงก์ (ทั้ง inline และ reference-style) คง path เดิม เพื่อให้ mdbook ยัง build ได้
- หัวข้อ (heading) แปลเป็นไทย ยกเว้นหัวข้อที่เป็นชื่อตัวปรับแต่งของ `proptest-derive` (เช่น `filter`, `weight`) ซึ่งคงไว้ตามเดิมเพราะเป็นชื่อ API
- anchor ของลิงก์ภายในเล่มอ่านจาก HTML ที่ build แล้วเสมอ (mdbook ตัดวรรณยุกต์ไทยออกจาก slug) จึงไม่เดา anchor เอง
