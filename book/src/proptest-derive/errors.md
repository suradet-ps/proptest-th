# ดัชนีข้อผิดพลาด

[ตัวติดตาม issue]: https://github.com/proptest-rs/proptest

## E0001

[พารามิเตอร์ไลฟ์ไทม์]: https://doc.rust-lang.org/stable/book/second-edition/ch10-03-lifetime-syntax.html#lifetime-annotations-in-struct-definitions

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับชนิดข้อมูลที่มี[พารามิเตอร์ไลฟ์ไทม์]อยู่ ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo<'a> {
    bar: &'a str,
}
```

[issue#9]: https://github.com/proptest-rs/proptest/issues/9

ปัจจุบันยังไม่สามารถนิยาม `Strategy` ที่สร้างชนิดข้อมูลซึ่งเป็นเจเนอริกด้านไลฟ์ไทม์ได้ (เช่น `&'a T`) ดังนั้น proptest จึงไม่สามารถอิมพลีเมนต์เทรต `Arbitrary` ให้กับชนิดข้อมูลลักษณะนั้นได้เช่นกัน และด้วยเหตุนี้คุณจึงไม่สามารถใช้ `#[derive(Arbitrary)]` กับชนิดข้อมูลเหล่านั้นได้ ปัจจุบัน GAT (Generic Associated Types) สามารถใช้งานได้บน Rust เวอร์ชันเสถียร (stable) ตั้งแต่ 1.65 แล้ว และเราจะกลับมาพิจารณาว่าจะรองรับฟีเจอร์นี้อย่างไร ติดตามความคืบหน้าได้ที่[issue ติดตามเรื่องนี้][issue#9]

## E0002

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับชนิดข้อมูลแบบ `union` ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
union IU32 {
    signed: i32,
    unsigned: u32,
}
```

สาเหตุหลักของข้อผิดพลาดนี้มีสองประการ:

1. ไม่สามารถ `#[derive(Debug)]` บนชนิดข้อมูล `union` ได้ และการอิมพลีเมนต์ด้วยตนเองก็ไม่สามารถล่วงรู้ได้ว่าแวเรียนต์ใดถูกต้อง จึงแทบไม่มีวิธีอิมพลีเมนต์ที่ถูกต้องและเป็นไปได้เลย

2. เราไม่สามารถระบุได้โดยอัตโนมัติว่าจะสร้างแวเรียนต์ใดระหว่าง `signed` กับ `unsigned` แม้ว่าเราจะสามารถเปิดให้คุณระบุผ่านแอตทริบิวต์อย่าง `#[proptest(select)]` บนแวเรียนต์ได้ แต่ในตอนนี้เราเลือกใช้แนวทางที่รัดกุมกว่า หากคุณมีกรณีการใช้งานจริงสำหรับ `#[derive(Arbitrary)]` บนชนิดข้อมูล `union` โปรดแจ้งให้เราทราบผ่าน[ตัวติดตาม issue]

## E0003

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ struct ที่มี[ชนิดข้อมูลที่ไม่สามารถสร้างอินสแตนซ์ได้ (uninhabited types)](https://doc.rust-lang.org/nomicon/exotic-sizes.html#empty-types) ซึ่งหมายความว่า struct ดังกล่าวเองก็ไม่สามารถสร้างอินสแตนซ์ได้เช่นกัน ด้วยเหตุนี้จึงไม่มีการอิมพลีเมนต์ `Arbitrary` ที่สมเหตุสมผลได้ เพราะไม่สามารถสร้างค่าของ struct นั้นออกมาได้

ตัวอย่างง่ายๆ:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Uninhabited {
    inhabited: u32,
    never: !,
}
```

เนื่องจากไม่มีค่าใดที่สามารถกำหนดให้กับฟิลด์ `never` ได้ จึงเป็นไปไม่ได้ที่จะสร้างอินสแตนซ์ของ struct `Uninhabited`

ความสามารถของ Proptest ในการระบุ uninhabited type นั้นยังมีจำกัด หากระบบไม่รู้จักว่าชนิดข้อมูลหนึ่งเป็น uninhabited type ระบบจะสันนิษฐานว่าชนิดข้อมูลนั้นมีค่าได้ (inhabited) และคุณจะได้รับข้อผิดพลาดแจ้งว่าชนิดข้อมูลดังกล่าวไม่ได้อิมพลีเมนต์เทรต `Arbitrary` แทน

## E0004

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ enum ที่ไม่มีแวเรียนต์เลย ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Uninhabited {}
```

enum ลักษณะนี้ไม่มีค่าใดๆ ที่เป็นไปได้เลย จึงไม่สมเหตุสมผลที่จะมีการอิมพลีเมนต์ `Arbitrary` ให้ เนื่องจากไม่สามารถสุ่มสร้างค่าใดๆ ขึ้นมาได้

## E0005

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ enum ที่แวเรียนต์ทั้งหมดเป็น uninhabited types โดยใช้ตรรกะเดียวกับที่อธิบายไว้ใน [`E0003`](#e0003) ส่งผลให้ตัว enum นั้นเองไม่สามารถสร้างอินสแตนซ์ใดๆ ได้โดยสิ้นเชิง

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Uninhabited {
    Never(!),
    NeverEver(!, !),
}
```

## E0006

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ enum ที่แวเรียนต์ซึ่งมีค่าได้ทั้งหมดถูกกำกับด้วย [`#[proptest(skip)]`] กล่าวอีกนัยหนึ่งคือ proptest ถูกห้ามไม่ให้สร้างแวเรียนต์ใดๆ ของ enum นั้น ส่งผลให้ไม่สามารถสร้างค่าของ enum ดังกล่าวได้เลย

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum MyEnum {
    // Ordinarily, proptest would be able to generate either of these variants,
    // but both are forbidden, so in the end proptest isn't allowed to generate
    // anything at all.
    #[proptest(skip)]
    UnitVariant,
    #[proptest(skip)]
    SimpleVariant(u32),
    // This variant is implicitly skipped because proptest knows it is
    // uninhabited.
    Uninhabited(!),
}
```

## E0007

ข้อผิดพลาดนี้เกิดขึ้นเมื่อแอตทริบิวต์ [`#[proptest(strategy = "expr")]`] หรือ [`#[proptest(value = "expr")]`] ถูกนำไปใช้กับไอเท็มเดียวกับที่มี `#[derive(Arbitrary)]` อยู่

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(value = "MyStruct(42)")]
struct MyStruct(u32);
```

กรณีนี้ถูกปฏิเสธเนื่องจากไม่มีสิ่งใดที่ถูก "derive" จริงๆ คุณควรเขียนอิมพลีเมนต์เทรต `Arbitrary` ขึ้นมาเองโดยตรงแทน

## E0008

ข้อผิดพลาดนี้เกิดขึ้นเมื่อใช้ [`#[proptest(skip)]`] กับไอเท็มที่ไม่สามารถข้ามได้ ตัวอย่างเช่น ฟิลด์ของ struct ไม่สามารถข้ามได้ เพราะ Rust บังคับให้ทุกฟิลด์ของ struct ต้องมีค่า

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct WidgetContainer {
    desired_widget_count: usize,
    #[proptest(skip)]
    widgets: Vec<Widget>,
}
```

โดยทั่วไป หากคุณต้องการบอกไม่ให้ proptest สุ่มสร้างค่าของฟิลด์ แนวทางที่ถูกต้องคือการใช้ [`#[proptest(value = "expr")]`] เพื่อกำหนดค่าคงที่ด้วยตนเอง ตัวอย่างเช่น โค้ดด้านบนสามารถเขียนให้ถูกต้องได้ดังนี้:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct WidgetContainer {
    desired_widget_count: usize,
    #[proptest(value = "vec![]")] // Always generate an empty widget vec
    widgets: Vec<Widget>,
}
```

## E0009

ข้อผิดพลาดนี้เกิดขึ้นเมื่อใช้ [`#[proptest(weight = <integer>)]`] กับไอเท็มที่ไม่สมเหตุสมผล เช่น ฟิลด์ของ struct ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Point {
    x: u32,
    #[proptest(weight = 42)]
    y: u32,
}
```

แอตทริบิวต์ `weight` จะมีความหมายก็ต่อเมื่อ proptest มีตัวเลือกระหว่างหลายไอเท็ม ซึ่งก็คือบรรดาแวเรียนต์ของ enum ในทางตรงกันข้าม สำหรับฟิลด์ของ struct นั้น proptest จำเป็นต้องกำหนดค่าให้ครบ_ทุก_ฟิลด์ จึงไม่มีการเลือกว่า "จะเอาอันนี้หรืออันนั้น"

## E0010

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการกำหนด [`#[proptest(params = "type")]`] และ/หรือ [`#[proptest(no_params)]`] ทั้งบนตัวไอเท็มเองและบนไอเท็มแม่ของมัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(params = "String")]
struct Foo {
    #[proptest(no_params)]
    bar: String,
}
```

หากไอเท็มแม่มีการกำหนดพารามิเตอร์ไว้อย่างชัดเจน การกำหนดนั้นจะถือเป็นพารามิเตอร์สำหรับอิมพลีเมนต์ `Arbitrary` ทั้งหมด และไอเท็มลูกจะต้องทำงานร่วมกับค่านั้นโดยไม่สามารถระบุพารามิเตอร์ของตนเองได้

## E0011

ข้อผิดพลาดนี้เกิดขึ้นเมื่อกำหนด [`#[proptest(params = "type")]`] บนฟิลด์ แต่ไม่ได้ระบุกลยุทธ์อย่างชัดเจนด้วย [`#[proptest(strategy = "expr")]`] หรือตัวปรับแต่งอื่นในลักษณะเดียวกัน ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(param = "u8")]
    some_string: String,
}
```

ตัวอย่างนี้แสดงให้เห็นว่าทำไมต้องระบุควบคู่กัน: การอิมพลีเมนต์ `Arbitrary` ของ `String` รับพารามิเตอร์ชนิด `proptest::string::StringParam` แต่ในที่นี้เราพยายามส่ง `u8` เข้าไป

แม้ว่าโค้ดที่ถูกสร้างขึ้นอาจทำงานได้หากชนิดข้อมูลที่ระบุใน `param` ตรงกับชนิดข้อมูลของกลยุทธ์เริ่มต้น แต่การระบุชนิดพารามิเตอร์ด้วยตนเองโดยไม่เปลี่ยนกลยุทธ์ก็ไม่มีประโยชน์อันใด ดังนั้นการระบุเฉพาะ `param` เพียงอย่างเดียวจึงถูกสั่งห้ามในทุกกรณี

## E0012

ข้อผิดพลาดนี้เกิดขึ้นเมื่อกำหนด [`#[proptest(filter = "expr")]`] บนไอเท็ม แต่ไอเท็มที่ครอบมันอยู่ได้ระบุวิธีการสร้างค่าทั้งหมดโดยตรงไว้แล้ว ซึ่งทำให้ข้ามตัวกรองนี้ไปโดยไม่ได้ตรวจสอบเลย

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    #[proptest(value = "Foo::Bar(42)")]
    Bar {
        #[proptest(filter = "is_even")]
        even_number: u32,
    },
    // ...
}
```

ในตัวอย่างนี้ แวเรียนต์ `Bar` ทั้งหมดได้ระบุวิธีการสร้างค่าของตัวเองไว้โดยสมบูรณ์ ส่งผลให้ clause `filter` บน `even_number` ไม่มีโอกาสได้ทำงานเลย

## E0013

ข้อผิดพลาดนี้จะเกิดขึ้นหากมีการใช้ outer attribute ในรูปแบบ `#![proptest(..)]` กับสิ่งที่อยู่ภายใต้ `#[derive(Arbitrary)]`

ตั้งแต่ Rust 1.30.0 เป็นต้นมา ยังไม่พบวิธีที่ทำให้เกิดข้อผิดพลาดนี้ได้ เนื่องจากคอมไพเลอร์ของ Rust จะปฏิเสธแอตทริบิวต์ดังกล่าวตั้งแต่แรก

## E0014

ข้อผิดพลาดนี้เกิดขึ้นเมื่อใช้แอตทริบิวต์ `#[proptest]` แบบเปล่าๆ กับไอเท็มใดๆ เนื่องจากมันไม่มีเนื้อหาที่สื่อความหมายได้เลย

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest]
    field: u8,
}
```

รูปแบบการใช้งานที่ถูกต้องมีเพียงแบบ `#[proptest(..)]` เท่านั้น

## E0015

ข้อผิดพลาดนี้เกิดขึ้นเมื่อพบแอตทริบิวต์ในรูปแบบ `#[proptest = value]` ในบริบทใดๆ

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest = 1234]
    field: u8,
}
```

## E0016

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งค่าลิเทอรัล (literal) (ซึ่งไม่ใช่รูปแบบ `key = value`) เข้าไปใน `#[proptest(..)]` ในบริบทใดๆ

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(1234)]
    field: u8,
}
```

## E0017

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งใดๆ ของ `#[proptest(..)]` ถูกกำหนดซ้ำมากกว่าหนึ่งครั้งบนไอเท็มเดียวกัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(no_params, no_params)]
struct Foo(u32);
```

## E0018

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งตัวปรับแต่งที่ไม่รู้จักเข้าไปใน `#[proptest(..)]`

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(frobnicate = "true")]
struct Foo(u32);
```

โปรดดู[เอกสารอ้างอิงตัวปรับแต่ง](modifiers.md)เพื่อตรวจสอบว่ามีตัวปรับแต่งใดบ้างที่สามารถใช้งานได้

## E0019

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งค่าใดๆ เพิ่มเติมเข้าไปใน [`#[proptest(no_params)]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(no_params = "true")]
struct Foo(u32);
```

`no_params` ไม่รับการตั้งค่าใดๆ รูปแบบที่ถูกต้องคือ `#[proptest(no_params)]` เท่านั้น

## E0020

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งค่าใดๆ เพิ่มเติมเข้าไปใน [`#[proptest(skip)]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    Small,
    #[proptest(skip = "yes")]
    Huge(ExpensiveType),
}
```

`skip` ไม่รับการตั้งค่าใดๆ รูปแบบที่ถูกต้องคือ `#[proptest(skip)]` เท่านั้น

## E0021

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(weight = <integer>)]`] ได้รับค่าจำนวนเต็มที่ไม่ถูกต้อง หรือไม่ได้ระบุค่าใดๆ เลย

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    #[proptest(weight)]
    V1,
    #[proptest(weight = heavy)]
    V2,
}
```

รูปแบบเดียวที่ยอมรับได้คือ `#[proptest(weight = <integer>)]` โดยที่ `<integer>` ต้องเป็นลิเทอรัลจำนวนเต็มที่มีค่าอยู่ในช่วงของ `u32` หรือเป็นค่าเดียวกันที่ครอบด้วยเครื่องหมายคำพูด (quotation marks)

## E0022

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้ทั้ง [`#[proptest(no_params)]`] และ [`#[proptest(params = "type")]`] กับไอเท็มเดียวกัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(no_params, params = "u8")]
struct Foo(u32);
```

คุณต้องเลือกใช้อย่างใดอย่างหนึ่งตามผลลัพธ์ที่ต้องการ

## E0023

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้แอตทริบิวต์ [`#[proptest(params = "type")]`] ที่ไม่ถูกต้องกับไอเท็ม

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(params = "Vec<u8")] // Note missing '>'
struct Foo(u32);
```

มีหลายสาเหตุที่ทำให้เกิดข้อผิดพลาดนี้ได้:

- ไม่ระบุค่าใดๆ เลย เช่น `#[proptest(params)]`

- ส่งค่าอื่นที่ไม่ใช่สตริง เช่น `#[proptest(params = 42)]`

- ส่งชนิดข้อมูลที่มีไวยากรณ์ผิดรูปแบบภายในสตริง ดังเช่นในตัวอย่างข้างต้น (ดูเพิ่มเติมที่[ข้อควรระวังเกี่ยวกับไวยากรณ์](#ไวยากรณ-rust-ทีถูกตอง))

## E0024

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้แอตทริบิวต์ `#[proptest ..]` ที่ไม่ถูกต้องด้วยไวยากรณ์ที่เครต `proptest-derive` ยังไม่พร้อมที่จะจัดการ

เงื่อนไขที่ทำให้เกิดข้อผิดพลาดนี้ขึ้นอยู่กับเวอร์ชันของ Rust

## E0025

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้ [`#[proptest(strategy = "expr")]`], [`#[proptest(value = "expr")]`] หรือ [`#[proptest(regex = "string")]`] มากกว่าหนึ่งตัวกับไอเท็มเดียวกัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(value = "42", strategy = "Just(56)")]
    bar: u32,
}
```

ตัวปรับแต่งแต่ละตัวเหล่านี้ต่างอธิบายวิธีการสร้างค่าขึ้นมาอย่างสมบูรณ์ในตัวเอง จึงไม่สามารถนำมาใช้ร่วมกันบนสิ่งเดียวกันได้ คุณต้องเลือกใช้อย่างใดอย่างหนึ่งตามผลลัพธ์ที่ต้องการ

## E0026

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้รูปแบบที่ไม่ถูกต้องของ [`#[proptest(strategy = "expr")]`] หรือ [`#[proptest(value = "expr")]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(value = "3↑↑↑↑3")] // String content is not valid Rust syntax
    g1: u128,
}
```

มีหลายสาเหตุที่ทำให้เกิดข้อผิดพลาดนี้ได้:

- ไม่ระบุค่าใดๆ เลย เช่น `#[proptest(value)]`

- ใช้รูปแบบอื่นที่ไม่อนุญาต เช่น `#[proptest(value("a", "b"))]`

- ส่งนิพจน์ภายในสตริงที่ไม่ใช่ไวยากรณ์ Rust ที่ถูกต้อง ดังเช่นในตัวอย่างข้างต้น (ดูเพิ่มเติมที่[ข้อควรระวังเกี่ยวกับไวยากรณ์](#ไวยากรณ-rust-ทีถูกตอง))

## E0027

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้รูปแบบที่ไม่ถูกต้องของ [`#[proptest(filter = "expr")]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(filter = "> 3")] // String content is not an expression
    big_number: u128,
}
```

มีหลายสาเหตุที่ทำให้เกิดข้อผิดพลาดนี้ได้:

- ไม่ระบุค่าใดๆ เลย เช่น `#[proptest(filter)]`

- ใช้รูปแบบอื่นที่ไม่อนุญาต เช่น `#[proptest(filter("a", "b"))]`

- ส่งนิพจน์ภายในสตริงที่ไม่ใช่ไวยากรณ์ Rust ที่ถูกต้อง ดังเช่นในตัวอย่างข้างต้น (ดูเพิ่มเติมที่[ข้อควรระวังเกี่ยวกับไวยากรณ์](#ไวยากรณ-rust-ทีถูกตอง))

## E0028

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งที่บ่งบอกว่าจะต้องสร้างค่า ถูกนำไปใช้กับแวเรียนต์ของ enum ที่ถูกกำกับด้วย [`#[proptest(skip)]`] ไว้ด้วย

ตัวอย่าง:

```rust,compile_fail

#[derive(Debug, Arbitrary)]
enum Enum {
    V1(u32),
    #[proptest(skip, value = "Enum::V2(42)")]
    V2(u32),
}
```

ในกรณีนี้ ตัวปรับแต่ง [`#[proptest(value = "expr")]`] บ่งชี้ว่าผู้ใช้ต้องการให้สร้างค่าสำหรับแวเรียนต์ของ enum แต่ในขณะเดียวกัน [`#[proptest(skip)]`] กลับระบุว่าไม่ให้สร้างแวเรียนต์นั้น

## E0029

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งที่ใช้จำกัดหรือควบคุมวิธีการสร้างค่าของแวเรียนต์ใน enum ถูกนำไปใช้กับยูนิตแวเรียนต์ (unit variant)

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    #[proptest(value = "Foo::V1")]
    UnitVariant,
    // ...
}
```

ยูนิตแวเรียนต์มีค่าที่เป็นไปได้เพียงค่าเดียว จึงมีกลยุทธ์ที่เป็นไปได้เพียงหนึ่งเดียวเท่านั้น ด้วยเหตุนี้ การพยายามระบุกลยุทธ์ทางเลือกหรือการกรองแวเรียนต์ลักษณะนี้จึงไม่มีประโยชน์อันใด

## E0030

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งที่ใช้จำกัดหรือควบคุมวิธีการสร้างค่าของ struct ถูกนำไปใช้กับยูนิตสตรักต์ (unit struct)

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(params = "u8")]
struct UnitStruct;
```

ยูนิตสตรักต์มีค่าที่เป็นไปได้เพียงค่าเดียว จึงมีกลยุทธ์ที่เป็นไปได้เพียงหนึ่งเดียวเท่านั้น ด้วยเหตุนี้ การพยายามระบุกลยุทธ์ทางเลือกหรือการกรอง struct ลักษณะนี้จึงไม่มีประโยชน์อันใด

## E0031

ข้อผิดพลาดนี้เกิดขึ้นเมื่อใช้ [`#[proptest(no_bound)]`] กับสิ่งที่ไม่ใช่ตัวแปรชนิดข้อมูล (type variable)

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(no_bound)]
    bar: u32,
}
```

ตัวปรับแต่ง `no_bound` มีความหมายเฉพาะกับตัวแปรชนิดข้อมูลแบบเจเนอริก (generic type variable) ดังเช่นในตัวอย่างนี้:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo<#[proptest(no_bound)] T> {
    #[proptest(value = "None")]
    bar: Option<T>,
}
```

## E0032

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งค่าใดๆ เข้าไปใน [`#[proptest(no_bound)]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo<#[proptest(no_bound = "yes")] T> {
    _bar: PhantomData<T>,
}
```

รูปแบบเดียวที่ถูกต้องของตัวปรับแต่งนี้คือ `#[proptest(no_bound)]`

## E0033

ข้อผิดพลาดนี้เกิดขึ้นเมื่อผลรวมของค่าน้ำหนักบนแวเรียนต์ต่างๆ ใน enum มีค่าเกินขอบเขตของ `u32` (overflow)

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    #[proptest(weight = 3_000_000_000)]
    ThreeFifths,
    #[proptest(weight = 2_000_000_000)]
    TwoFifths,
}
```

วิธีแก้ปัญหาเพียงอย่างเดียวคือการลดขนาดของค่าน้ำหนักลง เพื่อให้ผลรวมพอดีกับขอบเขตของ `u32` โปรดระลึกไว้ว่า แวเรียนต์ที่ไม่ได้ระบุตัวปรับแต่ง `weight` จะมีค่าน้ำหนักเทียบเท่ากับ `#[proptest(weight = 1)]` โดยปริยาย

## E0034

ข้อผิดพลาดนี้เกิดขึ้นเมื่อใช้ [`#[proptest(regex = "string")]`] ด้วยไวยากรณ์ที่ไม่ถูกต้อง

รูปแบบที่พบบ่อยที่สุดคือ `#[proptest(regex = "string-regex")]` และ `#[proptest(regex("string-regex"))]`

## E0035

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้ทั้ง [`#[proptest(regex = "string")]`] และ [`#[proptest(params = "type")]`] บนไอเท็มเดียวกัน

ค่าที่สร้างขึ้นผ่านเรกูลาร์เอ็กซ์เพรสชันไม่รับพารามิเตอร์ใดๆ ดังนั้นตัวปรับแต่ง `params` จึงไม่มีความหมาย

## "ไวยากรณ์ Rust ที่ถูกต้อง"

นิยามของ "ไวยากรณ์ Rust ที่ถูกต้อง" ในตัวปรับแต่งแบบสตริงต่างๆ นั้นถูกกำหนดโดยเครต `syn` หากไวยากรณ์ที่ถูกต้องถูกปฏิเสธ คุณสามารถแก้ปัญหาเฉพาะหน้า (work around) ได้สองสามวิธี ขึ้นอยู่กับว่าไวยากรณ์นั้นกำลังอธิบายอะไรอยู่:

สำหรับชนิดข้อมูล คุณสามารถนิยามนามแฝงชนิดข้อมูล (type alias) ให้กับชนิดข้อมูลดังกล่าวได้โดยตรง ตัวอย่างเช่น:

```rust,compile_fail
type RetroBox = ~str; // N.B. "~str" is not valid Rust 1.30 syntax

//...
#[derive(Debug, Arbitrary)]
#[proptest(params = "RetroBox")]
struct MyStruct { /* ... */ }
```

สำหรับค่า โดยทั่วไปคุณสามารถแยกโค้ดออกมาเป็นค่าคงที่หรือฟังก์ชันได้ ตัวอย่างเช่น:

```rust,compile_fail
// N.B. Rust 1.30 does not have an exponentiation operator.
const PI_SQUARED: f64 = PI ** 2.0;

//...
#[derive(Debug, Arbitrary)]
struct MyStruct {
    #[proptest(value = "PI_SQUARED")]
    factor: f64,
}
```

หากคุณจำเป็นต้องใช้วิธีแก้ปัญหาเฉพาะหน้าลักษณะนี้ โปรดพิจารณา[แจ้งประเด็นปัญหา (issue)](https://github.com/proptest-rs/proptest/issues) ให้เราทราบด้วย

[`#[proptest(filter = "expr")]`]: modifiers.md#filter
[`#[proptest(no_bound)]`]: modifiers.md#no_bound
[`#[proptest(no_params)]`]: modifiers.md#no_params
[`#[proptest(params = "type")]`]: modifiers.md#params
[`#[proptest(regex = "string")]`]: modifiers.md#regex
[`#[proptest(skip)]`]: modifiers.md#skip
[`#[proptest(strategy = "expr")]`]: modifiers.md#strategy
[`#[proptest(value = "expr")]`]: modifiers.md#value
[`#[proptest(weight = <integer>)]`]: modifiers.md#weight
