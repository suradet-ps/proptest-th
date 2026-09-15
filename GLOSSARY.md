# พจนานุกรมศัพท์ (Glossary) — proptest ฉบับภาษาไทย

ตารางนี้รวบรวมคำศัพท์เชิงเทคนิคและแนวทางการแปลที่ใช้อย่างสม่ำเสมอตลอดทั้งเล่ม เพื่อให้การแปลทุกบทมีความถูกต้อง ลื่นไหล และเป็นธรรมชาติสำหรับนักพัฒนาซอฟต์แวร์ชาวไทย

| ศัพท์ต้นฉบับ | คำแปลไทย | หมายเหตุ |
|---|---|---|
| property testing | การทดสอบพร็อพเพอร์ตี (property testing) | หรือการทดสอบเชิงคุณสมบัติ กล่าวถึงครั้งแรกด้วย "การทดสอบพร็อพเพอร์ตี (property testing)" |
| property | พร็อพเพอร์ตี | คุณสมบัติหรือข้อกำหนดที่ต้องเป็นจริงกับอินพุตทุกค่า |
| strategy | กลยุทธ์ (strategy) | คง `Strategy` เมื่อหมายถึงเทรตหรือชนิดข้อมูลในโค้ด |
| test case | กรณีทดสอบ / เคสทดสอบ | "case" เดี่ยวใช้ "กรณี" หรือ "เคส" ตามบริบท |
| test runner | ตัวรันทดสอบ | คง `TestRunner` เมื่อหมายถึงชนิดข้อมูล |
| shrinking / shrink | การย่อขนาดเคสทดสอบ (shrinking) / ชริงก์ (shrink) | การตัดทอน/ย่ออินพุตที่ล้มเหลวให้มีขนาดเล็กที่สุดที่ยังทำให้เกิดบั๊ก |
| failure persistence | การบันทึกกรณีล้มเหลวไว้ทดสอบซ้ำ (failure persistence) | กลไกบันทึกเคสที่ล้มเหลวลงไฟล์ `proptest-regressions` เพื่อป้องกันการเกิดบั๊กซ้ำ (regression) |
| seed | ซีด (seed) | ค่าตั้งต้นของตัวกำเนิดเลขสุ่ม |
| fork / forking | การฟอร์ก (fork) | การแยกรันกรณีทดสอบในโปรเซสย่อย |
| timeout | ไทม์เอาต์ (timeout) | การจำกัดเวลาทำงาน |
| trait | เทรต (trait) | |
| crate | เครต (crate) | |
| macro | มาโคร (macro) | |
| combinator | คอมบิเนเตอร์ (combinator) | ฟังก์ชันที่ใช้ประกอบกลยุทธ์ เช่น `prop_map`, `prop_filter` |
| arbitrary | ค่าใดๆ / สุ่มค่าใดๆ (arbitrary) | หมายถึงค่าใดๆ โดยไม่เจาะจง (ห้ามแปลว่า "ตามอำเภอใจ"), คง `Arbitrary` เมื่อหมายถึงเทรต |
| enum | enum (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| variant | วาเรียนต์ (variant) | ตัวเลือกย่อยของ enum |
| struct | struct (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| field | ฟิลด์ (field) | |
| derive | derive (คงชื่อเดิม) | |
| regular expression | เรกูลาร์เอ็กซ์เพรสชัน (regular expression) | หรือนิพจน์ปรกติ / regex |
| filter / filtering | การกรอง / ตัวกรอง | คง `prop_filter` ในโค้ด |
| rejection sampling | การสุ่มแบบปฏิเสธ (rejection sampling) | การสุ่มค่าขึ้นมาแล้วทิ้งไปหากไม่ตรงตามเงื่อนไขตัวกรอง |
| state machine | สเตตแมชชีน (state machine) | แบบจำลองสถานะและการเปลี่ยนสถานะ |
| transition | ทรานซิชัน / การเปลี่ยนสถานะ (transition) | |
| invariant | อินแวเรียนต์ (invariant) | กฎหรือคุณสมบัติที่ต้องคงความเป็นจริงอยู่เสมอทุกสถานะ |
| pre-condition / post-condition | เงื่อนไขก่อนทำงาน / เงื่อนไขหลังทำงาน (pre-condition / post-condition) | |
| system under test (SUT) | ระบบที่กำลังทดสอบ (SUT) | |
| counter-example | ตัวอย่างค้าน (counter-example) | อินพุตหรือกรณีที่พิสูจน์ว่าพร็อพเพอร์ตีไม่เป็นจริง |
| assertion | แอสเซอร์ชัน (assertion) | การตรวจสอบเงื่อนไขในโค้ด |
| panic | การแพนิก (panic) | |
| fuzz / fuzzing | ฟัซ / การฟัซ (fuzzing) | การทดสอบด้วยการป้อนอินพุตสุ่มจำนวนมหาศาล |
| bound | ข้อกำหนดขอบเขตชนิดข้อมูล / บาวด์ (bound) | เช่น `T: Arbitrary` |
| modifier | ตัวปรับแต่ง (modifier) | ตัวกำหนดพฤติกรรมใน `#[proptest(..)]` |
| attribute | แอตทริบิวต์ (attribute) | |
| canonical | แบบมาตรฐาน (canonical) | |
| trait object | เทรตออบเจกต์ (trait object) | |
| dynamic dispatch | การดิสแพตช์แบบไดนามิก (dynamic dispatch) | |
| uninhabited | ไม่มีค่าที่เป็นไปได้ (uninhabited) | ชนิดข้อมูลที่ไม่สามารถสร้างอินสแตนซ์ได้ (เช่น `!`) |
| lifetime | ไลฟ์ไทม์ (lifetime) | ช่วงอายุของข้อมูลในหน่วยความจำ |
| entropy | เอนโทรปี (entropy) | แหล่งความสุ่ม (entropy source) |
| heap | ฮีป (heap) | หน่วยความจำฮีป |
| distribution | การแจกแจง (distribution) | การแจกแจงความน่าจะเป็นของการสุ่มค่า |
| unit variant / unit struct | variant / struct แบบยูนิต (unit) | |
| closure | โคลเชอร์ (closure) | |

## หลักการทั่วไป

- **ชื่อทางเทคนิค**: ชื่อเครื่องมือ คำสั่ง CLI ตัวเลือก (flag) ชื่อเครต/เทรต/ฟังก์ชัน/ชนิดข้อมูล และ URL **ไม่แปล** เช่น `proptest`, `proptest-derive`, `Strategy`, `Config`, `#[derive(Arbitrary)]`
- **โค้ดทุกบล็อก (` ```...``` `)**: ต้องคงไว้ตามต้นฉบับภาษาอังกฤษทุกตัวอักษร (byte-identical) รวมถึงคอมเมนต์และการเว้นวรรคภายในโค้ด
- **ลิงก์ (Links)**: ทั้ง inline links และ reference links ต้องชี้ไปยัง target เดิมเสมอ เพื่อให้ mdbook build ผ่านและไม่เกิด broken links
- **หัวข้อ (Headings)**: แปลเป็นไทยอย่างเป็นธรรมชาติและกระชับ ยกเว้นหัวข้อที่เป็นชื่อตัวปรับแต่งของ `proptest-derive` (เช่น `filter`, `weight`, `params`) ซึ่งคงไว้ตามเดิมเพราะเป็นชื่อ API
- **Anchor ของลิงก์ภายในเล่ม**: ตรวจสอบกับ HTML ที่ mdbook ทำการ build แล้วเสมอ โดย mdbook จะตัดสระ/วรรณยุกต์ไทยออกจาก slug อัตโนมัติ จึงต้องตรวจสอบด้วย `scripts/check-links.ps1` ทุกครั้ง
