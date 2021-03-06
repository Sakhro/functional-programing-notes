# Functional programing notes with code examples

---

## Table of contents
- [Functional programing notes with code examples](#functional-programing-notes-with-code-examples)
  - [Table of contents](#table-of-contents)
  - [Type Signatures](#type-signatures)
  - [Setoit](#setoit)
    - [Laws](#laws)
  - [Ord](#ord)
    - [Laws](#laws-1)
  - [Box](#box)
    - [Examples:](#examples)
  - [LazyBox](#lazybox)
    - [Examples:](#examples-1)
  - [Either](#either)
    - [Examples:](#examples-2)
  - [Function](#function)
    - [Laws](#laws-2)
    - [Examples](#examples-3)
  - [Compose](#compose)
    - [Examples](#examples-4)
  - [Monad transformers](#monad-transformers)
    - [Examples](#examples-5)
  - [Free Monad](#free-monad)
    - [Examples](#examples-6)
  - [Semigroup](#semigroup)
    - [Laws](#laws-3)
    - [Examples:](#examples-7)
  - [Monoid](#monoid)
    - [Laws](#laws-4)
    - [Examples:](#examples-8)
  - [Task](#task)
    - [Examples:](#examples-9)
  - [Applicative](#applicative)
    - [Laws](#laws-5)
    - [Examples](#examples-10)
    - [Alt](#alt)
  - [Laws](#laws-6)
    - [Plus](#plus)
  - [Laws](#laws-7)
    - [Alternative](#alternative)
  - [Laws](#laws-8)
  - [Foldable](#foldable)
  - [Traversable](#traversable)
  - [Laws](#laws-9)
    - [Examples](#examples-11)
  - [Functors](#functors)
    - [Laws](#laws-10)
    - [Examples](#examples-12)
  - [Contravariant](#contravariant)
    - [Laws](#laws-11)
    - [Examples](#examples-13)
  - [Apply](#apply)
    - [Laws](#laws-12)
    - [Examples](#examples-14)
  - [Monads](#monads)
    - [Laws](#laws-13)
  - [Natural Transformations](#natural-transformations)
    - [Laws](#laws-14)
    - [Examples](#examples-15)
  - [Isomorphisms and round trip data transformations](#isomorphisms-and-round-trip-data-transformations)
    - [Laws](#laws-15)
    - [Examples](#examples-16)
  - [Real world app examples](#real-world-app-examples)
    - [Validation library](#validation-library)
    - [Spotify app](#spotify-app)
    - [Redux type app](#redux-type-app)
  - [Resources](#resources)

---

## Type Signatures

The type signature notation used in this document is described below:<sup
id="sanctuary-types-return">[1](#sanctuary-types)</sup>

* `::` _"is a member of"._
    - `e :: t` can be read as: "the expression `e` is a member of type `t`".
    - `true :: Boolean` - "`true` is a member of type `Boolean`".
    - `42 :: Integer, Number` - "`42` is a member of the `Integer` and
      `Number` types".
* _New types can be created via type constructors._
    - Type constructors can take zero or more type arguments.
    - `Array` is a type constructor which takes one type argument.
    - `Array String` is the type of all arrays of strings. Each of the
      following has type `Array String`: `[]`, `['foo', 'bar', 'baz']`.
    - `Array (Array String)` is the type of all arrays of arrays of strings.
      Each of the following has type `Array (Array String)`: `[]`, `[ [], []
      ]`, `[ [], ['foo'], ['bar', 'baz'] ]`.
* _Lowercase letters stand for type variables._
    - Type variables can take any type unless they have been restricted by
      means of type constraints (see fat arrow below).
* `->` (arrow) _Function type constructor._
    - `->` is an _infix_ type constructor that takes two type arguments where
      left argument is the input type and the right argument is the output type.
    - `->`'s input type can be a grouping of types to create the type of a
      function which accepts zero or more arguments. The syntax is:
      `(<input-types>) -> <output-type>`, where `<input-types>` comprises zero
      or more comma–space (`, `)-separated type representations and parens
      may be omitted for unary functions.
    - `String -> Array String` is a type satisfied by functions which take a
      `String` and return an `Array String`.
    - `String -> Array String -> Array String` is a type satisfied by functions
      which take a `String` and return a function which takes an `Array String`
      and returns an `Array String`.
    - `(String, Array String) -> Array String` is a type satisfied by functions
      which take a `String` and an `Array String` as arguments and return an
      `Array String`.
    - `() -> Number` is a type satisfied by functions
      which do not take arguments and return a `Number`.
* `~>` (squiggly arrow) _Method type constructor._
    - When a function is a property of an Object, it is called a method. All
      methods have an implicit parameter type - the type of which they are a
      property.
    - `a ~> a -> a` is a type satisfied by methods on Objects of type `a` which
      take a type `a` as an argument and return a value of type `a`.
* `=>` (fat arrow) _Expresses constraints on type variables._
    - In `a ~> a -> a` (see squiggly arrow above), `a` can be of any type.
      `Semigroup a => a ~> a -> a` adds a constraint such that the type `a`
      must now satisfy the `Semigroup` typeclass. To satisfy a typeclass means
      to lawfully implement all functions/methods specified by that typeclass.

For example:

```
fantasy-land/traverse :: Applicative f, Traversable t => t a ~> (TypeRep f, a -> f b) -> f (t b)
'-------------------'    '--------------------------'    '-'    '-------------------'    '-----'
 '                        '                               '      '                        '
 '                        ' - type constraints            '      ' - argument types       ' - return type
 '                                                        '
 '- method name                                           ' - method target type
```

```JS
add :: Int -> Int -> Int
--      ^ arg  ^ arg
--                    ^ return
```

> In English, this says that our add function takes an integer x, and returns a function that takes an integer y, which returns an integer (probably x + y).

---

## Setoit

> A setoid is any type with a notion of equivalence. We use plenty of setoids (integers, booleans, strings) almost every time you use the == operator, so this shouldn’t be too tricky.

Based on Fantasy Land-compliant it must have a prototype method called `equals`:

```JS
equals :: Setoid a => a ~> a -> Boolean
```

### Laws

- **Reflexivity** `a.equals(a) === true`
- **Symmetry** or **commutativity** `a.equals(b) === b.equals(a)`
- **Transitivity** `If a.equals(b)` and `b.equals(c)`, then it’s always `true` that `a.equals(c)`

---

## Ord

> `Ord` types are types with a total ordering. That means that, given any two values of a given `Ord` type, you can determine whether one be greater than the other. To do this, we actually only need one method. Given that all `Ord` types must also be `Setoid` types, it could actually have been any of the comparison operators (`>`, `>=`, `<`, `<=`; think about why any of these would have worked), but the spec settled on `<=` (less-than-or-equal), which it refers to as `lte`:

```JS
lte :: Ord a => a ~> a -> Boolean
```

```JS
// Greater than. The OPPOSITE of lte.
// gt :: Ord a => a -> a -> Boolean
const gt = function (x, y) {
  return !lte(x, y)
}

// Greater than or equal.
// gte :: Ord a => a -> a -> Boolean
const gte = function (x, y) {
  return gt(x, y) || x.equals(y)
}

// Less than. The OPPOSITE of gte!
// lt :: Ord a => a -> a -> Boolean
const lt = function (x, y) {
  return !gte(x, y)
}

// And we already have lte!
// lte :: Ord a => a -> a -> Boolean
const lte = function (x, y) {
  return x.lte(y)
}
```

### Laws

- **Totality** `a.lte(b) || b.lte(a) === true`
- **Antisymmetry** `a.lte(b) && b.lte(a) === a.equals(b)`
- **Transitivity** `a.lte(b) && b.lte(c) === a.lte(c)`
  
---

## Box

We do composition withing the context. And `Box` is our context. It is also an indentity functor

```JS
const Box = x => ({
  ap: b2 => b2.map(x),
  chain: f => f(x),
  map: f => Box(f(x)),
  fold: f => f(x),
  inspect: () => `Box(${x})`
})
```

### Examples:

```JS
const nextCharFromNumberString = (str) => 
  Box(str.trim())
    .map(s => parseInt(s))
    .map(n => n + 1)
    .map(nn => String.toCharCode(nn))

nextCharFromNumberString("   64 ") // Box(A)
```

> What happened here? We've captured each assignment in a very minimal context. S cannot be used outside of this little error function in map. Despite calling it the same variable here, we can change this to R or whatever we want to call it.

> The point is, each expression has its own state completely contained. We can break up our work flow, and go top to bottom, doing one thing at a time, composing together. That's key. Map is composition, because it takes input to output and passes it along to the next map. We're composing this way.

```JS
const stripDollarSign = (moneyString) => moneyString.replace(/\$/g, '')
const stripPercentSign = (percentString) => percentString.replace(/%/g, '')
const percentToPercentDecimal = (float) => float / 100

const moneyToFloat = (moneyString) =>
  Box(moneyString)
    .map(stripDollarSign)
    .map(parseFloat)

const percentToFloat = (percentString) =>
  Box(percentString)
    .map(stripPercentSign)
    .map(parseFloat)
    .map(percentToPercentDecimal)

export const applyDiscount = (priceString, discountString) =>
  moneyToFloat(priceString)
    .fold(priceFloat =>
      percentToFloat(discountString)
        .fold(discountFloat =>
          priceFloat - (priceFloat * discountFloat)))

applyDiscount("$5.00", "20%")
```

----

## LazyBox

Box example using lazy evaulation

```JS
const LazyBox = g => ({
  map: f => LazyBox(() => f(g())),
  fold: f => f(g()),
})
```

### Examples:

```JS
LazyBox(str.trim())
  .map(s => parseInt(s))
  .map(n => n + 1)
  .fold(nn => String.toCharCode(nn))
```
> This gives us purity by virtue of laziness. Basically, nothing happens, so we don't have any impure side effects, until the very end, when we call fold. We're pushing it all the way down to the bottom. This is how a variety of types define map, where they have a function inside them instead of a concrete value, such as promises, observables, or streams, things like this.

---

## Either

> Either is defined as a right or a left. These are two sub-types, or sub-classes if you will, of either, and either doesn't actually come into play. It will just refer to one of these two types.

```JS
const Either = Right || Left

const Right = x => ({
  chain: f => f(x),
  // Transform the inner value
  // map :: Either a b ~> (b -> c) -> Either a c
  map: f => Right(f(x)),
  // Get the value with the right-hand function
  // fold :: Either a b ~> (a -> c, b -> c) -> c
  fold: (f, g) => g(x),
  inspect: () => `Right(${x})`
})

const Left = x => ({
  chain: f => f(x),
  // Do nothing
  // map :: Either a b ~> (b -> c) -> Either a c
  map: f => Left(x),
  // Get the value with the left-hand function
  // fold :: Either a b ~> (a -> c, b -> c) -> c
  fold: (f, g) => f(x),
  inspect: () => `Left(${x})`
})

const fromNullable = x =>
  [undefined, null].includes(x) 
    ? Left(x) 
    : Right(x)

const tryCatch = f => {
  try {
    const result = f()
    return Right(result)
  } catch (e) {
    return Left(e)
  }
}

```

### Examples:

```JS
Right(3)
  .map(x => x + 1)
  .map(x => x / 2) // Right(2)
  .fold(x => "error", x => x) // 2

Left(3)
  .map(x => x + 1)
  .map(x => x / 2) // Right(3)
  .fold(x => "error", x => x) // error
```

```JS
const findColor = (name: string) => {
  const colors = {red: '#ff4444', green: 'verde', blue: 'azul'}
  const found = colors[name]
  return fromNullable(found)
}

findColor("red")
  .map(c => c.slice(1))
  .fold(e => "No color", c => c.toUpperCase()) // FF4444
```


```JS
const getPort = () => 
  tryCatch(() => fs.readFile("config.json"))
  .chain(c => tryCatch(() => JSON.stringify(c)))
  .fold(() => 3000, c => c.port)
```

```JS
// export const openSite = (currentUser) => {
//   if (currentUser) {
//     return renderPage(currentUser)
//   } else {
//     return showLogin()
//   }
// }

const openSite = (currentUser: { username: string } | void): string => {
  const showLogin = () => 'Please login'
  const renderPage = (currentUser: { username: string }) => `Hello ${currentUser.username}`

  return fromNullable(currentUser)
    .fold(showLogin, renderPage)
}
```

```JS
// export const getPrefs = (user) => {
//   if (user.premium) {
//     return user.preferences
//   } else {
//     return 'DEFAULT_PREFERENCES'
//   }
// }

export const getPrefs = (user: { preferences: string, premium: boolean | void }): string => {
  const getPremiumStatus = (user: { preferences: string, premium: boolean | void }): Right | Left => fromNullable(user.premium)
  const loadDefaultPrefs = () => 'DEFAULT_PREFERENCES'
  const loadUserPrefs = (user) => user.preferences

  return new Right(user)
    .chain(getPremiumStatus)
    .fold(loadDefaultPrefs, () => loadUserPrefs(user))
}
```

```JS

// export const getStreetName = (user) => {
//   const address = user.address
//
//   if (address) {
//     const street = address.street
//
//     if (street) {
//       return street.name
//     }
//   }
//   return 'no street'
// }

export const getStreetName = (user: { address: {} | void }): string => {
  const getAddress = (user) => fromNullable(user.address)
  const getStreet = (address) => fromNullable(address.street)
  const getName = (street) => street.name
  const onError = (_e) => 'no street'
  const onSuccess = (name) => name

  return new Right(user)
    .chain(getAddress)
    .chain(getStreet)
    .map(getName)
    .fold(onError, onSuccess)
}
```

```JS
// export const concatUniq = (x, ys) => {
//   const found = ys.filter(y => y===x)[0]
//   return foind ? ys : ys.concat(x)
// }

export const concatUniq = (x: number, ys: Array<number>): Array<number> => {
  const combine = (x, ys) => ys.concat(x)
  const returnOriginal = (ys) => ys

  return fromNullable(ys.filter(y => y === x)[0])
    .fold(() => combine(x, ys), () => returnOriginal(ys))
}
```

```JS
// export const wrapExample = (example: { previewPath?: string }): { previewPath?: string } => {
//   if (example.previewPath) {
//     try {
//       example.preview = fs.readFileSync(example.previewPath)
//     } catch (e) {
//       console.log(e)
//     }
//   }
//   return example
// }

export const wrapExample = (example: { previewPath?: string }): { previewPath?: string } => {
  const readFile = (filepath) => tryCatch(() => fs.readFileSync(filepath, 'utf-8'))
  const onError = () => example
  const onSuccess = (preview) => Object.assign({}, example, { preview })
  // const onSuccess = (preview) => { ...example, preview )

  return fromNullable(example.previewPath)
    .chain(readFile)
    .fold(onError, onSuccess)
}
```

```JS
// export const parseUrl = (config) => {
//   const urlRegEx = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/
//   try {
//     const c = JSON.parse(config)
//     if (!c.url) return null
//     return c.url.match(urlRegEx)
//   } catch (e) {
//     return null
//   }
// }

export const parseUrl = (config: string): string | void => {
  const onError = () => null
  const onSuccess = (url) => url

  return tryCatch(() => JSON.parse(config))
    .chain((c) => fromNullable(c.url))
    .fold(onError, onSuccess)
}
```

---

## Function

Functions are functors

```JS
map :: Functor f => (a -> b) -> f a -> f b

// (a -> b) ~> (b -> c) -> a -> c
Function.prototype.map = function (that) {
  return x => that(this(x))
}
```

### Laws

- Identity
```JS
f.map(id)
  // By definition function's `map`
  === x => id(f(x))
  // By definition of `id`
  === x => f(x)
  // We're there!
  === f
```

- Composition
```JS
compose(map(h), map(g))(f)
  // By composition's definition
  === map(h)(map(g)(f))
  // By map's definition
  === (map(g)(f)).map(h)
  // ... and again...
  === f.map(g).map(h)
  // By function's map definition
  === (x => g(f(x))).map(h)
  // ... and again... eep...
  === y => h((x => g(f(x)))(y))
  // Applying y to (x => g(f(x)))...
  === y => h(g(f(y)))
  // By composition's definition...
  === y => compose(h, g)(f(y))
  // By function's map definition...
  === (y => f(y)).map(compose(h, g))
  // YAY!
  === f.map(compose(h, g))
```

### Examples 

```JS
const Fn = (run) => ({
  run,
  chain: (f) => Fn((x) => f(run(x)).run(x)),
  map: (f) => Fn((x) => f(run(x))),
  concat: (other) => Fn((x) => run(x).concat(other.run(x))),
});


Fn(toUpper).concat(Fn(exclaim)).run('Hello');

Fn(toUpper)
  .chain((upper) => Fn((y) => [upper, exclaim(y)]))
  .run('Hello');

Fn.of('Hi')
  .map(toUpper)
  .chain((upper) => Fn((y) => [upper, exclaim(y)]))
  .run('Hello');

const app = Fn.of('Start')
  .map(toUpper)
  .chain((upper) => Fn((config) => [upper, config]));

const app = Fn.of('Start')
  .map(toUpper)
  .chain((upper) => Fn.ask.map((config) => [upper, config]));

app.run({ port: 3000 }); // [ 'START', { port: 3000 } ] from the line 565
```

```JS
// Endo is composable function.

const Endo = (run) => ({
  run,
  concat: (other) => Endo((x) => run(other.run(x))),
});

Endo.empty = () => Endo((x) => x)

const app = List([toUpper, exclaim])
  .foldMap(Endo, Endo.empty(""))  
  
app.run("Endo") // ENDO!

```

```JS

const classToClassName = html =>
    html.replace(/class\=/ig, 'className=')

const updateStyleTag = html =>
    html.replace(/style="(.*)"/ig, 'style={{$1}}')

const htmlFor = html =>
    html.replace(/for=/ig, 'htmlFor=')

const ex1 = html => 
	Endo(classToClassName)
	.concat(Endo(updateStyleTag))
	.concat(Endo(htmlFor))
	.run(html)

// OR

List([classToClassName, updateStyleTag, htmlFor])
  .foldMap(Endo, Endo.empty())
  .run(html)

// Same

[classToClassName, updateStyleTag, htmlFor]
  .reduce((acc, x) => acc.concat(Endo(x)), Endo.empty())
	.run(html)
```
---

## Compose

We can't use chain/foldMap with the Compose.

### Examples

```JS
const Compose = (F, G) => {
  const M = fg => ({
    extract: () => fg,
    map: f => M(fg.map(g => g.map(f)))
  })

  M.of = x => M(F.of(G.of(x)))

  return M
} 

const TaskEither = Compose(Task, Either)

TaskEither.of(2)
  .map(two => two * 10)
  .map(twenty => twenty + 1)
  .extract()
  .fork(console.error, either => 
    either.fold(console.log, console.log)) // 21
```
---

## Monad transformers

### Examples

```JS
const find = (table, query) =>
  Task.of(Either.fromNulluble(find(table, query)))

// Insted of this bellow
const app = () => 
  find(users, { id: 1 }) // Task(Either(User))
  .chain(eu => 
    eu.fold(Task.rejected, u => find(following, { follow_id: u.id }))
  ).chain(eu => 
    eu.fold(Task.rejected, fo => find(users, { id: fo.user_id }))
  )
  .fork(console.error, eu => eu.fold(console.error, console.log))

// To this
const TaskEither = TaskT(Either) 
// TaskEither.of(Either...) => Task(Either(Either...))
// TaskEither.lift(Either...) => Task(Either(x))

const find = (table, query) =>
  TaskEither.lift(Either.fromNulluble(find(table, query)))

const app = () =>
  find(users, { id: 1 }) // Task(Either(User))
  .chain(u => find(following, { follow_id: u.id })) // Task(Either(User))
  .chain(fo => find(users, { id: fo.user_id })) // Task(Either(User))
  .fork(console.error, eu => eu.fold(console.error, console.log))
```

```JS
const Fn = g =>
({
  map: f =>
    Fn(x => f(g(x))),
  chain: f =>
    Fn(x => f(g(x)).run(x)),
  run: g
})
Fn.ask = Fn(x => x)
Fn.of = x => Fn(() => x)

const FnT = M => {
  const Fn = g =>
	({
	  map: f =>
		Fn(x => g(x).map(f)),
	  chain: f =>
		Fn(x => g(x).chain(y => f(y).run(x))),
	  run: g
	})
  Fn.ask = Fn(x => M.of(x))
  Fn.of = x => Fn(() => M.of(x))
  Fn.lift = x => Fn(() => x)
  return Fn
}

const FnEither = FnT(Either)

const ex1 = () => FnEither.ask.map(x => x.port)

ex1(1).run({port: 8080}).fold(x => x, x => x) // 8080

const fakeDb = xs => ({find: (id) => Either.fromNullable(xs[id])})

const connectDb = port =>
    port === 8080 ? Right(fakeDb(['red', 'green', 'blue'])) : Left('failed to connect')

const ex1a = id => 
	ex1()
	.chain(port => FnEither.lift(connectDb(port)))
	.chain(db => FnEither.lift(db.find(id)))
	
ex1a(1).run({port: 8080}).fold(x => x, x => x) // green
```

```JS
const posts = [{id: 1, title: 'Get some Fp'}, {id: 2, title: 'Learn to architect it'}, {id: 3}]

const postUrl = (server, id) => [server, id].join('/')

const fetch = url => url.match(/serverA/ig) ? Task.of({data: JSON.stringify(posts)}) : Task.rejected(`Unknown server ${url}`)

const ReaderTask = FnT(Task)

const ex2 = id =>
	ReaderTask.ask
	.chain(server => ReaderTask.lift(fetch(postUrl(server, id)).map(x => x.data).map(JSON.parse)))

ex2(30)
	.run('http://serverA.com')
	.fork(
		e => console.error(e),
		posts => posts[0].title
	) // 'Get some Fp'

```

---

## Free Monad

### Examples

```JS
const {liftF} = require('../lib/free')
const {Id} = require('../lib/types')
const {taggedSum} = require('daggy')

const Http = taggedSum('Http', {Get: ['url'], Post: ['url', 'body']})

const httpGet = (url) => liftF(Http.Get(url))
const httpPost = (url, body) => liftF(Http.Post(url, body))

const app = () =>
  httpGet('/home')
  .chain(contents => httpPost('/analytics', contents))

const interpret = x =>
  x.cata({
    Get: url => Id.of(`contents for ${url}`),
    Post: (url, body) => Id.of(`posted ${body} to ${url}`)
  })

const res = app().foldMap(interpret, Id.of)
```

---

## Semigroup

> Semi-groups are simply a type with a concat method that are associative

```JS
concat :: Semigroup a => a ~> a -> a
```

```JS
const Sum = x => ({
  x,
  concat: ({ x: y }) => Sum(x + y)
})

const Product = x => ({
  x,
  concat: ({ x: y }) => Product(x * y)
})

const All = x => ({
  x,
  concat: ({ x: y }) => All(x && y)
})

const Any = x => ({
  x,
  concat: ({ x: y }) => Any(x || y)
})

const Max = x => ({
  x,
  concat: ({ x: y }) => Max(x > y ? x : y)
})

const Min = x => ({
  x,
  concat: ({ x: y }) => Min(x <> y ? x : y)
})

const First = either => ({
  fold: f => f(either),
  concat: other => other.isLeft ? o : First(either)
})

const Fn = f => ({
  fold: f,
  concat: o => Fn(x => f(x).concat(o.fold(x)))
})

const Paid = (x, y) => ({
  x,
  y,
  concat: ({ x: x1, y: y1 }) => Pair(x.concat(x1), y.concat(y1))
})
```

### Laws

- Associativity `a.concat(b).concat(c) === a.concat(b.concat(c))`

### Examples:

```JS
Sum(1).concat(Sum(2)) // Sum(3)

All(true).concat(All(false)) // All(false)
All(true).concat(All(true)) // All(true)

First("first").concat(First("second")) // First("first")
```

```JS
// We use Map for concat method for object
const { Map } = require('immutable-ext')

const acc1 = Map({
  name: First("Niko"),
  paid: All(false),
  points: Sum(10),
  friends: ["Franklin"]
})

const acc2 = Map({
  name: First("Niko"),
  paid: All(true),
  points: Sum(2),
  friends: ["Gatsby"]
})

acc1.concat(acc2) // { name: First("Niko"), paid: All(true), points: Sum(12), friends: ["Franklin", "Gatsby"] }
```

```JS
const hasVowels = x => !!x.match(/[aeiou]/ig)
const longWord = x => x.length >= 5

const both = Fn(compose(All, hasVowels)).concat(Fn(compose(All, longWord)))

["gym", "bird", "lilac"],filter(x => both.fold(x).x) // [lilac]
```
---

## Monoid

> If we have a special element like the zero here under addition, we have what's called a monoid, that is a semigroup with a special element in there that acts like a neutral identity.

```JS
empty :: Monoid m => () -> m
```

```JS
...
Sum.empty = () => Sum(0)

...
All.empty = () => All(true)

...
Any.empty = () => Any(false)

...
Max.empty = () => Max(-Infinity)

...
Min.empty = () => Min(Infinity)

...
Product.empty = () => Product(1)

// Cheat
...
First.empty = () => First(Left())
```

### Laws

- Right identity `MyType(x).concat(MyType.empty()) === MyType(x)`
- Left identity `MyType.empty().concat(MyType(x)) === MyType(x)`

### Examples:

```JS
Sum.empty().concat(Sum(1).concat(Sum(2))) // Sum(3)

All(true).concat(All(true)).concat(All.empty()) // All(true)
```

```JS
const sum = xs => xs.reduce((acc, x) => acc + x, 0)

const all = xs => xs.reduce((acc, x) => acc && x, true)
```

```JS
// A friendly neighbourhood monoid fold.
// fold :: Monoid m => (a -> m) -> [a] -> m
const fold = M => xs => xs.reduce(
  (acc, x) => acc.concat(M(x)),
  M.empty())

// We can now use our monoids for (almost) all
// our array reduction needs!
fold(Sum)([1, 2, 3, 4, 5]).val // 15
fold(Product)([1, 2, 3]).val   // 6
fold(Max)([9, 7, 11]).val      // 11
fold(Sum)([]).val              // 0 - ooer!
```

```JS
// We import List type for foldMap method
const { List } = require('immutable-ext')

const stats = List.of(
  { page: "Home", views: 40 },
  { page: "About", views: 10 },
  { page: "Blog", views: 4 },
)

stats.foldMap(x => fromNullable(x.view).map(Sum), Right(Sum(0))) // Right(Sum(54))
```

```JS
// We import List type for foldMap method
const { List } = require('immutable-ext')

const find = (xs, f) => 
  List(xs)
    .foldMap(x => First(f(x) ? Right(x) : Left()), First.empty())
    .fold(x => x)

find([3, 4, 5, 6, 7], x => x > 4) // Right(5)
```

```JS
const hasVowels = x => !!x.match(/[aeiou]/ig)
const longWord = x => x.length >= 5

const both = Fn(compose(All, hasVowels)).concat(Fn(compose(All, longWord)))

["gym", "bird", "lilac"],filter(x => both.fold(x).x) // [lilac]
```

```JS
const { Map, List } = require('immutable-ext')

Map({ brian: Sum(4), sara: Sum(2) })
  .fold(Sum.empty()) // Sum(6)

Map({ brian: 4, sara: 2 })
  .map(Sum)
  .fold(Sum.empty()) // Sum(6)

List.of([Sum(1), Sum(2), Sum(3)])
  .fold(Sum.empty()) // Sum(6)

List.of([1, 2, 3])
  .foldMap(Sum, Sum.empty()) // Sum(6)
```

---

## Task

We're going to use data.task on NPM from Folktale in this examples

### Examples:
```JS
import Task from "data.task"

Task.of(1)
  .map(x => x + 1)
  .chain(x => Task.of(x + 1))
  .fork(e => "error", x => x) // 3

Task.rejected(1)
  .map(x => x + 1)
  .chain(x => Task.of(x + 1))
  .fork(e => e, x => x) // 1
```

```JS
import Task from "data.task"

const launchMissiles = () => 
  new Task((rej, res) => {
    console.log("launch missiles")
    res("missile")
  })

const app = launchMissiles()
  .map(x => x + "!")

/**
 * we can keep extending things, and composing along, 
 * if we allow user to fork the app where he wants, 
 * so and our whole application remains pure.
*/
app
  .map(x => x + "!")
  .fork(e => "error", x => x) // missile!!
```

```JS
import Task from "data.task"
import fs from 'fs'

// const app = () => {
//   fs.readFile("config.json", "utf-8", (err, contents) => {
//     if (err) throw err

//     const newContents = contents.replace(/8/g, "6")

//     fs.writeFile("config.json", newContents, (err, _) => {
//       if (err) throw err

//       console.log("success")
//     })
//   })
// }

const readFile = (filename, enc) =>
  new Task((rej, res) => 
    fs.readFile(filename, enc, (err, contents) => 
      err ? rej(err) : res(contents)))

const writeFile = (filename, contents) =>
  new Task((rej, res) => 
    fs.writeFile(filename, contents, (err, success) => 
      err ? rej(err) : res(success)))

const app = 
  readFile("config.json", "utf-8")
    .map(contents => contents.replace(/8/g, "6"))
    .chain(contents => writeFile("config.json", contents))

app.fork(e => console.log("error"), x => console.log("success"))
```

```JS
const readFile = futurize(fs.readFile)

const files = List(["box.js", "config.json"])

files.map(fn => readFile(fn, "utf-8")) // [Task, Task]

// Traverse help us to flip types: [Task] => Task([...])
const res1 = files.traverse(Task.of, fn => readFile(fn, "utf-8"))

res1.fork(e => "error", result => result) // List([{...}, {...}])
```
> Not all types have a traverse instance, that means they can't define traverse. Things like stream for instance. However, the more important thing to know is, if you see a traverse, yes you can traverse it.

```JS
const httpGet = (path, params) => 
  Task.of(`${path} result`)

Map({ home: "/", about: "/about-us", blog: "blog" })
  .map(route => httpGet(route, {})) // Map({home: Task("/ result), ...}) but we want to have Task(Map({ home: "/ result", ... })), for this we use traverse

Map({ home: "/", about: "/about-us", blog: "blog" })
  .traverse(Task.of, route => httpGet(route, {}))
  .fork(e => "error", x => x) // Map({ home: "/ result", ...})

Map({ home: ["/", "/home"], about: ["/about-us"] })
  .traverse(Task.of, routes => 
    List(routes)
      .traverse(Task.of, route => httpGet(route, {})))
  .fork(e => "error", x => x) // Map({ home: List("/ result", "/home result"), ...})
```

---

## Applicative

```JS
of :: Applicative f => a -> f a
```

### Laws
- Identity `v.ap(A.of(x => x)) === v`
- Homomorphism `A.of(x).ap(A.of(f)) === A.of(f(x))`
- Interchange `A.of(y).ap(u) === u.ap(A.of(f => f(y)))`

### Examples

```JS
// append :: a -> [a] -> [a]
const append = y => xs => xs.concat([y])

// There's that sneaky lift2 again!
// lift2 :: Applicative f
//       => (a -> b -> c, f a, f b)
//       -> f c
const lift2 = (f, a, b) => b.ap(a.map(f))

// insideOut :: Applicative f
//           => [f a] -> f [a]
const insideOut = (T, xs) => xs.reduce(
  (acc, x) => lift2(append, x, acc),
  T.of([])) // To start us off!

// For example...

// Just [2, 10, 3]
insideOut(Maybe, [ Just(2)
                 , Just(10)
                 , Just(3) ])

// Nothing
insideOut(Maybe, [ Just(2)
                 , Nothing
                 , Just(3) ])
```

---

### Alt

```JS
alt :: Alt f => f a ~> f a -> f a
```

## Laws
- Associativity `a.alt(b).alt(c) === a.alt(b.alt(c))`
- Distributivity `a.alt(b).map(f) === a.map(f).alt(b.map(f))`

---

### Plus

```JS
zero :: Plus f => () -> f a
```

## Laws
- Right identity - zero on the right `x.alt(A.zero()) === x`
- Left identity `A.zero().alt(x) === x`
- Annihilation `A.zero().map(f) === A.zero()`

---

### Alternative

> There are no special functions for this one, as it is simply the name for a structure that implements both `Plus` and `Applicative`

## Laws
- Distributivity `x.ap(f.alt(g)) === x.ap(f).alt(x.ap(g))`
- Annihilation `x.ap(A.zero()) === A.zero()`

---

## Foldable

```JS
reduce :: Foldable f => f a ~> ((b, a) -> b, b) -> b
```

```JS
fold :: (Foldable f, Monoid m)
     => (a -> m) -> f a -> m

// A friendly neighbourhood monoid fold.
// fold :: Monoid m => (a -> m) -> [a] -> m
const fold = M => xs => xs.reduce(
  (acc, x) => acc.concat(M(x)),
  M.empty())
```

---

## Traversable

```JS
traverse :: Applicative f, Traversable t
         => t a -> (TypeRep f, a -> f b)
         -> f (t b)
```

## Laws 
 - identity `u.traverse(F, F.of) === F.of(u)`
 - natural transformation `t(u.sequence(F)) === u.traverse(G, t)`

### Examples

```JS
// getById :: Int -> Task e User
const getById = id => // Do some AJAX

// insideOut :: Applicative f
//           => [f a] -> f [a]
const insideOut = (T, xs) => xs.reduce(
  (acc, x) => lift2(append, x, acc),
  T.of([]))

// paralleliseTaskArray
//   :: [Int] -> Task e [User]
const paralleliseTaskArray = users =>
  insideOut(Task, users.map(API.getById))
```

```JS
Array.prototype.traverse =
  function (T, f) {
    return this.reduce(
      //    Here's the map bit! vvvv
      (acc, x) => lift2(append, f(x), acc),
      T.of([]))
  }

// Don't worry, though: `sequence` can also
// be written as a super-simple `traverse`!
const sequence = (T, xs) =>
  xs.traverse(T, x => x)
```

```JS
// Transform _2, then `map` in the _1!
Pair.prototype.traverse = function (_, f) {
  return f(this._2).map(
    x => Pair(this._1, x))
}

// Keep the nothing OR map in the Just!
Maybe.prototype.traverse = function (T, f) {
  return this.cata({
    Just: x => f(x).map(Maybe.Just),
    Nothing: () => T.of(Maybe.Nothing)
  })
}

// Lift all the bits, then rebuild!
Tree.prototype.traverse = function (T, f) {
  return this.cata({
    Node: (l, n, r) => lift3(
      l => n => r =>
        Tree.Node(l, n, r),

      l.traverse(T, f),
      f(n),
      r.traverse(T, f))
    Leaf: () => T.of(Tree.Leaf)
  })
}
```

```JS
// toChar :: Int -> Either String Char
const toChar = n => n < 0 || n > 25
  ? Left(n + ' is out of bounds!')
  : Right(String.fromCharCode(n + 65))

// Right(['A', 'B', 'C', 'D'])
[0,  1,  2,  3].traverse(Either, toChar)

// Left('-2 is out of bounds!')
[0, 15, 21, -2].traverse(Either, toChar)
```


---

## Functors

> Functor types are containers. Not all container types are functors.

```JS
// Any functor must have a `map` method:
map :: Functor f => f a ~> (a -> b) -> f b
```

### Laws

- Composition `fx.map(f).map(g) === fx.map(x => f(g(x))) 
              || u.map(f).map(g) === u.map(x => g(f(x)))
              || U(x).map(id) === U(id(x)) === U(x)`
  
```JS
const exp1 = Box("Squirrels")
  .map(s => s.substr(5))
  .map(s => s.toUpperCase())

const exp2 = Box("Squirrels")
  .map(s => s.substr(5).toUpperCase())

exp1 === exp2
```
- Identity `fx.map(id) === id(fx) 
            || u.map(x => x) === u
            || U(x).map(g).map(f) === U(g(x)).map(f) === U(f(g(x))) === U(x).map(x => f(g(x)))`

```JS
const id = x => x

const exp1 = Box("crayons").map(id)
const exp2 = id(Box("crayons"))

exp1 === exp2
```

```JS
/// Applicative Functors for multiple arguments

Box(x => x + 1).ap(Box(2)) // Box(3)
Box(x => y => x + y).ap(Box(2)).ap(Box(3)) // Box(5)
```

```JS
F(x).map(f) === F(f).ap(F(x))

const liftA2 = (f, fx, fy) => 
  fx.map(f).ap(fy)

liftA2(add, Box(2), Box(4)) // Box(6)
```

```JS
const Just = x => ({
  // Transform the inner value
  // map :: Maybe a ~> (a -> b) -> Maybe b
  map: f => Just(f(x)),

  // Get the inner value
  // fold :: Maybe a ~> (b, a -> b) -> b
  fold: (_, f) => f(x)
})

const Nothing = ({
  // Do nothing
  // map :: Maybe a ~> (a -> b) -> Maybe b
  map: f => Nothing,

  // Return the default value
  // fold :: Maybe a ~> (b, a -> b) -> b
  fold: (d, _) => d
})
```

### Examples

```JS
const $ = selector => 
  Either.of({ selector, height: 10 })

const getScreenSize = (screen, head, foot) => 
  screen - (head.height + foot.height)

$("header").chain(head => 
  $("footer").map(footer => 
    getScreenSize(800, head, footer)))

// With curring

const getScreenSize = screen => head => foot => 
  screen - (head.height + foot.height)

Either.of(getScreenSize(800))
  .ap($("header"))
  .ap($("footer")) // Right(780)

liftA2(getScreenSize(800), $("header"), $("footer")) // Right(780)
```

```JS
// List comprehensions with Applicative Functors

List.of(x => x).ap(List([1,2,3])) // List([1, 2, 3])

List.of(x => y => `${x}-${y}`)
  .ap(List(["T-Shirt", "Sweater"]))
  .ap(List(["M", "S", "L"])) // List(["T-Shirt-M", "T-Shirt-S", "T-Shirt-L", "Sweater-M", "Sweater-S", "Sweater-L])
```

```JS
const DB = {
  find: id =>
    new Task((rej, res) => 
      setTimeout(() => 
        res({ id, title: `Project ${id}` }), 100))
}

const reportHeader = (p1, p2) => 
  `Report: ${p1.title} compared to ${p2.title}`

// Sequential
DB.find(20).chain(p1 => 
  DB.find(8).map(p2 => 
    reportHeader(p1, p2)))

// Parallel
Task.of(p1 => p2 => reportHeader(p1, p2))
  .ap(DB.find(20))
  .ap(DB.find(8))
  .fork(e => "error", x => "success")
```

---

## Contravariant

Same as Functor, however backwards. Like `pipe` and `compose`. `contramap` its like a before hook, when `map` is after

```JS
contramap :: f a ~> (b -> a) -> f b
```

### Laws

- Identity `U.contramap(x => x) === U`
- Composition `U.contramap(f).contramap(g) === U.contramap(x => f(g(x)))`

### Examples

```JS
// f :: String -> Int
const f = x => x.length

// ['Hello', 'world']
['Hello', 'world'].map(f).contramap(f)
```

```JS
// type Predicate a = a -> Bool
// The `a` is the *INPUT* to the function!
const Predicate = daggy.tagged('Predicate', ['f'])

// Make a Predicate that runs `f` to get
// from `b` to `a`, then uses the original
// Predicate function!
// contramap :: Predicate a ~> (b -> a)
//                          -> Predicate b
Predicate.prototype.contramap =
  function (f) {
    return Predicate(
      x => this.f(f(x))
    )
  }

// isEven :: Predicate Int
const isEven = Predicate(x => x % 2 === 0)

// Take a string, run .length, then isEven.
// lengthIsEven :: Predicate String
const lengthIsEven =
  isEven.contramap(x => x.length)
```

> `lengthIsEven` converts a `String` to an `Int`, then to a `Bool`. We don’t care that there’s an `Int` somewhere in the pipeline - we just care about what the input value has to be.

```JS
// type Equivalence a = a -> a -> Bool
// `a` is the type of *BOTH INPUTS*!
const Equivalence = daggy.tagged('Equivalence', ['f'])

// Add a pre-processor for the variables.
Equivalence.prototype.contramap =
  function (g) {
    return Equivalence(
      (x, y) => this.f(g(x), g(y))
    )
  }

// Do a case-insensitive equivalence check.
// searchCheck :: Equivalence String
const searchCheck =

  // Basic equivalence
  Equivalence((x, y) => x === y)

  // Remove symbols
  .contramap(x => x.replace(/\W+/, ''))

  // Lowercase alpha
  .contramap(x => x.toLowerCase())

// And some tests...
searchCheck.f('Hello', 'HEllO!') // true
searchCheck.f('world', 'werld')  // false
```

```JS
const Reducer = run => ({
  run,
  contramap: f => 
    Reducer((acc, x) => run(acc, f(x)))
})

Reducer(login.contramap(pay => pay.user))
  .concat(Reducer(changePage).contramap(pay => pay.currentPage))
  .run({
    user: {},
    currentPage: {}
  })
```

```JS
const Pred = run => ({
	run,
	concat: other => Pred(x => run(x) && other.run(x)),
	contramap: f => Pred(x => run(f(x)))
});

const greaterThanFour = Pred(x => x > 4)
const startFromS = Pred(x => x,startsWith("s"))

const p = greaterThanFour
  .contramap(x => x.length) // Transform string to number for predicate function
  .concat(startFromS)

p.run("Hello")
```

```JS
const extension = file => file.name.split('.')[1]

const matchesAny = regex => str =>
    str.match(new RegExp(regex, 'ig'))

const matchesAnyP = pattern => Pred(matchesAny(pattern)) // Pred(str => Bool)

const pred = file =>
	matchesAnyP('txt|md')
	.contramap(extension)
	.concat(matchesAnyP('functional').contramap(x => x.contents))
	.run(file)

[
  {name: 'blah.dll', contents: '2|38lx8d7ap1,3rjasd8uwenDzvlxcvkc'},
  {name: 'intro.txt', contents: 'Welcome to the functional programming class'},
  {name: 'lesson.md', contents: 'We will learn about monoids!'},
  {name: 'outro.txt', contents: 'Functional programming is a passing fad which you can safely ignore'}
].filter(pred)

```

---

## Apply

> Is just a mechanism for combining contexts (worlds!) together without unwrapping them.

```JS
ap :: Apply f => f a ~> f (a -> b) -> f b
--                 a ->   (a -> b) ->   b
```

> If we ignore the `f`s, we get the second line, which is our basic **function application**: we apply a value of type `a` to a function of type `a -> b`, and we get a value of type `b`. Woo! What’s the difference with `ap`? All those bits are wrapped in the **context** of our `f` functor!

```JS
// compose :: (b -> c) -> (a -> b) -> a -> c
const compose = f => g => x => f(g(x))
```

### Laws 
- composition `x.ap(g.ap(f.map(compose))) === x.ap(g).ap(f)` or `x.map(compose(f)(g)) === x.map(g).map(f)`

### Examples

```JS
// Remember: `f` MUST be curried!
// lift2 :: Applicative f
//       =>  (a ->   b ->   c)
//       -> f a -> f b -> f c
const lift2 = f => a => b =>
  b.ap(a.map(f))

// But, if we write lift3...
const lift3 =
  f => a => b => c =>
    c.ap(b.ap(a.map(f)))
```

If we use functor
```JS
// lift2F :: Functor f
//        => (  a ->   b ->      c)
//        ->  f a -> f b -> f (f c)
const lift2F = f => as => bs =>
  as.map(a => bs.map(b => f(a)(b)))
```

```JS
const Identity = daggy.tagged('Identity', ['x'])

// map :: Identity a ~> (a -> b)
//                   -> Identity b
Identity.prototype.map = function (f) {
  return new Identity(f(this.x))
}

// ap :: Identity a ~> Identity (a -> b)
//                  -> Identity b
Identity.prototype.ap = function (b) {
  return new Identity(b.x(this.x))
}

// Identity(5)
lift2(x => y => x + y)
     (Identity(2))
     (Identity(3))
```

```JS
// Our implementation of ap.
// ap :: Array a ~> Array (a -> b) -> Array b
Array.prototype.ap = function (fs) {
  return [].concat(... fs.map(
    f => this.map(f)
  ))
}

// 3 x 0 elements
// []
[2, 3, 4].ap([])

// 3 x 1 elements
// [ '2!', '3!', '4!' ]
[2, 3, 4]
.ap([x => x + '!'])

// 3 x 2 elements
// [ '2!', '3!', '4!'
// , '2?', '3?', '4?' ]
[2, 3, 4]
.ap([ x => x + '!'
    , x => x + '?' ])
```

```JS
return lift2(x => y => x + y)(array1)(array2)

// ... is the same as...

const result = []

for (x in array1)
  for (y in array2)
    result.push(x + y)

return result
```

---

## Monads

To be a monad type must have `.of` and `.chain` methods (same as `flatMap`, `bind`, `>>==`). `Box`, `Either`, `Task`, `List` is a monads

### Laws

```JS
const join = m => 
  m.chain(x => x)

const m = Box(Box(Box(3)))

join(m.map(join)) === join(join(m))
```

```JS
const m = Box("wonder")

join(Box.of(m)) === join(m.map(Box.of))
```

## Natural Transformations

> Natural transformations is just a type conversion.  It's taking one functor to another. A natural transformation is actually a function that takes a functor holding some a to another functor holding that a. it's a structural change.

### Laws

```JS
nt(x).map(f) == nt(x.map(f))

boxToEither(Box(100)).map(x => x * 2) === boxToEither(Box(100).map(x => x * 2))
```

### Examples

```JS
const eitherToTask = e => 
  e.fold(Task.rejected, Task.of)

eitherToTask(Right("right"))
  .fork(e => console.log("err", e), x => console.log("res", x)) // res right

eitherToTask(Left("left"))
  .fork(e => console.log("err", e),  x => console.log("res", x)) // err left
```

```JS
const boxToEither = b => 
  b.fold(Right)

boxToEither(Box(100)) // Right(100)
```

```JS
List(["hello", "world"])
  .chain(x => List(x.split(""))) // List(["h", "e", ...])
```

```JS
const fake = id => ({
  id,
  name: "user1",
  bestFriendId: id + 1
})

const DB = ({
  find: id => 
    new Task((rej, res) => 
      res(id > 2 ? Right(fake(id)): Left("not found")))
})

DB.find(3) 
  .chain(either => 
    either.map(user => DB.find(user.bestFriend))) // Right(Task(Right(User)))

// Same as above with NT
DB.find(3) 
  .chain(eitherToTask)
  .chain(user => DB.find(user.bestFriend))
```

## Isomorphisms and round trip data transformations

```JS
const Iso = (to, from) => ({
  to, 
  from,
})
```

### Laws

```JS
from(to(x)) === x
to(from(y)) === y
```

### Examples

```JS
// String ~ [Char]
const chars = Iso(s => s.split(""), c => c.join(""))

chars.to("hello") // ["h", "e", "l", "l", "o"]
chars.from(chars.to("hello")) // hello
```

```JS
const truncate = str =>
  chars.from(chars.to(str).slice(0, 3)).concat("...")

truncate("hello") // hel...
```

```JS
// [a] ~ Either null a
const singleton = Iso(e => e.fold(() => [], x => [x]), ([x]) => x ? Right(x) : Left())

const filterEither = (e, pred) =>
  singleton.from(singleton.to(e).filter(pred))

filterEither(Right("hello"), x => x.match(/h/ig))
  .map(x => x.toUpperCase()) // If first argument not match the predicate this line don't run
```

## Real world app examples

### Validation library 

```JS
const Success = (x) => ({
  x,
  isFail: false,
  fold: (f, g) => g(x),
  concat: (other) => (other.isFail ? other : Success(x)),
});

const Fail = (x) => ({
  x,
  isFail: true,
  fold: (f, g) => f(x),
  concat: (other) => (other.isFail ? Fail(x.concat(other.x)) : Fail(x)),
});

const Validation = (run) => ({
  run,
  concat: (other) =>
    Validation((key, val) => run(key, val).concat(other.run(key, val))),
});

const isEmail = Validation((key, val) =>
  !!/@/.test(val) ? Success(val) : Fail([`${key} need to be valid email`])
);

const isPresent = Validation((key, val) =>
  !!val ? Success(val) : Fail([`${key} need to be present`])
);

const validate = (spec, obj) =>
  List(Object.keys(spec)).foldMap(
    (key) => spec[key].run(key, obj[key]),
    Success([obj])
  );

const validation = { name: isPresent, email: isPresent.concat(isEmail) };

const obj = {
  name: '',
  email: 'dadas',
};

const res = validate(validation, obj);
```

### Spotify app

> We form a plan to find the common ground between two artists from the spotify api. Then we sketch out a data flow to ensure we have what we need, when we need it.

```JS
const argv = new Task((rej, res) => res(process.argv))
const names = argv.map(argv => args.slice(2))

const Intersection = xs => ({
  xs,
  concat: ({ xs: ys }) => 
    Intersection(xs.filter(x => ys.some(y => x === y)))
})

const first = xs => 
  Either.fromNullable(xs[0])

const httpGet = url =>
  new Task((rej, res) => 
    request(url, (error, response, body) => 
      error ? rej(error) : res(body)))

const getJSON = url => 
  httpGet(url)
  .map(parse)
  .chain(eitherToTask)

const eitherToTask = e => 
  e.fold(Task.rejected, Task.of)

const parse = Either.try(JSON.parse)

const findArtist = name => 
  getJSON(`http://some-awesome-url-${name}`)
  .map(result => result.artists.items)
  .map(first)
  .chain(eitherToTask)

const relatedArtist = id =>
  getJSON(`http://some-awesome-url-${id}`)
  .map(result => result.artists)

const related = name => 
  findArtist(name)
  .map(artist => artist.id)
  .chain(relatedArtists)

const artistIntersection = rels => 
  rels.foldMap(Intersection).xs

// const artistIntersection = rels1 => rels2 => 
//   Intersection(rels1).concat(Intersection(rels2)).xs

// const main = ([name1, name2]) =>
//   Task.of(rels1 => rels2 => [rels1, rels2])
//   .ap(related(name1))
//   .ap(related(name2))

const main = names => 
  List(names)
  .traverse(Task.of, releted)
  .map(artistIntersection)

names
  .chain(main)
  .fork(console.error, console.log)
```

### Redux type app

```JS
// (acc, a) -> acc
// (a, acc) -> acc
// a -> (acc -> acc)
// a -> Endo(acc -> acc)

// Fn(a -> Endo(acc -> acc))

const login => payload => state =>
  payload.email
    ? {
      ...state,
      loggedIn: true,
    }
    : state

const setPrefs = payload => state =>
  payload.prefs
    ? {
      ...state,
      prefs: payload.prefs
    }
    : state

const reducer = Fn(login)
  .map(Endo)
  .concat(Fn(setPrefs).map(Endo))

const state = {
  loggedIn: false,
  prefs: {}
}
const payload = {
  email: "email@mail.com",
  pass: 123,
  prefs: {
    color: "#000",
  }
}

reducer.run(payload).run(state) // { loggedIn: true, prefs: { color: "#000" } }
```

---
## Resources
- https://egghead.io/courses/professor-frisby-introduces-composable-functional-javascript
- http://www.tomharding.me/fantasy-land/
- https://github.com/fantasyland/fantasy-land#type-representatives
- https://frontendmasters.com/courses/hardcore-js-v2/
- https://frontendmasters.com/courses/hardcore-js-patterns/