




# Brainstorming better i18n and copywriting

I am particularly interested in creating a kick-ass i18n tool for [meteor](http://meteor.com) but these concepts are by no means intrinsically coupled to it

----------------------------

#### Example file: (i.e.: `en.yaml`)

```yaml

location:        Canada

welcome-message: Welcome back {first-name}.

trees-planted:   {count} tree(s) planted in {year}

users-logged-in:
  0:   No one is logged in
  1-3: There (are) {count} (people) logged in
  4-x: {object.1.first-name}, {object.2.first-name}, and {count-rest} other(s) (are) logged in
  12:  There are a dozen people logged in
  x:   not needed!

users-logged-in-alt:
  object-name: people
  0:   No one is logged in
  1-2: There (are) {count} (people) logged in
  3-x: {people.1.first-name}, {people.2.first-name}, and {count-rest} other (people) are logged in
  12:  There are a dozen people logged in
  x:   not needed!
```

#### Using example file:
```
# text

i18n \location
> Canada



# text with interpolations

i18n \welcome-mesage, {first-name:"Mamoru"}
> Welcome back Mamoru.
                 ^



# text with plurals

i18n \trees-planted, {object:[{...}, +42 more], year:2009}
> 42 trees planted in 2009
  ^                    ^




# count-dependent text

# Commented above each of the following return values is the
# corresponding value that users-logged-in would have had to been

i18n \users-logged-in, {object:users-logged-in}

# []
> No one is logged in


# [{first-name:"Hideo"}]
> There is 1 person logged in
        ^  ^   ^

# [{first-name:"Hideo"}, {first-name:"Felini"}]
> There are 2 people logged in
         ^  ^   ^

# [{first-name:"Ozzu"}, {first-name:"Felini"}, {first-name:"Ridley"}]
> Ozzu, Felini, and 1 other person are logged in
                    ^         ^

# [{first-name:"Hideo"}, {first-name:"Ozzu"}, {first-name:"Felini"}, {first-name:"Ridley"}]
> Hideo, Ozzu, and 2 other people are logged in
                   ^         ^

# [{first-name:"Hideo"}, +11 more]
> There are a dozen people logged in
```

#### Particularly unresolved concepts:
```
# how could we achieve this return value?:
i18n \users-logged-in, {object:[{first-name:"Hideo"}, {first-name:"Ozzu"}, +12 more]}
> Hideo, Ozzu, and a dozen other people are logged in
                   ^   ^

# middleware?

i18n \trees-planted, {object:[{...}, +42 more], year:2009}, {named-numbers:on}
> forty-two trees planted in two-thousand-and-nine                         ^
      ^                               ^

i18n \trees-planted, {object:[{...}, +42 more], year:2009}, {named-numbers:[\count]}
> forty-two trees planted in 2009                                              ^
      ^


```


----------------------------
# Summary:

- human goals:
  * fun for us to make, fun for us to use
  * become a defacto-go-to i18n solution (or else what's the point?)
- technical goals:
  * human-parseable i18n syntax
  * as little magic as possible
  * light enough to get started
  * powerful enough to keep going
  * small core with middleware infrastructure
- local `count` can be derived from arg key `object` via `object.length`
- wrap plural-sensitive grammar in paraenthesis `(...)`
- wrap expressions in curly-brackets `{...}`
- count-dependent text key matches priority: `<num>`   `/ > /`   `<num>-<num>` `/ > /` `<num>-x` `/ > /` `x-<num>` `/ > /` `x`


#### special scope variables
* `object`     from args key `object:` / `undefined`
* `count`      from args key `count:` / calculated via `object.length` if `object` exists / `undefined`
* `count-rest` from local `count` - number of times *different* `object.<num>` keys were accessed / `undefined`

#### i18n dictionary formats
* Simple text: `<string>: <string>`
* Count-sensitive text `<string>: <dictionary>`
  * special keys: `<num>` / `<num>-<num>` / `x` / `<num>-x` / `x-<num>`
