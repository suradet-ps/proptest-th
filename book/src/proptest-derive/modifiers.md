# เอกสารอ้างอิงตัวปรับแต่ง

ตัวปรับแต่งทั้งหมดที่ `#[derive(Arbitrary)]` ตีความนั้นอยู่ในรูปแบบ `#[proptest(..)]` โดยเนื้อหาภายในวงเล็บจะเป็นไปตามไวยากรณ์แอตทริบิวต์ปกติของ Rust

ตัวปรับแต่งแต่ละตัวภายในวงเล็บนั้นเป็นอิสระต่อกัน กล่าวคือ การใส่ตัวปรับแต่งสองตัวในแอตทริบิวต์เดียวกันเทียบเท่ากับการมีแอตทริบิวต์ `#[proptest(..)]` สองตัวที่มีตัวปรับแต่งตัวละหนึ่งตัว

เพื่อความกระชับ บางครั้งตัวปรับแต่งจะถูกอ้างถึงด้วยชื่อเพียงอย่างเดียว เช่น "ตัวปรับแต่ง `weight`" หมายถึง `#[proptest(weight = nn)]` ไม่ใช่แอตทริบิวต์ `#[weight]` แบบลอยๆ

## `filter`

รูปแบบ: `#[proptest(filter = F)]` หรือ `#[proptest(filter(F))]` โดยที่ `F` เป็นได้ทั้งตัวระบุเดี่ยวๆ (คือตั้งชื่อฟังก์ชัน) หรือนิพจน์ Rust ในสตริง ไม่ว่าแบบไหน พารามิเตอร์ต้องประเมินได้เป็นอะไรก็ตามที่เป็น `Fn (&T) -> bool` โดยที่ `T` คือชนิดข้อมูลของสิ่งที่ถูกกรอง

ใช้ได้กับ: struct, enum, วาเรียนต์ของ enum, ฟิลด์

ตัวปรับแต่ง `filter` ช่วยกรองค่าที่สร้างขึ้นสำหรับฟิลด์ด้วยการสุ่มแบบปฏิเสธ เนื่องจากการสุ่มแบบปฏิเสธนั้นไม่มีประสิทธิภาพและรบกวนการชริงก์ จึงควรใช้เฉพาะกับเงื่อนไขที่พบได้ยากมากหรือไม่สามารถแสดงออกด้วยวิธีอื่นได้เท่านั้น ในหลายกรณี [`strategy`](#strategy) สามารถใช้แสดงพฤติกรรมที่ต้องการได้ตรงกว่าโดยไม่ต้องใช้การสุ่มแบบปฏิเสธ ดูรายละเอียดเพิ่มเติมได้ที่เอกสารประกอบของ [`prop_filter`]

อาร์กิวเมนต์ของตัวปรับแต่งต้องเป็นอาร์กิวเมนต์ที่ถูกต้องสำหรับพารามิเตอร์ตัวที่สองของ [`prop_filter`]

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::prelude::*;

#[derive(Debug, Arbitrary)]
#[proptest(filter = "|segment| segment.start != segment.end")]
struct NonEmptySegment {
    start: i32,
    end: i32,
}
```
เทียบเท่ากับ
```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::prelude::*;

fn is_nonempty(segment: &NonEmptySegment) -> bool {
    segment.start != segment.end
}

#[derive(Debug, Arbitrary)]
#[proptest(filter = "is_nonempty")]
struct NonEmptySegment {
    start: i32,
    end: i32,
}
```

ดังที่กล่าวไว้ข้างต้น ควรหลีกเลี่ยงการกรองเมื่อยังพอเป็นไปได้ที่จะแสดงกลยุทธ์ที่ไม่ใช้การกรองซึ่งให้ผลแบบเดียวกัน ตัวอย่างเช่น:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::{proptest, arbitrary::any, strategy::Strategy};

#[derive(Debug, Arbitrary)]
struct BadExample {
    // Don't do this! Your tests will run more slowly and shrinking won't work
    // properly.
    #[proptest(filter = "|x| x % 2 == 0")]
    even_number: u32,
}

#[derive(Debug, Arbitrary)]
struct GoodExample {
    // Directly generate even numbers only by transforming the set of all
    // `u32`s and then mapping it to the set of even `u32`s.
    #[proptest(strategy = "any::<u32>().prop_map(|x| x / 2 * 2)")]
    even_number: u32,
}
```

[`prop_filter`]: https://docs.rs/proptest/latest/proptest/strategy/trait.Strategy.html#method.prop_filter

## `no_bound`

รูปแบบ: `#[proptest(no_bound)]`

ใช้ได้กับ: นิยามชนิดข้อมูลเจเนอริกและพารามิเตอร์ชนิด

โดยปกติ เมื่อ `#[derive(Arbitrary)]` ถูกนำไปใช้กับไอเท็มที่มีพารามิเตอร์ชนิดเจเนอริก พารามิเตอร์ชนิดทุกตัวที่ "ถูกใช้" (ดูด้านล่าง) จะต้อง `impl Arbitrary` ตัวอย่างเช่น จากการประกาศแบบนี้:

```rust
# extern crate proptest_derive;
# use proptest_derive::Arbitrary;

#[derive(Debug, Arbitrary)]
struct MyStruct<T> {
    # t: T
    /* ... */
}
```

โค้ดประมาณนี้จะถูกสร้างขึ้น:

```rust
# extern crate proptest;
# use proptest::arbitrary::Arbitrary;

# #[derive(Debug)]
# struct MyStruct<T> {
# t: T
}

impl<T> Arbitrary for MyStruct<T> where T: Arbitrary {
    # type Parameters = u32;
    # type Strategy = proptest::strategy::BoxedStrategy<Self>;
    # fn arbitrary_with(_params: Self::Parameters) -> Self::Strategy { todo!() }
    /* ... */
}
```

การวาง `#[proptest(no_bound)]` บนนิยามชนิดข้อมูลเจเนอริกเทียบเท่ากับการวางแอตทริบิวต์เดียวกันบนพารามิเตอร์ชนิดทุกตัว

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::proptest;
# use std::marker::PhantomData;

#[derive(Debug, Arbitrary)]
#[proptest(no_bound)]
struct MyStruct<A, B, C> {
    # a: PhantomData<A>,
    # b: PhantomData<B>,
    # c: PhantomData<C>,
    /* ... */
}
```
นี่เทียบเท่ากับไวยากรณ์สมมติ (แต่ยังไม่รองรับในปัจจุบัน) แบบนี้:
```rust,compile_fail
# extern crate proptest_derive;
# use proptest_derive::Arbitrary;
# use std::marker::PhantomData;

#[derive(Debug, Arbitrary)]
struct MyStruct<
  #[proptest(no_bound)] A,
  #[proptest(no_bound)] B,
  #[proptest(no_bound)] C,
> {
    # a: PhantomData<A>,
    # b: PhantomData<B>,
    # c: PhantomData<C>,
    /* ... */
}
```

พารามิเตอร์ชนิดจะ "ถูกใช้" เมื่อเงื่อนไขต่อไปนี้เป็นจริง:

- นิยาม enum หรือ struct อ้างถึงมันอย่างน้อยหนึ่งครั้ง และการอ้างอิงนั้นไม่ได้อยู่ภายในอาร์กิวเมนต์ชนิดของ `PhantomData`

- ไอเท็มที่อ้างถึงพารามิเตอร์ชนิดนั้นไม่มีตัวปรับแต่งของ proptest ที่แทนที่การใช้งาน `Arbitrary` ตามปกติ เช่น [`skip`](#skip) หรือ [`value`](#value)

ด้วยเหตุข้างต้น โดยทั่วไปแล้ว `#[proptest(no_bound)]` จำเป็นเฉพาะเมื่อพารามิเตอร์ชนิดถูกใช้ในชนิดข้อมูลอื่นซึ่งตัวเองไม่ได้มีบาวด์ `Arbitrary` อยู่บนชนิดนั้น

## `no_params`

รูปแบบ: `#[proptest(no_params)]`

ใช้ได้กับ: struct, enum, วาเรียนต์ของ enum, ฟิลด์

บน struct หรือ enum `no_params` ทำให้ชนิดข้อมูลพารามิเตอร์ของ `Arbitrary` เป็น `()` การมอบหมายงานอัตโนมัติทั้งหมดไปยัง `Arbitrary` บนสมาชิกของไอเท็มจะใช้ `Default::default()` สำหรับพารามิเตอร์ของมัน

บนวาเรียนต์ของ enum หรือฟิลด์ จะระงับการเพิ่มพารามิเตอร์ใดๆ ของวาเรียนต์หรือฟิลด์นั้นเข้าไปในพารามิเตอร์ของ struct ทั้งตัว หากวาเรียนต์หรือฟิลด์นั้นมอบหมายงานไปยัง `Arbitrary` สำหรับค่าของมันโดยอัตโนมัติ การเรียก `Arbitrary` นั้นจะใช้ `Default::default()` สำหรับพารามิเตอร์ของตัวเอง

ดูข้อมูลเพิ่มเติมเกี่ยวกับวิธีการทำงานของพารามิเตอร์ได้ที่[ตัวปรับแต่ง `param`](#param)

## `params`

รูปแบบ: `#[proptest(params = T)]` หรือ `#[proptest(params(T))]` โดยที่ `T` เป็นได้ทั้งตัวระบุเดี่ยวๆ หรือโค้ด Rust ในสตริง ไม่ว่าแบบไหน ค่านั้นต้องตั้งชื่อชนิดข้อมูล Rust ที่เป็นรูปธรรมซึ่งนำเทรต `Default` ไปใช้

ใช้ได้กับ: struct, enum, วาเรียนต์ของ enum, ฟิลด์

[เทรต `Arbitrary`] ระบุชนิดข้อมูล `Parameters` ซึ่งใช้ควบคุมการสร้างค่า โดยค่าเริ่มต้น ชนิดข้อมูล `Parameters` คือทูเพิลของพารามิเตอร์ที่ถูกส่งต่อไปยังการนำเทรต `Arbitrary` อื่นๆ โดยอัตโนมัติ

หากใช้กับ struct หรือ enum `params` จะแทนที่ชนิดข้อมูล `Parameters` ทั้งหมด การมอบหมายงานอัตโนมัติใดๆ ไปยังการนำเทรต `Arbitrary` อื่นๆ จะใช้ `Default::default()` เนื่องจากไม่มีวิธีอัตโนมัติในการหาค่าที่เหมาะสม (ถ้ามี) ภายในชนิดข้อมูล `params`

หากใช้กับวาเรียนต์ของ enum หรือฟิลด์ `params` จะระบุชนิดข้อมูลพารามิเตอร์สำหรับไอเท็มนั้นเพียงอย่างเดียว ราวกับว่าชนิดข้อมูลของมันมีการนำเทรต `Arbitrary` ไปใช้ซึ่งรับชนิดข้อมูลนั้น ในกรณีนี้ _ต้อง_ ระบุ [`value`](#value) หรือ [`strategy`](#strategy) อย่างใดอย่างหนึ่ง เนื่องจากโดยทั่วไปแล้วชนิดข้อมูลพารามิเตอร์จะไม่เข้ากันได้กับการเรียก `Arbitrary` ตามปกติ (และในกรณีที่เข้ากันได้ `params` ก็จะไร้ประโยชน์ถ้าไม่ถูกใช้)

นิพจน์ใดๆ (เช่นในตัวปรับแต่ง [`value`](#value) และ [`strategy`](#strategy)) ที่อยู่ภายใต้ไอเท็มซึ่งมีตัวปรับแต่ง `params` จะสามารถเข้าถึงตัวแปรชื่อ `params` ซึ่งมีชนิดข้อมูลตามที่ส่งใน `#[proptest(params = ..)]`

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::prelude::*;

#[derive(Debug)]
struct WidgetRange(usize, usize);

impl Default for WidgetRange {
    fn default() -> Self { Self(0, 100) }
}

#[derive(Debug, Arbitrary)]
#[proptest(params(WidgetRange))]
struct WidgetCollection {
    #[proptest(strategy = "params.0 ..= params.1")]
    desired_widget_count: usize,
    // ...
}

// ...

proptest! {
    #[test]
    fn test_something(wc in any_with::<WidgetCollection>(WidgetRange(10, 20))) {
        assert!(wc.desired_widget_count >= 10 && wc.desired_widget_count <= 20);
    }
}
```

[เทรต `Arbitrary`]: https://docs.rs/proptest/latest/proptest/arbitrary/trait.Arbitrary.html

## `regex`

รูปแบบ: `#[proptest(regex = "string")]` หรือ `#[proptest(regex("string"))]` โดยที่ `string` เป็นนิพจน์ปรกติ อาจเรียกใช้ในรูป `#[proptest(regex(function_name))]` ได้ด้วย โดยที่ `function_name` เป็นฟังก์ชันที่ไม่รับอาร์กิวเมนต์และคืนค่าเป็น `&'static str`

ใช้ได้กับ: ฟิลด์

ตัวปรับแต่งนี้ระบุให้สร้างสตริงอักขระหรือสตริงไบต์สำหรับฟิลด์ที่ตรงกับนิพจน์ปรกติที่กำหนด

ตัวปรับแต่ง `regex` เทียบเท่ากับการใช้ตัวปรับแต่ง [`strategy`](#strategy) แล้วห่อสตริงด้วย [`string_regex`] หรือ [`bytes_regex`] โดยใช้ได้เฉพาะกับฟิลด์ชนิด `String` หรือ `Vec<u8>`

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::proptest;
#[derive(Debug, Arbitrary)]
struct FileContent {
    #[proptest(regex = "[a-z0-9.]+")]
    name: String,
    #[proptest(regex = "([0-9]+\n)*")]
    content: Vec<u8>,
}
```

[`string_regex`]: https://docs.rs/proptest/latest/proptest/string/fn.string_regex.html
[`bytes_regex`]: https://docs.rs/proptest/latest/proptest/string/fn.bytes_regex.html

## `skip`

รูปแบบ: `#[proptest(skip)]`

ใช้ได้กับ: วาเรียนต์ของ enum

การติดแอตทริบิวต์ `#[proptest(skip)]` ให้วาเรียนต์ของ enum จะป้องกันไม่ให้ proptest สร้างวาเรียนต์นั้น วิธีนี้มีประโยชน์เมื่อไม่มีวิธีที่สมเหตุสมผลในการสร้างวาเรียนต์นั้น หรือเมื่อคุณต้องการหยุดสร้างวาเรียนต์บางตัวชั่วคราวระหว่างพัฒนา

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::prelude::*;

#[derive(Debug, Arbitrary)]
enum DataSource {
    Memory(Vec<u8>),

    // There's no way to produce an "arbitrary" file handle, so we skip
    // generating this case.
    #[proptest(skip)]
    File(std::fs::File),
}
```

การติดแอตทริบิวต์ `#[proptest(skip)]` ให้วาเรียนต์ที่มีค่าอยู่ทุกตัวของ enum ถือเป็นข้อผิดพลาด เพราะจะทำให้ proptest ไม่มีทางเลือกใดๆ ในการสร้าง enum นั้นเลย

## `strategy`

รูปแบบ: `#[proptest(strategy = S)]` หรือ `#[proptest(strategy = S)]` โดยที่ `S` เป็นได้ทั้งสตริงที่บรรจุนิพจน์ Rust ซึ่งประเมินได้เป็น `Strategy` ที่เหมาะสม หรือตัวระบุเดี่ยวๆ ที่ตั้งชื่อฟังก์ชันซึ่งเมื่อเรียกโดยไม่มีอาร์กิวเมนต์จะคืนค่า `Strategy` ดังกล่าว

ใช้ได้กับ: วาเรียนต์ของ enum, ฟิลด์

โดยค่าเริ่มต้น วาเรียนต์ของ enum จะถูกสร้างโดยการเรียกซ้ำเข้าไปในนิยามของมัน เช่นเดียวกับการประกาศ struct และฟิลด์จะถูกสร้างโดยเรียกใช้ `Arbitrary` บนชนิดข้อมูลของฟิลด์เพื่อสร้าง `Strategy` ตัวปรับแต่ง `strategy` ช่วยให้ระบุกลยุทธ์แบบกำหนดเองโดยตรงด้วยมือได้

ในกรณีของฟิลด์ กลยุทธ์ต้องสร้างค่าที่มีชนิดข้อมูลเดียวกับฟิลด์นั้น ส่วนวาเรียนต์ของ enum กลยุทธ์ต้องสร้างค่าของชนิดข้อมูล enum นั้นเอง และค่าเหล่านั้นควรเป็นของวาเรียนต์ที่กำลังพูดถึง

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::prelude::*;
# use proptest::strategy::Strategy;

#[derive(Debug, Arbitrary)]
enum Token {
    Delimitation {
        // This field is still generated via Arbitrary
        delimiter: Delimiter,

        // But for this field we use a custom strategy
        #[proptest(strategy = "1..(10 as u32)")]
        count: u32,

        // Here we also use a custom strategy, generated by the function
        // `offset_strategy`.
        #[proptest(strategy = "offset_strategy()")]
        offset: u32,
    },

    // Specify how to generate the whole enum variant
    #[proptest(strategy = "\"[a-zA-Z]+\".prop_map(Token::Word)")]
    Word(String),
}

#[derive(Debug, Arbitrary)]
enum Delimiter {
    # Nope
    /* ... */
 }

fn offset_strategy() -> impl Strategy<Value = u32> {
  0..(100 as u32)
}
```

## `value`

รูปแบบ: `#[proptest(value = V)]` หรือ `#[proptest(value(V))]` โดยที่ V เป็นได้ดังนี้: (a) นิพจน์ Rust ที่อยู่ในสตริง; (b) ตัวอักษรค่า (literal) อื่นๆ หรือ (c) ตัวระบุเดี่ยวๆ ที่ตั้งชื่อฟังก์ชันซึ่งไม่รับอาร์กิวเมนต์

ใช้ได้กับ: วาเรียนต์ของ enum, ฟิลด์

ตัวปรับแต่ง `value` บ่งบอกว่า proptest ควรใช้นิพจน์หรือฟังก์ชันที่กำหนดเพื่อสร้างค่าสำหรับฟิลด์ แทนที่จะผ่านกลไกการสร้างค่าตามปกติ

อาร์กิวเมนต์ของ `value` จะถูกใช้เป็นนิพจน์สำหรับค่าฟิลด์หรือวาเรียนต์ของ enum ที่จะสร้างโดยตรง ยกเว้นในรูปแบบที่สามซึ่งเป็นตัวระบุเดี่ยวๆ มันจะถูกเรียกเป็นฟังก์ชันที่ไม่รับอาร์กิวเมนต์เพื่อสร้างค่านั้น

การใช้ `value` เทียบเท่ากับการใช้ [`strategy`](#strategy) แล้วห่อค่าด้วย `LazyJust`

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::prelude::*;
# use std::time::Instant;

#[derive(Debug, Arbitrary)]
struct EventCounter {
    // We always start with the first two fields set to 0/None
    #[proptest(value = 0)]
    number_seen: u64,

    #[proptest(value = "None")]
    last_seen_time: Option<Instant>,

    // This field is generated normally
    max_events: u64,
}
```

## `weight`

รูปแบบ: `#[proptest(weight = W)]` หรือ `#[proptest(weight(W))]` โดยที่ `W` เป็นนิพจน์ที่ประเมินได้เป็น `u32` นอกจากนี้ `weight` ยังเขียนย่อเป็น `w` ได้ด้วย เช่น `#[proptest(w = W)]`

ใช้ได้กับ: วาเรียนต์ของ enum

ตัวปรับแต่ง `weight` กำหนดว่า proptest จะสร้างวาเรียนต์ของ enum ตัวหนึ่งๆ บ่อยเพียงใด น้ำหนักเป็นค่าเปรียบเทียบกัน เช่น วาเรียนต์ที่มี `weight = 3` มีโอกาสถูกสร้างมากกว่าวาเรียนต์ที่มี `weight = 2` อยู่ 50% และมีโอกาสถูกสร้างเป็นสามเท่าของวาเรียนต์ที่มี `weight = 1`

วาเรียนต์ที่ไม่มีตัวปรับแต่ง `weight` เทียบเท่ากับการติดแอตทริบิวต์ `#[proptest(weight = 1)]`

ตัวอย่าง:

```rust
# extern crate proptest_derive;
# extern crate proptest;
# use proptest_derive::Arbitrary;
# use proptest::proptest;
#[derive(Debug, Arbitrary)]
enum FilterOption {
    KeepAll,
    DiscardAll,

    // This option is presumably harder for the code to handle correctly,
    // so we generate it more frequently than the other options.
    #[proptest(weight = 3)]
    OnlyMatching(String),
}
```
