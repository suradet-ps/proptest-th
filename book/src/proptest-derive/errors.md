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

ยังไม่สามารถนิยาม `Strategy` ที่สร้างชนิดข้อมูลซึ่งเป็นเจเนอริกด้านไลฟ์ไทม์ได้ (เช่น `&'a T`) ดังนั้น proptest จึงไม่สามารถนำเทรต `Arbitrary` ไปใช้กับชนิดข้อมูลแบบนั้นได้เช่นกัน และด้วยเหตุนี้คุณจึงไม่สามารถ `#[derive(Arbitrary)]` ให้ชนิดข้อมูลแบบนั้นได้ ปัจจุบัน GAT ใช้งานได้บน stable Rust ตั้งแต่เวอร์ชัน 1.65 และเราจะกลับมาพิจารณาว่าจะรองรับเรื่องนี้อย่างไร ติดตามความคืบหน้าได้ที่[ประเด็นที่กำลังติดตาม][issue#9] เกี่ยวกับเรื่องนี้

## E0002

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับชนิดข้อมูล `union` ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
union IU32 {
    signed: i32,
    unsigned: u32,
}
```

สาเหตุหลักของข้อผิดพลาดนี้มีสองประการ

1. ไม่สามารถ `#[derive(Debug)]` บนชนิดข้อมูล `union` ได้ และการนำไปใช้ด้วยมือก็ไม่สามารถรู้ได้ว่าวาเรียนต์ใดถูกต้อง จึงมีการนำไปใช้ที่ถูกต้องให้เลือกไม่มากนัก

2. ประการที่สอง เราไม่สามารถบอกได้โดยอัตโนมัติว่าควรสร้างวาเรียนต์ใดระหว่าง `signed` กับ `unsigned` แม้เราจะเปิดทางให้คุณบอกมาโครได้ผ่านแอตทริบิวต์อย่าง `#[proptest(select)]` บนวาเรียนต์ แต่เราก็เลือกแนวทางที่ระมัดระวังกว่าในตอนนี้ หากคุณมีกรณีการใช้งาน `#[derive(Arbitrary)]` กับชนิดข้อมูล `union` โปรดแจ้งให้เราทราบผ่าน[ตัวติดตาม issue]

## E0003

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ struct ที่มี[ชนิดข้อมูลที่ไม่มีค่าใดๆ](https://doc.rust-lang.org/nomicon/exotic-sizes.html#empty-types)อยู่ด้วย ซึ่งหมายความว่า struct นั้นเองก็ไม่มีค่าใดๆ และด้วยเหตุนี้จึงไม่มีการนำเทรต `Arbitrary` ไปใช้ที่สมเหตุสมผล เพราะไม่สามารถสร้างค่าของ struct นั้นได้

ตัวอย่างง่ายๆ:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Uninhabited {
    inhabited: u32,
    never: !,
}
```

เนื่องจากไม่มีค่าใดที่กำหนดให้ฟิลด์ `never` ได้ จึงเป็นไปไม่ได้เช่นกันที่จะสร้างอินสแตนซ์ของ struct `Uninhabited`

ความสามารถของ Proptest ในการระบุชนิดข้อมูลที่ไม่มีค่าใดๆ นั้นมีจำกัด หากมันไม่รู้จักชนิดข้อมูลหนึ่งว่ามีค่าไม่ได้ ชนิดข้อมูลนั้นจะถูกสันนิษฐานว่ามีค่าอยู่แทน และคุณจะได้ข้อผิดพลาดว่าชนิดข้อมูลนั้นไม่ได้นำเทรต `Arbitrary` ไปใช้แทน

## E0004

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ enum ที่ไม่มีวาเรียนต์เลย ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Uninhabited {}
```

enum แบบนี้ไม่มีค่าใดๆ เลย จึงไม่สมเหตุสมผลที่จะมีการนำเทรต `Arbitrary` ไปใช้ให้ เพราะไม่สามารถสร้างค่าใดๆ ได้

## E0005

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ enum ที่วาเรียนต์ทั้งหมดไม่มีค่าใดๆ โดยใช้ตรรกะเดียวกับที่อธิบายไว้ใน [`E0003`](#e0003) ผลก็คือ enum นั้นไม่มีค่าใดๆ ทั้งสิ้น

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Uninhabited {
    Never(!),
    NeverEver(!, !),
}
```

## E0006

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ `#[derive(Arbitrary)]` ถูกใช้กับ enum ที่วาเรียนต์ซึ่งมีค่าอยู่ทุกตัวถูกทำเครื่องหมายด้วย [`#[proptest(skip)]`] กล่าวอีกนัยหนึ่งคือ proptest ถูกห้ามไม่ให้สร้างวาเรียนต์ใดๆ ของ enum นั้น ดังนั้นจึงไม่สามารถสร้าง enum นั้นได้

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

ข้อผิดพลาดนี้เกิดขึ้นเมื่อแอตทริบิวต์ [`#[proptest(strategy = "expr")]`] หรือ [`#[proptest(value = "expr")]`] ถูกใช้กับไอเท็มเดียวกันที่มี `#[derive(Arbitrary)]` อยู่

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(value = "MyStruct(42)")]
struct MyStruct(u32);
```

กรณีนี้ถูกปฏิเสธเพราะไม่มีอะไรถูก "derive" จริงๆ ในความหมายตรงตัว ควรใช้การนำเทรต `Arbitrary` ไปใช้ที่เขียนออกมาเองแทน

## E0008

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(skip)]`] ถูกใช้กับไอเท็มที่ข้ามไม่ได้ ตัวอย่างเช่น ฟิลด์ของ struct ข้ามไม่ได้ เพราะ Rust กำหนดให้ทุกฟิลด์ของ struct ต้องมีค่า

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct WidgetContainer {
    desired_widget_count: usize,
    #[proptest(skip)]
    widgets: Vec<Widget>,
}
```

โดยทั่วไป วิธีที่เหมาะสมในการขอให้ proptest ไม่สร้างค่าของฟิลด์คือใช้ [`#[proptest(value = "expr")]`] เพื่อกำหนดค่าคงที่ด้วยตัวเอง ตัวอย่างเช่น โค้ดข้างบนสามารถเขียนอย่างถูกต้องได้ดังนี้:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct WidgetContainer {
    desired_widget_count: usize,
    #[proptest(value = "vec![]")] // Always generate an empty widget vec
    widgets: Vec<Widget>,
}
```

## E0009

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(weight = <integer>)]`] ถูกใช้กับไอเท็มที่การทำเช่นนั้นไม่สมเหตุสมผล เช่นฟิลด์ของ struct ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Point {
    x: u32,
    #[proptest(weight = 42)]
    y: u32,
}
```

แอตทริบิวต์ `weight` จะสมเหตุสมผลเฉพาะเมื่อ proptest มีตัวเลือกท่ามกลางหลายไอเท็ม นั่นคือวาเรียนต์ของ enum ในทางตรงกันข้าม กับฟิลด์ของ struct proptest ต้องให้ค่าสำหรับ_ทุก_ฟิลด์ จึงไม่มีการเลือกแบบ "อันนี้หรืออันนั้น"

## E0010

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(params = "type")]`] และ/หรือ [`#[proptest(no_params)]`] ถูกตั้งค่าทั้งบนไอเท็มและบนไอเท็มแม่ของมัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(params = "String")]
struct Foo {
    #[proptest(no_params)]
    bar: String,
}
```

หากไอเท็มแม่มีการกำหนดพารามิเตอร์อย่างชัดเจนใดๆ มันจะนิยามพารามิเตอร์ทั้งหมดสำหรับการนำเทรต `Arbitrary` ไปใช้ทั้งชุด และไอเท็มลูกจะต้องทำงานร่วมกับค่านั้นและระบุพารามิเตอร์ของตัวเองไม่ได้

## E0011

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(params = "type")]`] ถูกตั้งค่าบนฟิลด์ แต่ไม่มีการกำหนดกลยุทธ์อย่างชัดเจนด้วย [`#[proptest(strategy = "expr")]`] หรือตัวปรับแต่งอื่นในทำนองเดียวกัน ตัวอย่างเช่น:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(param = "u8")]
    some_string: String,
}
```

ตัวอย่างนี้แสดงให้เห็นว่าทำไมต้องระบุทั้งคู่: การนำเทรต `Arbitrary` ไปใช้ของ `String` รับ `proptest::string::StringParam` แต่ตรงนี้เราพยายามส่ง `u8` ให้มัน

แม้โค้ดที่ถูกสร้างขึ้นจะทำงานได้หากชนิดข้อมูลที่ให้ใน `param` เหมือนกับชนิดข้อมูลของกลยุทธ์เริ่มต้น แต่การระบุชนิดข้อมูลพารามิเตอร์ด้วยมือก็จะไม่มีจุดประสงค์อะไร ดังนั้นการระบุเพียง `param` จึงถูกห้ามในทุกกรณี

## E0012

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(filter = "expr")]`] ถูกตั้งค่าบนไอเท็ม แต่ไอเท็มที่บรรจุมันระบุวิธีสร้างค่าทั้งหมดโดยตรง ซึ่งจะเกิดขึ้นโดยไม่ปรึกษาตัวกรองเลย

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

ในตัวอย่างนี้ วาเรียนต์ `Bar` ทั้งตัวระบุวิธีสร้างตัวเองขึ้นมาทั้งหมด ด้วยเหตุนี้ อนุประโยค `filter` บน `even_number` จึงไม่มีโอกาสได้ทำงาน

## E0013

ข้อผิดพลาดนี้จะเกิดขึ้นหากแอตทริบิวต์ภายนอกในรูปแบบ `#![proptest(..)]` ถูกใช้กับอะไรก็ตามที่อยู่ภายใต้ `#[derive(Arbitrary)]`

ตั้งแต่ Rust 1.30.0 เป็นต้นมา ไม่มีวิธีที่รู้จักใดๆ ที่ทำให้เกิดข้อผิดพลาดนี้ เพราะคอมไพเลอร์ Rust จะปฏิเสธแอตทริบิวต์นั้นก่อนแล้ว

## E0014

ข้อผิดพลาดนี้เกิดขึ้นเมื่อแอตทริบิวต์ `#[proptest]` แบบเปล่าๆ ถูกใช้กับอะไรก็ตาม เนื่องจากมันไม่มีเนื้อหาที่มีความหมาย

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest]
    field: u8,
}
```

การใช้แอตทริบิวต์นี้อย่างถูกต้องมีเพียงรูปแบบ `#[proptest(..)]` เท่านั้น

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

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งค่าคงที่ (literal) (ซึ่งต่างจาก `key = value`) เข้าไปใน `#[proptest(..)]` ในบริบทใดๆ

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(1234)]
    field: u8,
}
```

## E0017

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งใดๆ ของ `#[proptest(..)]` ถูกตั้งค่ามากกว่าหนึ่งครั้งบนไอเท็มเดียวกัน

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

โปรดดู[เอกสารอ้างอิงตัวปรับแต่ง](modifiers.md)เพื่อดูว่ามีตัวปรับแต่งใดบ้าง

## E0019

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งอะไรเพิ่มเติมเข้าไปใน [`#[proptest(no_params)]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(no_params = "true")]
struct Foo(u32);
```

`no_params` ไม่รับการกำหนดค่าใดๆ รูปแบบที่ถูกต้องคือ `#[proptest(no_params)]` ตรงๆ

## E0020

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการส่งอะไรเพิ่มเติมเข้าไปใน [`#[proptest(skip)]`]

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    Small,
    #[proptest(skip = "yes")]
    Huge(ExpensiveType),
}
```

`skip` ไม่รับการกำหนดค่าใดๆ รูปแบบที่ถูกต้องคือ `#[proptest(skip)]` ตรงๆ

## E0021

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(weight = <integer>)]`] ได้รับจำนวนเต็มที่ไม่ถูกต้อง หรือไม่ได้รับอะไรเลย

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

รูปแบบเดียวที่ยอมรับได้คือ `#[proptest(weight = <integer>)]` โดยที่ `<integer>` เป็นได้ทั้งจำนวนเต็มแบบ literal ที่พอดีกับ `u32` หรือค่าแบบเดียวกันที่อยู่ในเครื่องหมายคำพูด

## E0022

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้ [`#[proptest(no_params)]`] และ [`#[proptest(params = "type")]`] มากกว่าหนึ่งอย่างกับไอเท็มเดียวกัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(no_params, params = "u8")]
struct Foo(u32);
```

ต้องเลือกแอตทริบิวต์อย่างใดอย่างหนึ่งตามผลที่ต้องการ

## E0023

ข้อผิดพลาดนี้เกิดขึ้นเมื่อแอตทริบิวต์ [`#[proptest(params = "type")]`] ที่ไม่ถูกต้องถูกใช้กับไอเท็ม

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(params = "Vec<u8")] // Note missing '>'
struct Foo(u32);
```

มีหลายวิธีที่ทำให้เกิดข้อผิดพลาดนี้:

- ไม่ส่งอะไรเลย เช่น `#[proptest(params)]`

- ส่งค่าอย่างอื่นที่ไม่ใช่สตริง เช่น `#[proptest(params = 42)]`

- ส่งชนิดข้อมูลที่ผิดรูปแบบในสตริง ดังในตัวอย่างข้างบน (ดูเพิ่มเติมที่[ข้อควรระวังเกี่ยวกับไวยากรณ์](#ไวยากรณ-rust-ทีถูกตอง))

## E0024

ข้อผิดพลาดนี้เกิดขึ้นเมื่อแอตทริบิวต์ `#[proptest ..]` ที่ไม่ถูกต้องถูกใช้ด้วยไวยากรณ์ที่ครีต `proptest-derive` ไม่พร้อมจะจัดการ

เงื่อนไขที่ทำให้เกิดข้อผิดพลาดนี้เปลี่ยนแปลงไปตามเวอร์ชันของ Rust

## E0025

ข้อผิดพลาดนี้เกิดขึ้นเมื่อมีการใช้มากกว่าหนึ่งอย่างจาก [`#[proptest(strategy = "expr")]`], [`#[proptest(value = "expr")]`] หรือ [`#[proptest(regex = "string")]`] กับไอเท็มเดียวกัน

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(value = "42", strategy = "Just(56)")]
    bar: u32,
}
```

ตัวปรับแต่งแต่ละตัวเหล่านี้อธิบายวิธีสร้างค่าอย่างสมบูรณ์ จึงใช้ทั้งคู่กับสิ่งเดียวกันไม่ได้ ต้องเลือกอย่างใดอย่างหนึ่งตามผลที่ต้องการ

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

มีหลายวิธีที่ทำให้เกิดข้อผิดพลาดนี้:

- ไม่ส่งอะไรเลย เช่น `#[proptest(value)]`

- ใช้รูปแบบอื่นที่ไม่อนุญาต เช่น `#[proptest(value("a", "b"))]`

- ส่งนิพจน์ในสตริงที่ไม่ใช่ไวยากรณ์ Rust ที่ถูกต้อง ดังในตัวอย่างข้างบน (ดูเพิ่มเติมที่[ข้อควรระวังเกี่ยวกับไวยากรณ์](#ไวยากรณ-rust-ทีถูกตอง))

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

มีหลายวิธีที่ทำให้เกิดข้อผิดพลาดนี้:

- ไม่ส่งอะไรเลย เช่น `#[proptest(filter)]`

- ใช้รูปแบบอื่นที่ไม่อนุญาต เช่น `#[proptest(filter("a", "b"))]`

- ส่งนิพจน์ในสตริงที่ไม่ใช่ไวยากรณ์ Rust ที่ถูกต้อง ดังในตัวอย่างข้างบน (ดูเพิ่มเติมที่[ข้อควรระวังเกี่ยวกับไวยากรณ์](#ไวยากรณ-rust-ทีถูกตอง))

## E0028

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งที่บ่งบอกว่าจะต้องมีการสร้างค่าถูกใช้กับวาเรียนต์ของ enum ซึ่งถูกทำเครื่องหมาย [`#[proptest(skip)]`] ไว้ด้วย

ตัวอย่าง:

```rust,compile_fail

#[derive(Debug, Arbitrary)]
enum Enum {
    V1(u32),
    #[proptest(skip, value = "Enum::V2(42)")]
    V2(u32),
}
```

ในที่นี้ ตัวปรับแต่ง [`#[proptest(value = "expr")]`] บ่งบอกว่าผู้ใช้ตั้งใจจะให้สร้างค่าบางอย่างสำหรับวาเรียนต์ของ enum แต่ในขณะเดียวกัน [`#[proptest(skip)]`] กลับบ่งบอกว่าไม่ให้สร้างวาเรียนต์นั้น

## E0029

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งที่จำกัดหรือควบคุมวิธีสร้างค่าของวาเรียนต์ของ enum ถูกใช้กับวาเรียนต์แบบยูนิต

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
enum Foo {
    #[proptest(value = "Foo::V1")]
    UnitVariant,
    // ...
}
```

วาเรียนต์แบบยูนิตมีค่าเดียวที่เป็นไปได้ จึงมีกลยุทธ์เพียงตัวเดียวเท่านั้น ดังนั้นการพยายามระบุกลยุทธ์ทางเลือกหรือกรองวาเรียนต์แบบนี้จึงไม่มีประโยชน์

## E0030

ข้อผิดพลาดนี้เกิดขึ้นเมื่อตัวปรับแต่งที่จำกัดหรือควบคุมวิธีสร้างค่าของ struct ถูกใช้กับ struct แบบยูนิต

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
#[proptest(params = "u8")]
struct UnitStruct;
```

struct แบบยูนิตมีค่าเดียวที่เป็นไปได้ จึงมีกลยุทธ์เพียงตัวเดียวเท่านั้น ดังนั้นการพยายามระบุกลยุทธ์ทางเลือกหรือกรอง struct แบบนี้จึงไม่มีประโยชน์

## E0031

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(no_bound)]`] ถูกใช้กับสิ่งที่ไม่ใช่ตัวแปรชนิด

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo {
    #[proptest(no_bound)]
    bar: u32,
}
```

ตัวปรับแต่ง `no_bound` สมเหตุสมผลเฉพาะบนตัวแปรชนิดเจเนอริก ดังนี้

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo<#[proptest(no_bound)] T> {
    #[proptest(value = "None")]
    bar: Option<T>,
}
```

## E0032

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(no_bound)]`] ได้รับอะไรก็ตามเพิ่มเติม

ตัวอย่าง:

```rust,compile_fail
#[derive(Debug, Arbitrary)]
struct Foo<#[proptest(no_bound = "yes")] T> {
    _bar: PhantomData<T>,
}
```

รูปแบบเดียวที่ถูกต้องของตัวปรับแต่งนี้คือ `#[proptest(no_bound)]`

## E0033

ข้อผิดพลาดนี้เกิดขึ้นเมื่อผลรวมของน้ำหนักบนวาเรียนต์ของ enum เกินขอบเขตของ `u32`

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

ทางแก้เดียวคือลดขนาดของน้ำหนักลงเพื่อให้ผลรวมพอดีกับ `u32` โปรดจำไว้ว่าวาเรียนต์ที่ไม่มีตัวปรับแต่ง `weight` ก็ยังมี `#[proptest(weight = 1)]` โดยผล

## E0034

ข้อผิดพลาดนี้เกิดขึ้นเมื่อ [`#[proptest(regex = "string")]`] ถูกใช้ด้วยไวยากรณ์ที่ไม่ถูกต้อง

รูปแบบที่พบบ่อยที่สุดคือ `#[proptest(regex = "string-regex")]` และ `#[proptest(regex("string-regex"))]`

## E0035

ข้อผิดพลาดนี้เกิดขึ้นเมื่อทั้ง [`#[proptest(regex = "string")]`] และ [`#[proptest(params = "type")]`] ถูกใช้กับไอเท็มเดียวกัน

ค่าที่สร้างผ่านนิพจน์ปรกติไม่รับพารามิเตอร์ใดๆ ดังนั้นตัวปรับแต่ง `params` จึงไร้ความหมาย

## "ไวยากรณ์ Rust ที่ถูกต้อง"

นิยามของ "ไวยากรณ์ Rust ที่ถูกต้อง" ในตัวปรับแต่งแบบสตริงต่างๆ นั้นถูกกำหนดโดยครีต `syn` หากไวยากรณ์ที่ถูกต้องถูกปฏิเสธ คุณสามารถหลบหลีกได้สองสามวิธี ขึ้นอยู่กับว่าไวยากรณ์นั้นอธิบายอะไรอยู่:

สำหรับชนิดข้อมูล เพียงนิยามนามแฝงชนิด (type alias) ให้กับชนิดข้อมูลนั้น ตัวอย่างเช่น

```rust,compile_fail
type RetroBox = ~str; // N.B. "~str" is not valid Rust 1.30 syntax

//...
#[derive(Debug, Arbitrary)]
#[proptest(params = "RetroBox")]
struct MyStruct { /* ... */ }
```

สำหรับค่า โดยทั่วไปคุณสามารถแยกโค้ดออกมาเป็นค่าคงที่หรือฟังก์ชันได้ ตัวอย่างเช่น

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

หากคุณจำเป็นต้องใช้วิธีหลบหลีกแบบนี้ โปรดพิจารณา[แจ้งปัญหา](https://github.com/proptest-rs/proptest/issues)ด้วย

[`#[proptest(filter = "expr")]`]: modifiers.md#filter
[`#[proptest(no_bound)]`]: modifiers.md#no_bound
[`#[proptest(no_params)]`]: modifiers.md#no_params
[`#[proptest(params = "type")]`]: modifiers.md#params
[`#[proptest(regex = "string")]`]: modifiers.md#regex
[`#[proptest(skip)]`]: modifiers.md#skip
[`#[proptest(strategy = "expr")]`]: modifiers.md#strategy
[`#[proptest(value = "expr")]`]: modifiers.md#value
[`#[proptest(weight = <integer>)]`]: modifiers.md#weight
