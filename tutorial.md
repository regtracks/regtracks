# RegTracks tutorial

One note before beginning: a schema is a string or file containing some patterns and/or character sets.

- [Patterns](#patterns)
- [Rules](#rules)
  - [Matching rules](#matching-rules)
    - [Literals](#literals)
    - [Sets](#sets)
    - [Pattern references](#pattern-references)
- [Blocks](#blocks)
- [Modifiers](#modifiers)
- [Repeats](#repeats)
- [Collection (capture)](#collection-capture)
- [Character sets](#character-sets)
- [Variables](#variables)
- [Matching strings](#matching-strings)
  - [`match`](#match)
  - [`test`](#test)
  - [`replace`](#replace)
- [Options for matching](#options-for-matching)
  - [`global`](#global)
  - [Regex options](#regex-options)

## Patterns

RegTracks works with patterns. A pattern is a list of rules must be satisfied, in order, for the pattern to match successfully.

```
@factorial         # you can give your pattern a name!
||
any 0 to 9 as digit
  times forever
(!)
||
```

The pattern starts and ends with `||`. This mirrors the way the railroad diagrams are drawn (they start with two vertical lines). Anything in between these symbols are rules.

You can also reuse patterns:

```
@ident
||
name
number
||

@name
||
any a to z, A to Z as name
  times forever
||

@number
||
any digit as number
  times forever
||
```

Here we define two sub-patterns, and use them simply by specifying them by name in the `ident` pattern.

## Rules

A rule is a line in a pattern. Simple as that.

### Matching rules

Matching rules, as the name suggest match characters in some text. When parsing, the pattern tries the first matching rule it finds on the first character. If it works, it continues to the next one. If it fails, whether it continues or not depends on if the rule was optional, or was an alternative to a previous rule - don't worry, you can read more about that later.

Every time a rule matches, the parser moves on to the character after the one that just matched.

#### Literals

The most basic matching rule matches a literal character or string. It must be enclosed in `(`brackets`)`, like that.

For example:

```
(!)
(Name:)
(\#)
```

As can be seen above, when appearing in literals, some characters must be escaped with a backslash `\`. These are `{ } ( ) # $ @ \`. N.B, the backslash must only be escaped when it is meant to appear as a literal backslash.

So, `(!)` succeeds if the character that being parsed when the rule is reached is an exclamation mark `!`.

#### Sets

You can also match a set of characters, using this notation:

```
any a, b, f to z, \u00f5, (foo), 9, space
```

We introduce the matching rule using the keyword `any`, and then follow it with a set of characters or words, separated by commas.

Single characters can just appear by themselves, like `a` and `b` above. You can also use unicode characters using `\uxxxx`, where the `x`s are hexadecimal digits. In this case, it's matching the lovely character õ. The only thing that can't appear as normal is whitespace. You can use `\n`, `\t` and `\r`, as well as the special sets `space` and `whitespace` instead.

You can specify a range of characters using the keyword `to`. This is will accept any character in between the two characters (including both endpoints). This is done using the unicode numbers of each character.

If you want a word, you can specify it using literal notation, in brackets.

`space` is a special set. There are a few of these:

- `space` represents simply the space character ` `
- `anything` is any character except for newline
- `digit` is any character from 0 to 9
- `whitespace` is any whitespace character

You can also use the keyword `none` instead of `any` to match any character apart from the ones you specify.

#### Pattern references

As mentioned above, you can also just reference another pattern defined in the schema by name:

```
my_pattern
```

This will insert the pattern `my_pattern` in its place, and works just like a normal matching rule.

## Blocks

Blocks group rules together, allowing you to apply modifiers to them (see below). For example, you can make a whole block optional:

```
optionally {
  # rules here...
}
```

Or even collect all the matching characters under one identifier:

```
{
  # more rules...
} as thing
```

The same goes for the usage of `or` and `after`.

After opening a block with `{`, there must still be a newline. `}` also requires a newline after it (unless there's `as` after it).

## Modifiers

You can modify matching rules and blocks. If you want a rule to match if possible, but also don't mind if it doesn't match, you can use the keyword `optionally`.

```
optionally any whitespace
any a to z
```

The rules above allow one character of whitespace to come before any lowercase latin letter, but also don't mind if the first character is a letter.

You can also chain rules together with `or`. For example:

```
any a to z, A to Z
or {
  any digit
  (!)
  }
```

This will try to match a letter, but if it can't match a letter it will gladly accept a digit followed by a `!` instead. However, if a letter is matched, a digit cannot be matched after it. It's one or the other.

`not` makes sure that a rule or block _doesn't_ match. If it does match, then matching fails. For example:

```
(abc)
not (12)
any digit
  times 2
```

This matches `abc54` and `abc91` but not `abc12`.

## Repeats

You can let rules and blocks be repeatably matched using the keyword `times`. For example:

```
any a to z, A to Z
  times 2
```

This will match two characters from the set specified.

```
{
  any digit, a, b, c
  (.)
  }
  times 1 to 4
```

In this example, we repeat an entire block! This will match anywhere between 1 and 4 times - there must be least 1 match of the block, but no more than 4 matches.

You can also use `times forever` if you want something to repeat as many times as possible:

```
any 1, 3, 5, !
  times forever
```

This will match any character from the set until it finds something that doesn't match. It must match at least once, however. Once it can't match, it will move onto the next rule.

`forever` can also be used as the end of a range:

```
any whitespace
  times 4 to forever
```

This requires it to match at least 4 times, but there is no upper bound.

## Collection (capture)

Similar to regex's capture groups, you can specify that any matches should be stored under an identifier using the keyword `as`. For example:

```
any a to z as lowercase
  times forever
none ., ?, ! as separator
{
  (?)
  any a to z, A to Z, digit, _
  } as ident
or my_pattern as foo
```

Anything that matches and as collected will be available to access under the identifier specified (e.g. `lowercase`). Entire blocks can also be collected. This means a single rule can come under multiple idents and will as such be collected for all of them.

## Character sets

Character sets (or charsets) are, well, sets of characters. You can define one like this, outside of a pattern:

```
@my_set
a, b, (cheese), 1 to 5, \t
```

It's exactly the notation that follows a matching rule with `any`, just without the `any` or `none`.

You can then use it like this, for example:

```
any my_set
none x to z, my_set
```

## Variables

You can use a variable anywhere where you'd use a literal (a word or character enclosed in `(`brackets`)`). You need to specify the name of the variable after a `$` symbol, for example:

```
($myName)
any a to z, ($delim) as foo
```

If you have collected something (using `as`) under a certain identifier, you can use `$name` where `name` is the identifier to reference it. For example:

```
any a to z, A to Z as ident
  times 3
(-)
($ident)
```

would match `abc-abc` and `FGc-FGc`.

If you haven't collected anything under a given identifier, then your RegTracks parser will allow you to pass the default value of any variables to it before matching.

## Matching strings

You can match strings using one of `match` or `test`, which are called 'matching methods'. They will be available once you have a 'track', which can be created using a schema.

### `match`

This method takes two required parameters: the text to attempt to match, and the entry point into the schema (i.e. the name of the pattern to use).

It tries to match the text. If it matches, it will return an object (or object-like structure) with the values:

- match: the characters that were matched, as a string
- index: the index of the first character in the match
- collected: another object-like structure, with the keys being collection idents and the values being the strings collected under the relevant ident

If no match can be made, a `null`-like will be returned.

### `test`

`test` works exactly like `match`, except that it returns only `true` or `false`, depending on whether a match was made.

### `replace`

`replace` also works exactly like `match`, except that it will return the string with any matches replaced by a given replacement string.

## Options for matching

Your RegTracks parser will allow you to pass some options to it. These are:

### `global`

If `global` is set to true, then after a match is made, the next time you use the same pattern it will start searching from after the last match. Otherwise, the parser will stop after the first match and start searching from the beginning of the string each time.

### Regex options

Some options aren't included in RegTracks for design reasons:

- `i` (ignore case), because this really only works for letters of the latin alphabet, and it can be better to have case insensitivity explicitly shown with something like `a to z, A to Z`.

- `m` (multiline), because in RegTracks all matches are multiline

- `s` (dotAll), because it's an unnecessary complication

- `u` (unicode), because all unicode is allowed in RegTracks schemas

- `y` (sticky), because if you want to start from a certain index, just shorten your string
