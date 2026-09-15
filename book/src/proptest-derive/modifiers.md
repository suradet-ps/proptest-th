# เอกสารอ้างอิงตัวปรับแต่ง

ตัวปรับแต่ง (Modifiers) ทั้งหมดที่ `#[derive(Arbitrary)]` รองรับจะเขียนอยู่ในรูปแบบ `#[proptest(..)]` โดยข้อความภายในวงเล็บจะเป็นไปตามไวยากรณ์แอตทริบิวต์มาตรฐานของภาษา Rust

ตัวปรับแต่งแต่ละตัวภายในวงเล็บทำงานเป็นอิสระต่อกัน กล่าวคือ การใส่สองตัวปรับแต่งไว้ในแอตทริบิวต์เดียวกัน ย่อมให้ผลเทียบเท่ากับการเขียนแยกเป็นแอตทริบิวต์ `#[proptest(..)]` สองอันที่มีตัวปรับแต่งอันละตัว

เพื่อความกระชับ ในเอกสารนี้บางครั้งจะเรียกตัวปรับแต่งด้วยชื่อเพียงอย่างเดียว เช่น "ตัวปรับแต่ง `weight`" ซึ่งหมายถึง `#[proptest(weight = nn)]` ไม่ใช่แอตทริบิวต์ `#[weight]` เดี่ยวๆ

## `filter`

รูปแบบ: `#[proptest(filter = F)]` หรือ `#[proptest(filter(F))]` โดยที่ `F` สามารถเป็นได้ทั้งชื่อฟังก์ชัน หรือสตริงที่บรรจุนิพจน์ Rust ซึ่งไม่ว่าจะเขียนแบบใด ค่าดังกล่าวต้องประเมินได้เป็นฟังก์ชันหรือโคลเชอร์ที่ตรงตาม `Fn (&T) -> bool` (โดย `T` คือชนิดข้อมูลของสิ่งที่ถูกกรอง)

ใช้ได้กับ: struct, enum, วาเรียนต์ของ enum, ฟิลด์

ตัวปรับแต่ง `filter` ช่วยกรองค่าที่สร้างขึ้นมาด้วยกลไกการสุ่มแบบปฏิเสธ (rejection sampling) และเนื่องจากการสุ่มแบบปฏิเสธนั้นไม่มีประสิทธิภาพและรบกวนกระบวนการย่อขนาด (shrinking) จึงควรใช้เฉพาะกับเงื่อนไขที่เกิดได้ยากมาก หรือไม่สามารถเขียนนิยามด้วยวิธีอื่นได้เท่านั้น ในหลายกรณี การใช้ตัวปรับแต่ง [`strategy`](#strategy) จะช่วยกำหนดพฤติกรรมที่ต้องการได้ตรงเป้าหมายและมีประสิทธิภาพสูงกว่าโดยไม่ต้องพึ่งพาการสุ่มแบบปฏิเสธ ดูรายละเอียดเพิ่มเติมได้ที่เอกสารประกอบของ [`prop_filter`]

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

ดังที่ได้กล่าวไปแล้วข้างต้น ควรหลีกเลี่ยงการใช้ตัวกรองหากยังสามารถเขียนกลยุทธ์ที่ไม่ต้องกรองแต่ให้ผลลัพธ์เหมือนกันได้ ตัวอย่างเช่น:

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

โดยปกติ เมื่อนำ `#[derive(Arbitrary)]` ไปใช้กับโครงสร้างข้อมูลที่มีพารามิเตอร์ชนิดเจเนอริก (Generic Type Parameters) พารามิเตอร์ชนิดทุกตัวที่ "ถูกใช้งานจริง" (ดูเงื่อนไขด้านล่าง) จะถูกบังคับให้ต้องมี trait bound เป็น `T: Arbitrary` ตัวอย่างเช่น จากการประกาศโค้ดดังนี้:

```rust
# extern crate proptest_derive;
# use proptest_derive::Arbitrary;

#[derive(Debug, Arbitrary)]
struct MyStruct<T> {
    # t: T
    /* ... */
}
```

โค้ดที่มาโครสร้างขึ้นจะมีลักษณะประมาณนี้:

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

การใส่ `#[proptest(no_bound)]` บนระดับนิยามของชนิดข้อมูลเจเนอริก เทียบเท่ากับการใส่แอตทริบิวต์ดังกล่าวให้กับพารามิเตอร์ชนิดทุกตัวพร้อมกัน:

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
เทียบเท่ากับไวยากรณ์สมมติ (ซึ่งในปัจจุบันยังไม่รองรับการเขียนแบบนี้ตรงๆ):
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

พารามิเตอร์ชนิดเจเนอริกจะถือว่า "ถูกใช้งานจริง" ก็ต่อเมื่อตรงตามเงื่อนไขต่อไปนี้ทั้งหมด:

- มีการอ้างอิงถึงชนิดข้อมูลนั้นอย่างน้อยหนึ่งครั้งในนิยาม struct หรือ enum และการอ้างอิงนั้นไม่ได้อยู่ภายในอาร์กิวเมนต์ชนิดข้อมูลของ `PhantomData`

- สมาชิกหรือฟิลด์ที่อ้างอิงถึงพารามิเตอร์ชนิดนั้น ไม่ได้มีตัวปรับแต่งของ proptest ที่มาแทนที่การทำงานของ `Arbitrary` ตามปกติ เช่น [`skip`](#skip) หรือ [`value`](#value)

จากเงื่อนไขข้างต้น โดยทั่วไปแล้ว `#[proptest(no_bound)]` จึงจำเป็นเฉพาะเมื่อพารามิเตอร์ชนิดนั้นถูกนำไปใช้ภายในชนิดข้อมูลอื่นที่ตัวมันเองไม่ได้มี trait bound `Arbitrary` กำกับอยู่

## `no_params`

รูปแบบ: `#[proptest(no_params)]`

ใช้ได้กับ: struct, enum, วาเรียนต์ของ enum, ฟิลด์

บน struct หรือ enum ตัวปรับแต่ง `no_params` จะบังคับให้ชนิดข้อมูลพารามิเตอร์ของ `Arbitrary` เป็น unit type `()` โดยการเรียกส่งต่อไปยัง `Arbitrary` ของฟิลด์ย่อยทั้งหมดโดยอัตโนมัติจะใช้ค่า `Default::default()` แทน

บนวาเรียนต์ของ enum หรือฟิลด์ จะระงับไม่ให้นำพารามิเตอร์ของวาเรียนต์หรือฟิลด์นั้นไปรวมกับพารามิเตอร์ของ struct ตัวนอกสุด หากวาเรียนต์หรือฟิลด์นั้นมีการเรียกส่งต่อไปยัง `Arbitrary` เพื่อสร้างค่า การเรียกนั้นจะใช้ค่า `Default::default()` สำหรับพารามิเตอร์ของตัวเอง

ดูข้อมูลเพิ่มเติมเกี่ยวกับวิธีการทำงานของพารามิเตอร์ได้ที่ [ตัวปรับแต่ง `params`](#params)

## `params`

รูปแบบ: `#[proptest(params = T)]` หรือ `#[proptest(params(T))]` โดยที่ `T` สามารถเป็นได้ทั้งชื่อชนิดข้อมูล หรือโค้ด Rust ในสตริง ซึ่งไม่ว่าจะเขียนแบบใด ค่านั้นต้องเป็นชื่อชนิดข้อมูลที่เป็นรูปธรรม (concrete type) ในภาษา Rust ที่อิมพลีเมนต์เทรต `Default`

ใช้ได้กับ: struct, enum, วาเรียนต์ของ enum, ฟิลด์

[เทรต `Arbitrary`] กำหนดให้มี associated type ชื่อ `Parameters` ซึ่งใช้สำหรับส่งค่าพารามิเตอร์ไปควบคุมการสุ่มสร้างข้อมูล โดยค่าเริ่มต้น ชนิดข้อมูล `Parameters` จะเป็นทูเพิลของพารามิเตอร์ที่ถูกส่งต่อไปยังการอิมพลีเมนต์ `Arbitrary` อื่นๆ โดยอัตโนมัติ

หากใช้กับ struct หรือ enum ตัวปรับแต่ง `params` จะเข้ามาแทนที่ชนิดข้อมูล `Parameters` ทั้งหมด โดยการส่งต่อไปยังการอิมพลีเมนต์ `Arbitrary` อื่นๆ ภายในจะใช้ค่า `Default::default()` เนื่องจากระบบไม่สามารถแยกแยะได้โดยอัตโนมัติว่าควรดึงค่าใดจาก struct พารามิเตอร์ใหม่ไปส่งต่อ

หากใช้กับวาเรียนต์ของ enum หรือฟิลด์ ตัวปรับแต่ง `params` จะระบุชนิดข้อมูลพารามิเตอร์สำหรับสมาชิกตัวนั้นเพียงอย่างเดียว เสมือนว่าชนิดข้อมูลของมันมีการอิมพลีเมนต์ `Arbitrary` ที่รับชนิดพารามิเตอร์ดังกล่าว ซึ่งในกรณีนี้คุณ _จำเป็นต้อง_ ระบุ [`value`](#value) หรือ [`strategy`](#strategy) ควบคู่ไปด้วยเสมอ เนื่องจากชนิดข้อมูลพารามิเตอร์แบบกำหนดเองนี้มักจะไม่เข้ากันกับการเรียก `Arbitrary` ตามปกติ

นิพจน์ใดๆ (เช่นในตัวปรับแต่ง [`value`](#value) หรือ [`strategy`](#strategy)) ที่อยู่ภายใต้ขอบเขตของไอเท็มที่มีตัวปรับแต่ง `params` จะสามารถเข้าถึงตัวแปรพิเศษชื่อ `params` ซึ่งมีชนิดข้อมูลตามที่ระบุไว้ใน `#[proptest(params = ..)]` ได้ทันที

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

รูปแบบ: `#[proptest(regex = "string")]` หรือ `#[proptest(regex("string"))]` โดยที่ `string` คือเรกูลาร์เอ็กซ์เพรสชัน หรืออาจเขียนในรูป `#[proptest(regex(function_name))]` โดยที่ `function_name` เป็นชื่อฟังก์ชันที่ไม่รับอาร์กิวเมนต์และคืนค่าเป็น `&'static str`

ใช้ได้กับ: ฟิลด์

ตัวปรับแต่งนี้ใช้ระบุให้สุ่มสร้างสตริงอักขระ (`String`) หรือสตริงไบต์ (`Vec<u8>`) ที่สอดคล้องกับเรกูลาร์เอ็กซ์เพรสชันที่กำหนดสำหรับฟิลด์นั้น

ตัวปรับแต่ง `regex` เทียบเท่ากับการใช้ตัวปรับแต่ง [`strategy`](#strategy) ร่วมกับฟังก์ชัน [`string_regex`] หรือ [`bytes_regex`] โดยใช้ได้เฉพาะกับฟิลด์ชนิด `String` หรือ `Vec<u8>` เท่านั้น

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

การระบุแอตทริบิวต์ `#[proptest(skip)]` บนวาเรียนต์ของ enum จะสั่งให้ Proptest ข้ามและไม่สุ่มสร้างวาเรียนต์นั้น ฟีเจอร์นี้มีประโยชน์อย่างยิ่งเมื่อวาเรียนต์ดังกล่าวไม่สามารถสร้างค่าสุ่มได้ (เช่น เก็บ file handle หรือ network socket) หรือเมื่อคุณต้องการปิดการทดสอบวาเรียนต์บางตัวไว้ชั่วคราวระหว่างการพัฒนา

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

ข้อควรระวัง: การใส่ `#[proptest(skip)]` ให้กับทุกวาเรียนต์ของ enum จะถือเป็นข้อผิดพลาด (error) เพราะจะทำให้ Proptest ไม่เหลือตัวเลือกใดๆ สำหรับสุ่มสร้าง enum นั้นได้เลย

## `strategy`

รูปแบบ: `#[proptest(strategy = S)]` หรือ `#[proptest(strategy(S))]` โดยที่ `S` สามารถเป็นได้ทั้งสตริงที่บรรจุนิพจน์ Rust ซึ่งประเมินค่าได้เป็น `Strategy` ที่เหมาะสม หรือชื่อฟังก์ชันที่ไม่รับอาร์กิวเมนต์และคืนค่าเป็น `Strategy` ดังกล่าว

ใช้ได้กับ: วาเรียนต์ของ enum, ฟิลด์

โดยค่าเริ่มต้น วาเรียนต์ของ enum และฟิลด์ต่างๆ จะถูกสร้างขึ้นโดยเรียกใช้ `Arbitrary` ตามชนิดข้อมูลของฟิลด์นั้นๆ โดยอัตโนมัติ ตัวปรับแต่ง `strategy` ช่วยให้คุณสามารถระบุกลยุทธ์การสร้างข้อมูลที่กำหนดเอง (custom strategy) เข้าไปแทนที่ได้โดยตรง

ในกรณีของฟิลด์ กลยุทธ์ต้องสร้างค่าที่มีชนิดข้อมูลตรงกับฟิลด์นั้น ส่วนในกรณีของวาเรียนต์ของ enum กลยุทธ์ต้องสร้างค่าที่เป็นอินสแตนซ์ของ enum นั้นโดยตรง และควรเป็นค่าของวาเรียนต์ที่ระบุ

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

รูปแบบ: `#[proptest(value = V)]` หรือ `#[proptest(value(V))]` โดยที่ `V` สามารถเป็นได้ทั้ง: (a) นิพจน์ Rust ที่อยู่ในรูปสตริง, (b) ค่าลิเทอรัล (literal) ตรงๆ, หรือ (c) ชื่อฟังก์ชันที่ไม่รับอาร์กิวเมนต์

ใช้ได้กับ: วาเรียนต์ของ enum, ฟิลด์

ตัวปรับแต่ง `value` ช่วยให้คุณสามารถระบุนิพจน์หรือฟังก์ชันสำหรับกำหนดค่าคงที่/ค่าเฉพาะเจาะจงให้กับฟิลด์หรือวาเรียนต์ แทนที่จะใช้กลไกการสุ่มสร้างค่าตามปกติ

ค่าที่ระบุใน `value` จะถูกนำไปใช้เป็นค่าของฟิลด์หรือวาเรียนต์โดยตรง ยกเว้นในรูปแบบที่สาม (ชื่อฟังก์ชัน) ซึ่งระบบจะเรียกฟังก์ชันนั้นโดยไม่ส่งอาร์กิวเมนต์เพื่อรับค่าผลลัพธ์มาใช้งาน

การใช้ `value` มีค่าเทียบเท่ากับการใช้ [`strategy`](#strategy) ที่ห่อค่านั้นด้วย `LazyJust`

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

รูปแบบ: `#[proptest(weight = W)]` หรือ `#[proptest(weight(W))]` โดยที่ `W` เป็นนิพจน์ที่ประเมินได้เป็นตัวเลข `u32` นอกจากนี้ยังสามารถเขียนย่อเป็น `w` ได้ เช่น `#[proptest(w = W)]`

ใช้ได้กับ: วาเรียนต์ของ enum

ตัวปรับแต่ง `weight` ใช้กำหนดค่าน้ำหนักความน่าจะเป็นที่ Proptest จะสุ่มเลือกวาเรียนต์นั้นๆ ของ enum โดยน้ำหนักเป็นค่าสัมพัทธ์ (relative weight) เช่น วาเรียนต์ที่มี `weight = 3` จะมีโอกาสถูกสุ่มเลือกบ่อยกว่าวาเรียนต์ที่มี `weight = 2` อยู่ 50% และมีโอกาสถูกเลือกบ่อยเป็น 3 เท่าของวาเรียนต์ที่มี `weight = 1`

วาเรียนต์ที่ไม่ได้ระบุตัวปรับแต่ง `weight` จะถือว่ามีค่าเริ่มต้นเทียบเท่ากับ `#[proptest(weight = 1)]`

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
