# การรองรับ Web Assembly

ตั้งแต่เวอร์ชัน 0.9.2 เป็นต้นมา คุณสามารถคอมไพล์ Proptest บนแพลตฟอร์มเป้าหมาย `wasm` (WebAssembly) ได้แล้ว โปรดทราบว่าการทำงานในส่วนนี้ยังอยู่ใน**ขั้นทดลองอย่างยิ่ง (highly experimental)** และยังไม่ผ่านการทดสอบครอบคลุมในระดับที่เพียงพอ

ในไฟล์ `Cargo.toml` ให้กำหนดค่าดังนี้:

```toml
[dev-dependencies.proptest]
version = "$proptestVersion"
# The default feature set includes things like process forking which are not
# supported in Web Assembly.
default-features = false
# Enable using the `std` crate.
features = ["std"]
```

มี API บางส่วนที่ไม่สามารถใช้งานได้บนแพลตฟอร์มเป้าหมาย `wasm` (นอกเหนือจากส่วนที่ถูกตัดออกไปเมื่อปิด default features):

- การอิมพลีเมนต์เทรต `Arbitrary` สำหรับ `std::env::VarError`
