# เริ่มต้นใช้งาน

## Cargo

ในส่วน `[dev-dependencies]` ของ `Cargo.toml` ให้เพิ่ม

```toml
proptest-derive = "0.8.0"
```

ในเครตที่ใช้ Rust 2015 คุณต้องเพิ่ม

```
#[cfg(test)] extern crate proptest;
```

ไว้ที่ด้านบนสุดของเครต

### เกี่ยวกับการกำหนดเวอร์ชัน

ปัจจุบัน `proptest-derive` ยังอยู่ในขั้นทดลองและมีเวอร์ชันเป็นของตัวเอง เมื่อมันเสถียรมากขึ้น ก็จะถูกกำหนดเวอร์ชันพร้อมกับเครต `proptest` หลัก

## การใช้ derive

ภายในโมดูลทดสอบใดๆ ของคุณ เพียงเพิ่ม `#[derive(Arbitrary)]` ให้กับการประกาศ struct หรือ enum

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

หากต้องการใช้ `proptest-derive` กับชนิดข้อมูลที่_ไม่_ได้อยู่ในโมดูลทดสอบ โดยไม่ต้องพึ่งพา proptest สำหรับบิลด์หลักด้วย ปัจจุบันคุณต้องปิดกั้นแอตทริบิวต์ที่เกี่ยวข้องด้วยมือ นี่เป็นสิ่งที่เราวางแผนจะ[ปรับปรุงในอนาคต](https://github.com/proptest-rs/proptest/pull/106)


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
