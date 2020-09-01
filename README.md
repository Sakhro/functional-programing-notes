# Functional programing notes with code examples

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
  map: f => Right(f(x)),
  fold: (f, g) => g(x),
  inspect: () => `Right(${x})`
})

const Left = x => ({
  chain: f => f(x),
  map: f => Left(x),
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

## Semigroup

> Semi-groups are simply a type with a concat method that are associative

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

...
First.empty = () => First(Left())
```

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

## Functors

### Laws

```JS
fx.map(f).map(g) === fx.map(x => f(g(x)))

const exp1 = Box("Squirrels")
  .map(s => s.substr(5))
  .map(s => s.toUpperCase())

const exp2 = Box("Squirrels")
  .map(s => s.substr(5).toUpperCase())

exp1 === exp2
```

```JS
fx.map(id) === id(fx)

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

## Real world app example

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

---
### Resources
- https://egghead.io/courses/professor-frisby-introduces-composable-functional-javascript