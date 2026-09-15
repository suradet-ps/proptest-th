# เริ่มต้นใช้งาน

## Cargo

ในส่วน `[dev-dependencies]` ของไฟล์ `Cargo.toml` ให้เพิ่ม:

```toml
proptest-derive = "0.8.0"
```

สำหรับเครตที่ใช้ Rust 2015 คุณต้องเพิ่มบรรทัดนี้:

```
#[cfg(test)] extern crate proptest;
```

ไว้ที่ส่วนบนสุดของเครต

### เกี่ยวกับการกำหนดเวอร์ชัน

ปัจจุบัน `proptest-derive` ยังอยู่ในขั้นทดลองและมีเลขเวอร์ชันแยกต่างหากเป็นของตัวเอง เมื่อไลบรารีมีความเสถียรมากขึ้น จะมีการปรับเลขเวอร์ชันให้ตรงกัน (lock-step) กับเครตหลัก `proptest`

## การใช้ derive

ภายในโมดูลทดสอบใดๆ ของคุณ คุณสามารถเพิ่มแอตทริบิวต์ `#[derive(Arbitrary)]` ลงบนการประกาศ struct หรือ enum ได้ทันที:

```rust
#[cfg(test)]
mod test {
    use proptest::prelude::*;
    use proptest_derive::Arbitrary;

    #[derive(Arbitrary, Debug)]
    struct MyStruct {
        // ...
    }

    proptest! {
        #[test]
        fn test_one(my_struct: MyStruct) {
            // ...
        }

        // Equivalent to the above
        fn test_two(my_struct in any::<MyStruct>()) {
            // ...
        }
    }
}
```

หากต้องการใช้ `proptest-derive` กับชนิดข้อมูลที่ _ไม่ได้_ อยู่ในโมดูลทดสอบ (เช่น ชนิดข้อมูลที่อยู่ในโค้ดหลักของโปรแกรม) โดยไม่ต้องการให้บิลด์โปรดักชันหลักต้องมี dependency ผูกติดกับ proptest คุณจำเป็นต้องครอบแอตทริบิวต์ที่เกี่ยวข้องด้วย `#[cfg_attr(test, ...)]` ด้วยตนเอง ซึ่งทางทีมงานมีแผนจะ [ปรับปรุงจุดนี้ให้สะดวกขึ้นในอนาคต](https://github.com/proptest-rs/proptest/pull/106)

```rust
#[cfg(test)] use proptest_derive::Arbitrary;

#[derive(Debug)]
// derive(Arbitrary) is only available in tests
#[cfg_attr(test, derive(Arbitrary))]
struct MyStruct {
    // Attributes consumed proptest-derive must not be added when the
    // declaration is not being processed by derive(Arbitrary).
    #[cfg_attr(test, proptest(value = 42))]
    answer: u32,
    // ...
}
```
