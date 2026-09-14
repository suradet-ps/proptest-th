# การรองรับ Web Assembly

ตั้งแต่เวอร์ชัน 0.9.2 เป็นต้นมา สามารถคอมไพล์ proptest บนแพลตฟอร์มเป้าหมาย `wasm` ได้ โปรดทราบว่านี่เป็นฟีเจอร์**ขั้นทดลองอย่างมาก** และยังไม่ผ่านการทดสอบอย่างจริงจังเพียงพอ

ใน `cargo.toml` ให้เขียนประมาณนี้

```toml
[dev-dependencies.proptest]
version = "$proptestVersion"
# The default feature set includes things like process forking which are not
# supported in Web Assembly.
default-features = false
# Enable using the `std` crate.
features = ["std"]
```

API บางตัวไม่พร้อมใช้งานบนแพลตฟอร์มเป้าหมาย `wasm` (นอกเหนือจากตัวที่ถูกตัดออกไปเมื่อปิดฟีเจอร์เริ่มต้นบางตัว):

- การนำเทรต `Arbitrary` ไปใช้สำหรับ `std::env::VarError`
