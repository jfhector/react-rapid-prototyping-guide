# TS refreshers

# Freshness 

(Source: [Basarat's chapter on Freshness](https://basarat.gitbooks.io/typescript/content/docs/types/freshness.html))

## What's the rule? 

__Object litterals must only have _known_ properties of the interface / inline type they need to conform to.___

If a function argument must conform to an interface, which lists properties `x` and `y` (eg),
then:
.. __a variable/const__ passed as an argument __must have__ the properties `x` and `y` (i.e. similar to implementing a protocol in Swift)
.. __an object literal__ passed as an argument __must only have__ the properties `x` and `y`. (i.e. object literals must only specify known properties)

Eg with a variable:

```
function logName(something: { name: string }) {
    console.log(something.name);
}

var person = { name: 'matt', job: 'being awesome' };
var animal = { name: 'cow', diet: 'vegan, but has milk of own species' };
var random = { note: `I don't have a name property` };

logName(person); // okay
logName(animal); // okay
logName(random); // Error: property `name` is missing
```

Eg with an object litteral: 

```
function logName(something: { name: string }) {
    console.log(something.name);
}

logName({ name: 'matt' }); // okay
logName({ name: 'matt', job: 'being awesome' }); // Error: object literals must only specify known properties. `job` is excessive here.
```

Note: __object literals must only specify known properties__. This means that an object litteral passed into a function as an argument can only have the properties (optional or not) described in the interface (or inline type).

## What's the use case?

__Freshness avoid typos, by requiring that object litterals to only have known properties.__

Eg

```
function logIfHasName(something: { name?: string }) {
    if (something.name) {
        console.log(something.name);
    }
}
var person = { name: 'matt', job: 'being awesome' };
var animal = { name: 'cow', diet: 'vegan, but has milk of own species' };

logIfHasName(person); // okay
logIfHasName(animal); // okay
logIfHasName({neme: 'I just misspelled name to neme'}); // Error: object literals must only specify known properties. `neme` is excessive here.
```

## Why is this Freshness rule only in effect for object litterals?

__The reason why only object literals are type checked this way is because in this case additional properties that aren't actually used is almost always a typo or a misunderstanding of the API.__

## What do I need to do if I want more flexibility?

__A type can include _an index signature_ to explicitely indicate that excess properties are permittted.__

```
var x: { foo: number, [key: string]: any };
x = { foo: 1, baz: 2 };  // Ok, `baz` matched by index signature
```

(Note: instead of `key`, I can use whatever I want). (And obviously specify a stricter type than any).





# Specifying index signatures for objects

Eg

```
var x: { foo: number, [key: string]: any };
x = { foo: 1, baz: 2 };  // Ok, `baz` matched by index signature
```

(Note: instead of `key`, I can use whatever I want). (And obviously specify a stricter type than any).



# Other types

The type of an array of string is `string[]` (not `[string]` as a Swift).















# What's new in TypeScript (summary of the videos)

## What's new in TypeScript 2017

#### NULL AND UNDEFINED

with —strictNullChecking (which I’m encouraged to use!)

`null` and `undefined` are not valid values of any type except two unit types:
- the type `null`, which has only one value: null
- the type `undefined`, which has only one value: undefined

I can use union typed to combine them.
eg if I want a string that can also be null, the type is `string | null`

—

The type of an array with values of type string or null is `(string | null)[]`

#### CONTROL FLOW-BASED TYPE ANALYSIS

When it checks the type of the value of variable at a particular line of code, it looks at all the possible code paths leading this this line of code, and based on that determines what types the value can and and cannot be.
Eg if initially myVar is `string | null`, but I’m in a block where there’s been a JS type check to make sure it’s not null, then typescript knows that in that block, following that check, myVar can’t be null

TS also checks whether a declared variable has been assigned a value for sure or not.

#### TYPE GUARDS IN JS / TS

```
function countLines(text?: string): number {
	// here 'text' is of type 'string | undefined'

	if (!text) { return 0 }

	// my function’s code, in which text is of type 'string'
}
```

#### EMPTY STRINGS ARE FALSY, AND THE CHECKER KNOWS THIS

```
function test(s: string | string [] | undefined | null) {
	if (s) {
		s		// here the type of 's' is 'string | string[]'
	} else {
		s		// here the type of 's' is 'string | null | undefined' 
	}
}
```

this is because `’’` is falsy so that s might be a string (i.e. a falsy string) in the `else` clause

#### NULL IS OF TYPE OBJECT (it’s a mistake in JS), AND THE TYPECHECKER KNOWS THAT

```
function test(s: string | string [] | undefined | null) {
	if (typeof s === ‘object’ ) {
		s		// here the type of `s` is `string[] | null`
	} else {
		s		// here the type of `s` is `string | undefined` 
	}
}
```

—

```
`null == undefined AND THE TYPECHECKER KNOWS THAT

function test(s: string | string [] | undefined | null) {
	if (s == undefined ) {
		s		// here the type of `s` is `undefined | null`
	} else {
		s		// here the type of `s` is `string | string[]` 
	}
}
```

constrast with:

```
function test(s: string | string [] | undefined | null) {
	if (s === undefined ) {
		s		// here the type of `s` is `undefined`
	} else {
		s		// here the type of `s` is `string | string[] | null` 
	}
}
```

#### LITERAL TYPES

‘circle’ is a type, a unit type whose only value can be the string literal ‘circle’

#### DISCRIMINATED UNION TYPES

1. Use a property (eg `kind`) holding a string literal (eg. ‘circle’) as the discriminant on a union type:

```
type Shape =
	{kind: ‘circle’, radius: number} |
	{kind: ‘rectangle’, w: number, h: number} |
	{kind: ‘square’, size: number}
```

2. This way, I can discriminate among the different possible values of the union type based on the value of the shape property:

```
function getArea(shape: Shape) {
	switch (shape.kind) {
		case ‘circle’: return Math.PI * shape.radius ** 2
		case ‘rectangle’: return shape.w * shape.h
		case ‘square’: return shape.size ** 2
	}
}
```

#### NEVER TYPE

If a variable can only be one of type X Y or Z, and I’ve handled all of these types with cases in a switch statement, and returned (ie excited the function) for all these types,
if I look at the the of the variable after the switch block in the function, it is of type `never`.

I.e. all the possible types have been handled, so by the time I reach that line of code, the variable cannot be of any type.
I.e. the checker knows that I’m never going to get here, because I’ve handled all the cases.

Eg

```
type Shape =
	{kind: ‘circle’, radius: number} |
	{kind: ‘rectangle’, w: number, h: number} |
	{kind: ‘square’, size: number}

function getArea(shape: Shape) {
	switch (shape.kind) {
		case ‘circle’: return Math.PI * shape.radius ** 2
		case ‘rectangle’: return shape.w * shape.h
		case ‘square’: return shape.size ** 2
	}

	shape						// type 'never'
}
```

#### EXHAUSTIVENESS CHECKING

I can use this to check that I haven’t missed any case in a switch statement.

```
function assertNever(arg: never) {
	throw new Error(‘Unexpected object’)
}

type Shape =
	{kind: ‘circle’, radius: number} |
	{kind: ‘rectangle’, w: number, h: number} |
	{kind: ‘square’, size: number}

function getArea(shape: Shape) {
	switch (shape.kind) {
		case ‘circle’: return Math.PI * shape.radius ** 2
		// case ‘rectangle’: return shape.w * shape.h
		case ‘square’: return shape.size ** 2
	}

	assertNever(shape)							// Error Argument of type {kind: ‘rectangle’, w: number, h: number} is not assignable to parameter of type ‘never
}
```

#### TYPECHECKING IN JAVASCRIPT

VSCODE KNOWS WHAT NPM MODULES I HAVE INSTALLED
AND AUTOMATICALLY GETS ANY AVAILABLE TYPE DEFINITIONS FOR THAT MODULE ON DEFINITELY TYPED
in order to provide autocompletion


### Index access types, and lookup types
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
https://www.typescriptlang.org/docs/handbook/advanced-types.html

Index access types, aka lookup types
. look exactly like an element access, but written in type
. lookup the type of the value of a property of a type
. return a type

eg

```
interface Person {
name: string;
age: number;
location: string;
}

type P1 = Person["name"]; // string
type P2 = Person["name" | "age"]; // string | number
type P3 = string["charAt"]; // (pos: number) => string
type P4 = string[]["push"]; // (...items: string[]) => number
type P5 = string[][0]; // string

```

—

`U extends T`means:
whatever requirements are expressed by the T type/interface (ie here: it must have these properties with types of these values), U fulfils these requirements and maybe more. I can assign U to T, because U meets all the requirements of T.

(There might be a difference with object literals passed into functions as arguments, as they can only have properties named in the type description, by default).

—

`keyof T` yields the type of permitted property names for T, in an union of magic strings

eg

```
interface Person {
name: string;
age: number;
location: string;
}

type K1 = keyof Person; // "name" | "age" | "location"
type K2 = keyof Person[]; // "length" | "push" | "pop" | "concat" | ...
type K3 = keyof { [x: string]: Person };  // string
```

—

`U extends keyof T` means:
whatever requirements are expressed by the T type/interface (ie here: it must be one of the string literals that is a permitted property name of T), U fulfils these requirements and maybe more (maybe more requirements here means that U might be a subset of the string literal union expressed by `keyof T`). I can assign U to T, because U meets all the requirements of T.

—

`Pick` means
. for a type T
. for a type K, which is an union type of magic strings to is equal or more restrictive than `keyof T` (ie K can be assigned to `keyof T`)
. `Pick` is the type of an object which only has the subset of the keys of T expressed by the K union type
. ie an object like this:
`{[P in K]: T[P]}`

—

Here is an example usage for Pick:

`// From T pick a set of properties K
declare function pick(obj: T, ...keys: K[]): Pick`



## What's new in TypeScript 2018
https://channel9.msdn.com/Events/Build/2018/BRK2150

#### UNIT TYPES

In TS all values are also typed.
Eg there’s a type called 42, and it has one possible value: the value of 42.
These are called Unit Types

#### CATEGORY TYPES

Then there are different categories of types.
Eg the string category type is actually a union of all unit types that are string.
Eg the number category type is a union of all the unit types that are numbers.
Eg the boolean category type is the union type of the unit type true and the unit type false.

I can use union types make my own category types:
eg `0 | undefined | null`
eg `‘small’ l ‘medium’ | ‘big’`
eg `string | number | undefined`

#### CONTROL FLOW-BASED TYPE ANALYSIS

typechecking narrows down union types. (ie the typechecker knows what values my variables can have)
Eg if x is of type string | number | undefined, within a if (x) {} block it will be string | number

—

when I define an object type ‘a calling string’ (ie: {a: string} ),
you’re really just saying: this is any object that has a property a pf type string and then maybe some other properties

yes, it might sound counterintuitive that when I intersect two object types I get a union of property names: what I’m doing is increasing the constraints

#### INDEX TYPE
GIVES ME A UNION OF ALL OF AN OBJECT’S PROPERTY NAMES AS STRINGS

keyof T helps me reason about the names of the properties of the object T.

keyof answers the questions: what are the possible names of properties of this type?

```
type Foo = {
a: string 
b: number
c: string[]
}

keyof Foo            // ‘a’ l ‘b’ l ‘c’
```

#### INDEXED ACCESS TYPE

I’d like a type that is the same as the type of the a property in Foo

```
Foo[a] // string
Foo[c] // string[]
```

#### USING INDEX TYPE AND INDEXED ACCESS TYPE TO GET A UNION TYPE OF THE TYPES OF ALL THE VALUES CONTAINED IN AN OBJECT

```
Foo[keyof Foo] // string l number l string[]
```

**We’re operating on sets, all the time, so we’re trying to figure out ‘what could be the type of the value in any of the properties in Foo?’**

#### THIS ALLOWS YOU TO DO SAFE, CHECKED AT COMPILE TIME, OBJECT PROPERTY ACCESS

```
function getProperty(obj: T, key: K) {
obj[key] // T[K]
}
```

Remember that ‘K extends keyof T` means that K is an unit type or an union of unit types that _extends the restrictions expressed by keyof T_ (ie it’s the same union type, of q subset, even more restrictive union within it)

Example usage:

```
let foo: Foo
let aKey: keyof Foo

getProperty(foo, ‘a’) // return value will be of type string

getProperty(foo, ‘x’) // Error

getProperty(foo, key) // return value will be of type string | number | string[]
```

#### HIGHER ORDER TYPE EQUIVALENCES

`T | never <=> T`

`T & never <=> never`
ie never is a type that is so restrictive that no value can meet it

`keyof (A & B) <=> keyof A | keyof B`

#### CONDITIONAL TYPES

`T extends U ? X : Y`

With this, I don’t need to write overloads any more

Ie the return type in the example below is conditional to the type T of the parameter

#### NEW STRUCT FEATURES

Turn on —strictFunctionTypes

Turn on —strictPropertyInitialisation
can you please check that I’ve initialised my (class) properties, so that I don’t get errors when I access uninitialised (class) properties




