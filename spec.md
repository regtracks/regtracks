# The RegTracks Specification

Version 1.0.0 (2019-10-10)

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## 1

A schema is a string or file containing zero or more patterns and character sets, which abides by the rules set out in this specification.

### 1.1

Raw unicode may appear in a schema string and must be parsed without errors.

## 2

A pattern is a list of rules that is contained within with boundary delimeters.

### 2.1

An 'invoked pattern' is the one which the user uses as the entry point into the schema.

### 2.2

Rules are delimited by a newline.

### 2.3

When a pattern is used, each rule is executed sequentially starting from the top of the pattern.

#### 2.3.1

If no valid rule is found for a character at a given point, matching ends and is restarted from the beginning of the invoked pattern (the pattern used as the entry point into the schema), beginning by attempting to match the character after the last character that matching was started from.

### 2.4

A pattern is delimited by  `||` at the beginning and end of the pattern.

### 2.5

Empty lines should be ignored when parsing.

### 2.6

Patterns may be marked with a symbol. It's up to the parser to decide how to handle the case where a pattern is not marked.

## 3

A rule is a line of text which specifies what character is allowed at a certain position. Rules make up patterns.

### 3.1

'Matching' rules specify which characters can be accepted to allow parsing to progress down through the pattern.

#### 3.1.1

To match a single literal, the literal must be enclosed in the brackets `(` and `)`, e.g. `(a)` matches the character `a`, and `(cheese)` matches the string of characters `cheese`.

Some characters must be escaped throughout a schema, even if appearing in a literal. These are given in appendix B.

Rules 3.1.2.2.1-2 also apply to string literals.

#### 3.1.2

To match a set of values, the keyword `any` must be used, followed by a space and then the comma-delimited set that can be matched, e.g. `any h,t,p`.

#### 3.1.2.1

Since escapes and special sets are provided for whitespace (see appendix A), whitespace may appear between commas, however it has no semantic value.

#### 3.1.2.2

Single characters may be enclosed within brackets in a set, but any string literals must be within brackets, e.g. `any h,(p),(mouse),9` is valid.

#### 3.1.2.2.1

The control characters `\n` (newline), `\t` (tab) and `\r` (carriage return) are also accepted as 'characters', and follow the rule 3.1.2.2.

#### 3.1.2.2.2

Unicode characters are specified with `\uxxxx`, where all of the `x` characters are replaced by the hexadecimal unicode character code. Characters specified in this way follow 3.1.2.2 as well.

#### 3.1.2.3

A range of characters must be specified using the keyword `to`, seperated by spaces either side from two characters, e.g. `any a to z, 0 to 9`.

#### 3.1.2.4

If a non-bracketed alphanumeric string (allowing underscores) is an element in the set of values, then the symbol with the given name must be looked up. If the symbol refers to a character set, it must be included in the set of values. An error must occur if no such symbol exists or if it refers to a pattern instead of a character set.

Some special sets are built-in, and are given in appendix A.

#### 3.1.2.5

Some non-whitespace value must appear between commas in a character set.

#### 3.1.2.6

The keyword `none` may be used in the place of `any` to match anything except for the values specified, e.g. `none f to h,(cat),z` must match all the characters in `mouse` but not all of `scatman`. All the rules given in 3.1.2.x also apply to the set passed to `none`. Line-ending characters will also be matched.

#### 3.1.2.7

A single character set that is defined under a symbol elsewhere in the schema (or any built-in set) may be used by itself within a comma-delimited set, e.g. `any digit`, `none my_charset`. For built-in sets see appendix A.

#### 3.1.3

To match the start or end of the string, the keywords `start` and `end` must be used respectively.

#### 3.1.4

An entire pattern declared elsewhere in the schema may be used as a matching rule, and can have prefixes and suffixes applied to it. The syntax for this just the name of the pattern, for example: `my_pattern` or `optionally my_pattern as ident`.

### 3.2

'Repetition' rules apply to the last rule or, if coming after a `}`, the last block. They control how many times the rules can be used to match characters.

#### 3.2.1

Repetition rules must follow a matching rule or a block in the scheme flow.

#### 3.2.2

The keyword `times` may be used to specify a repetition of the last rule or block. When reached, matching continues from the top of the last rule or block.

#### 3.2.2.1

`times` must be followed by a number or range or the keyword `forever`, seperated by a space.

#### 3.2.2.1.1

If a single number is given, the last rule or block must match exactly that many times.

#### 3.2.2.1.1.1

The number must be > 0.

#### 3.2.2.1.2

A range is defined by two numbers, a lower bound and upper bound, with the keyword `to` in between seperated by spaces, e.g. `times 1 to 3`. It indicates that the last rule or block must match at least as many times as the lower bound, and must not match more times than the upper bound.

#### 3.2.2.1.2.1

The lower bound must be > 0, and the upper bound must be > the lower bound.

#### 3.2.2.1.2.2

The upper bound may alternatively be the keyword `forever`, defining that something must repeat at least the lower bound amount of times.

#### 3.2.2.1.3

If only the keyword `forever` is used, the last rule or block must match while it can, but if it fails to match, any rules immediately following the `forever`-repeated rule or block must be matched if possible.

#### 3.2.2.1.3.1

A rule repeated `forever` must match at least once unless it is optional.

### 3.3

A prefix modifies a rule or block, and is seperated by a space from a rule it precedes.

#### 3.3.1

If a prefix is used immediately before a rule, it must be a 'matching' rule.

#### 3.3.2

If used before a single rule, a prefix comes on the same line as a rule, before the main body of the rule. In this case, the prefix applies to the rule only.

#### 3.3.3

If used before a block-opening brace `{`, the prefix applies to the entire block.

#### 3.3.4

The prefix `optionally` allows a rule or block to be skipped. If a rule or block can be matched with this prefix can be matched, it is matched, the highest-up rule being matched first.

#### 3.3.5

The prefix `or` allows a rule or block to be used as a match instead of the previous rule or block. This allows 'branching' as seen in railroad diagrams.

#### 3.3.5.1

`or` must follow a block or other rule - it can never be attached to the first rule in a block or pattern.

#### 3.3.6

The prefix `after` specifies a rule that must be matched to allow a 'repetition' rule to repeat back to the start. It must be followed by a single rule or a block, e.g. `after any a-z` or `after { # some pattern goes below`.

This means that:

```
(a)
  times forever
    after (b)
```

should be equivalent to:

```
(a)
{
  (b)
  (a)
  }
  times forever
```

#### 3.3.6.1

The prefix `after` must prefix a rule that follows a repetition rule.

#### 3.3.7

The prefix `optionally` may be used in conjunction with `or` and `after`, but `or` must not be used with `after`.

### 3.4

The suffix `as` specifies that any characters matched should be stored under the identifier given.

#### 3.4.1

`as` must come after a 'matching' rule or after the end of a block, e.g. `any a to z as name`, where `name` is the identifier.

#### 3.4.1.1

The identifier must be at least one character long and alphanumeric (allowing underscores).

#### 3.4.2

The same identifier can be used in multiple places. While a invoked pattern is in execution, any characters matching collected with a given identifier should be stored in a string, sequentially. Once it finishes a successful match, they should be saved. The next match must start afresh.

#### 3.4.3

Any strings collected under identifiers should be discarded if a pattern does not reach its final delimiter. Only when a pattern reaches this delimiter should any collected strings be 'saved'.

#### 3.4.4

If `as` is used on a rule within a block that also has `as` used on it, the matched characters must be stored under both identifiers.

### 3.5

A brace `{` opens a block. Rules grouped in a block may be prefixed, suffixed, or used with a repetition rule.

#### 3.5.1

An opening brace may appear before or after a prefix, but must appear before a rule begins.

#### 3.5.2

An opening brace may only appear before a rule if the rule is a 'matching' rule.

#### 3.5.3

A closing brace must not appear before the end of a rule, with the exception of immediately before a suffix, e.g. `any digit,a-z}` or `} as ident`.

#### 3.5.4

Braces may appear on the same line as other rules, or on their own line, provided they also abide by the rest of the rules in 3.5.

#### 3.5.5

A block must contain at least one 'matching' rule.

## 4

Comments may be added with `#` after any rule. The comment lasts until the end of the line. Any `#` symbol must therefore be escaped with a backslash `\`, and any backslashes that should be taken literally must be escaped with another backslash `\`. For a list of characters that must be escaped, see appendix B.

## 5

A symbol is a line beginning with `@`, followed by an alphanumeric string (also allowing underscores `_`), e.g. `@foo_bar` or `@component`. This marks a pattern or character set, allowing it to be used elsewhere in the schema.

### 5.1

A symbol must appear precede the beginning delimiter of a pattern or a comma-delimited set of values, with no rule or symbol coming between it and the start of the pattern or set.

### 5.2

Symbol definitions must not appear within patterns.

### 5.3

Some symbol names are reserved and must not be defined by a user. These are any of the names appearing in appendix C.

### 5.4

Symbol names must be at least 2 characters long.

### 5.5

Symbol names must not start with a number.

### 5.6

A given symbol name must not be defined more than once.

## 6

A character set can be defined outside a pattern, allowing for reuse. It must be a comma-delimited list with the same specification as 3.1.2. Keywords other than `to` are not allowed in this context.

### 6.1

Character sets must be given a symbol as specified in 5.

### 6.2

A charset must not include itself in its list of matching criteria.

## 7

A variable may be used in the format string literal, e.g. `($name)` where `name` is replaced by the name of the variable. This may be used any place where a string literal could otherwise appear.

### 7.1

Variables should be passed to the parser by name when before it parses the text.

#### 7.1.1

If a variable is found but no variable with its name has been provided, an error must occur.

### 7.2

A variable name must be alphanumeric (allowing underscores).

## 8

Matching of strings must run according to this specification. Matching should be invoked by the user with one of two methods `match` and `test`, passing a string and options to the invoked method.

### 8.1

A string parser, called a 'track', must be created from a schema.

#### 8.1.1

Matching methods should be attached to a track.

### 8.2

`match` must parse the string according to the schema and return an object with keys and values:

- `match`: the substring which was matched
- `index`: the integer location of the start of the match
- `collected`: an object with collected characters, with the key being an identifier and the value the value associated with that identifier

If no match is made, `null` or a language's equivalent should be returned.

### 8.3

`test` must parse the string according to the schema, and return `true` if a match is made or `false` if no match is made.

### 8.4

Options may be passed to the matching method. These are specified in this section.

#### 8.4.1

`global` is an option that determines what happens after a match is made. If set to `true`, after match is made the index of the first character after the match should be saved and used as the starting point for the next time a matching method is invoked. If set to `false` or not specified, matching starts from the beginning of the string each time.

## 9

An error must occur if any of these rules are violated during schema parsing or string matching.


## Appendix A: built-in sets

| Set name    | Regex equivalent | Description                             |
|-------------|------------------|-----------------------------------------|
|`anything`   |`/./`             | Any character that doesn't end the line |
|`digit`      |`/[0-9]/`         | All arabic numerals                     |
|`space`      |`/ /`             | The space character                     |
|`whitespace` |`/\s/`            | Any whitespace                          |

## Appendix B: list of characters that must be escaped

| Character(s) |
|--------------|
|`#`           |
|`{` `}`       |
|`(` `)`       |
|`\`           |
|`@`           |
|`$`           |

## Appendix C: reserved names that cannot be used for symbols

| Name         |
|--------------|
|`after`       |
|`as`          |
|`any`         |
|`anything`    |
|`digit`       |
|`end`         |
|`forever`     |
|`none`        |
|`optionally`  |
|`or`          |
|`space`       |
|`start`       |
|`times`       |
|`to`          |
|`whitespace`  |
