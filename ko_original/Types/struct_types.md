# [struct 타입](#struct-types)

* Go 버전: 1.9
* 원문 : [Struct types](https://golang.org/ref/spec#Struct_types)
* 번역자 : [연규민](@voidsatisfaction)

구조체는 이름과 타입으로 구성된 필드(field)의 연속이다. 필드의 이름은 명시적(IdentifierList) 또는 암묵적(EmbeddedField)으로 표현할 수 있다. [blank](/Declarations%20and%20scope/blank_identifier.html) 식별자를 제외한 구조체 내 모든 필드의 이름은 [고유](/Declarations%20and%20scope/uniqueness_of_identifiers.html)해야 한다.

<pre>
<a id="StructType">StructType</a>     = "struct" "{" { <a href="#FieldDecl">FieldDecl</a> ";" } "}" .
<a id="FieldDecl">FieldDecl</a>      = (<a href="/Declarations%20and%20scope/constant_declarations.html#IdentifierList">IdentifierList</a> <a href="/Types/#Type">Type</a> | <a href="#EmbeddedField">EmbeddedField</a>) [ <a href="#Tag">Tag</a> ] .
<a id="EmbeddedField">EmbeddedField</a> = [ "*" ] <a href="/Types/#TypeName">TypeName</a> .
<a id="Tag">Tag</a>            = <a href="/Lexical%20elements/string_literals.html#string_lit">string_lit</a> .
</pre>

```
// An empty struct.
struct {}

// 6개의 필드를 가진 구조체.
struct {
	x, y int
	u float32
	_ float32  // 패딩(padding)
	A *[]int
	F func()
}
```

명시적인 이름이 없이 타입만 선언된 필드를 *임베디드 필드(embedded field)* 라고 한다. 임베디드 필드는 타입 이름 `T` 또는 `*T`와 같은 포인터로 나타낼 수 있다. 단, `T`는 포인터 타입이 될 수 없고, `*T`에는 인터페이스 타입을 쓸 수 없다. unqualified 타입 이름은 필드 이름처럼 사용된다.

```
// T1, *T2, P.T3 , *P.T 타입의 임베디드 필드 4개가 있는 구조체
struct {
	T1        // 필드 이름은 T1
	*T2       // 필드 이름은 T2
	P.T3      // 필드 이름은 T3
	*P.T4     // 필드 이름은 T4
	x, y int  // 필드 이름은 x, y
}
```

구조체의 필드 이름은 고유해야 하기 때문에 아래와 같은 선언문은 허용되지 않는다:

```
struct {
	T     // 임베디드 필드 *T, *P.T와 충돌
	*T    // 임베디드 필드 T, *P.T와 충돌
	*P.T  // 임베디드 필드 T, *T와 충돌
}
```

`x.f`가 [selector](/Expressions/selectors.html)로서 유효한 표현이며 `f`가 구조체 `x`의 임베디드 필드 속 필드나 [메서드](/Declarations%20and%20scope/method_declarations.html)일 경우 필드(혹은 메서드)`f`가 *승진(promoted)* 되었다고 말한다.

promoted 필드는 구조체의 [합성 리터럴](/Expressions/composite_literals.html)에서 필드 이름으로 사용 될 수 없는 것을 제외하면 일반적인 구조체의 필드처럼 동작한다.

struct 타입이 `S`이고 `T`라는 이름의 한 타입이 주어졌을때, 그 구조체의 메서드 집합(method set)에 promoted 메서드가 포함되는 경우는 다음과 같다:

  * `S`가 임베디드 필드 `T`를 포함하면, `S`와 `*S`의 [메서드 집합](/Types/method_sets.html)은 receiver를 `T`로 하는 promoted 메서드를 포함한다. 또한, `*S`의 메서드 집합은 receiver를 `*T`로 하는 promoted 메서드를 포함한다.
  * `S`가 임베디드 필드 `*T`를 포함하면, `S`와 `*S`의 메서드 집합은 receiver를 `T`나`*T`로 하는 promoted 메서드를 포함한다.

필드 선언시 문자열 리터럴 *태그* 를 덧붙일 수 있으며, 이것은 해당 필드의 선언문에서 필드의 속성이 된다. empty 태그 문자열은 태그의 부재를 의미한다. 태그들은 [reflection 인터페이스](https://golang.org/pkg/reflect/#StructTag)를 이용해 확인할 수 있으며, 구조체의 [타입 아이덴티티](/Properties%20of%20types%20and%20values/type_identity.html)를 판단할 때 영향을 준다. 그외의 경우에는 무시된다.

```
struct {
	x, y float64 ""  // empty 태그 문자열은 태그의 부재를 의미한다.
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// TimeStamp protocol buffer에 해당하는 구조체.
// reflect 패키지에서 정한 관례에 따라 태그 문자열로 protocol buffer 필드 숫자를 정의함
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```
