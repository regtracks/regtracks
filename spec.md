# The RegTracks Specification

1. A schema is a string or file containing zero or more patterns (see 2) and character sets (see 6), which abides by the rules set out in this specification.

2. A pattern is a list of rules that is contained within with boundary delimeters (see 2.3).

    1. A pattern runs from top to bottom, with rules (see 3) delimited by a newline.

    2. When a pattern is used, each rule is executed sequentially starting from the top of the pattern.

        1. If no valid rule is found for a character at a given point, matching ends and is restarted from the beginning of the invoked pattern (the pattern used as the entry point into the schema), beginning by attempting to match the last unmatched character.

            1. The exception to this is when the rule is succeeded by a pattern delimiter -

    3. A pattern is delimited by either `|`, which allows whitespace to come between characters parsed by the rules (effectively ignoring it), or `||`, which does not allow whitespace.

        1. A pattern must be ended with the same delimiter that it began with.

    4. Empty lines can be ignored when parsing.

    5. Patterns don't necessarily need to be marked with a symbol. It's up to the parser to decide how to handle this.

3. A rule is a line of text which specifies what character is allowed at a certain position. Rules make up patterns.

    1. 'Matching' rules specify which characters can be accepted to allow parsing to progress down through the pattern.

        1. To match a single literal, the literal should be enclosed in the brackets `(` and `)`, e.g. `(a)` matches the character `a`, and `(cheese)` matches the string of characters `cheese`.

            1. Some characters must be escaped throughout a schema, even if appearing in a literal. These are given in appendix B.

        2. To match a set of values, the keyword `any` should be used, followed by a space and then the comma-delimited set that can be matched, e.g. `any h,t,p`.

            1. Whitespace may appear between commas, since escapes and special sets are provided for whitespace (see 3.1.2.1.1 and appendix A).

            2. Single characters need not be enclosed in brackets in a set, but any string literals must be within brackets, e.g. `any h,p,(mouse),9`

                1. The control characters `\n` (newline), `\t` (tab) and `\r` (carriage return) are also accepted as 'characters', and need not be enclosed in brackets.

                2. Unicode characters can be specified with `\uxxxx`, where all of the `x` characters are replaced by the hexadecimal unicode character code. This need not be enclosed in brackets.

            3. A range of characters can be specified using the keyword `to`, seperated by spaces either side from two characters, e.g. `any a to z,0 to 9`.

            4. If a non-bracketed alphanumeric string (allowing underscores) is an element in the set of values, then the symbol with the given name will be looked up. If the symbol refers to a character set, it will be included in the set of values. An error will occur if no such symbol exists or if it refers to a pattern instead of a character set.

                1. Some special sets are built-in, and are given in appendix A.

        3. The keyword `none` can be used in the place of `any` to match anything except for the values specified, e.g. `none f-h,(cat),z` will match all the characters in `mouse` but not all of `scatman`. All the rules given in 3.1.2.x also apply to the set passed to `none`. Line-ending characters will not be matched [TODO].

        4. A single character set that is defined under a symbol elsewhere in the schema (or any built-in set) can be used by itself, e.g. `digit`, `my_charset`. For built-in sets see appendix A.

    2. 'Repetition' rules apply to the last rule or, if coming after a `}`, the last block (see 3.x). They control how many times the rules can be used to match characters.

        1. Repetition rules must follow a matching rule or a block in the scheme flow.

        2. The keyword `times` can be used to specify a repetition of the last rule or block. When reached, matching continues from the top of the last rule or block.

            1. `times` must be followed by a number or range or the keyword `forever`, seperated by a space.

                1. If a single number is given, the last rule or block must match exactly that many times.

                    1. The number must be > 0.

                2. A range is defined by two numbers, a lower bound and upper bound, with the keyword `to` in between seperated by spaces, e.g. `times 1 to 3`. It indicates that the last rule or block must match at least as many times as the lower bound, and cannot match more times than the upper bound.

                    1. The lower bound must be > 0, and the upper bound must be > the lower bound.

                    2. The upper bound may alternatively be the keyword `forever`, defining that something must repeat at least the lower bound amount of times.

                3. If the keyword `forever` is used, the last rule or block will match while it can, but if it fails to match, any rules immediately following the `forever`-repeated rule or block will be matched if possible.

    3. A prefix modifies a rule or block, and is seperated by a space from a rule it precedes.

        1. If used before a single rule, a prefix comes on the same line as a rule, before the main body of the rule. In this case, the prefix applies to the rule only.

        2. If used before a block-opening brace `{`, the prefix applies to the entire block.

        3. The prefix `optionally` allows a rule or block to be skipped. If a rule or block can be matched with this prefix can be matched, it is matched, the highest-up rule being matched first.

        4. The prefix `or` allows a rule or block to be used as a match instead of the previous rule or block. This allows 'branching' as seen in railroad diagrams.

            1. If `or` is used immediately before a rule, it must be a 'matching' rule.

            2. `or` must follow a block or other rule - it can never be attached to the first rule in a block or pattern.

        5. The prefix `after` is specifies a rule that must be matched to allow a 'repetition' rule to repeat back to the start. It must be followed by a single rule or a block, e.g. `after any a-z` or `after { # some pattern goes below`.

            1. The prefix `after` must prefix a rule that follows a repetition rule.

        6. The prefix `optionally` may be used in conjunction with `or` and `after`, but `or` must never be used with `after`.

    4. The suffix `as` specifies that any characters matched should be stored under the identifier given.

        1. `as` can only come after a 'matching' rule or after the end of a block, e.g. `any a to z as name`, where `name` is the identifier.

        2. The same identifier can be used in multiple places. While a pattern is in execution, any characters matching collected with a given identifier should be stored in a string, sequentially. Once a pattern has finished matching, the next time it is executed, any matches under the same identifier should be stored in a seperate string (although still under the same identifier name).

        3. Any strings collected under identifiers should be discarded if a pattern does not reach its final delimiter. Only when a pattern reaches this delimiter should any collected strings be 'saved'.

    5. A brace `{` opens a block. Rules grouped in a block can be prefixed (see 3.3), suffixed (3.4), or used with a repetition rule (3.2).

        1. An opening brace can appear before or after a prefix (see 3.3), but must appear before a rule begins.

        2. An opening brace can only appear before a rule if the rule is a 'matching' rule.

        3. A closing brace must not appear before the end of a rule, with the exception of immediately before a suffix (see 3.4), e.g. `any digit,a-z}`, `} as ident`.

        4. Braces may appear on the same line as other rules, or on their own line, provided they also abide by the rest of the rules in 3.5.

        5. A block must contain at least one 'matching' rule.

4. Comments can be added with `#` after any rule. The comment lasts until the end of the line. Any `#` symbol must therefore be escaped with a backslash `\`, and any backslashes that should be taken literally must be escaped with another backslash `\`. For a list of characters that must be escaped, see appendix B.

5. A symbol is a line beginning with `@`, followed by an alphanumeric string (also allowing underscores `_`), e.g. `@foo_bar` or `@component`. This marks a pattern or character set, allowing it to be used elsewhere in the schema.

    1. A symbol must appear on the line immediately preceding the beginning delimiter of a pattern or a comma-delimited set of values.

    2. Symbols must not appear within patterns.

    3. Some symbol names are reserved and cannot be used. These are any of the names appearing in appendix C.

    4. Symbol names must be at least 2 characters long.

    5. Symbol names cannot start with a number.

[TODO pattern flags?]

6. A character set can be defined outside a pattern, allowing for reuse. It must be a comma-delimited list with the same specification as 3.1.2. Keywords are not allowed in this context.

    1. Character sets must be given a symbol as specified in 5.

7. A variable may be used in a string literal, e.g. `($name)` where `name` is replaced by the name of the variable.

    1. Variables should be passed to the parser by name when before it parses the text.

8. An error will occur if any of these rules are violated during schema parsing.

## Appendix A: built-in sets

| Set name    | Regex equivalent | Description                             |
|-------------|------------------|-----------------------------------------|
|`anything`   |`/./`             | Any character that doesn't end the line |
|`digit`      |`/[0-9]/`         | All digits                              |
|`end`        |`/$/`             | The end of the string                   |
|`space`      |`/ /`             | The space character                     |
|`start`      |`/^/`             | The start of the string                 |
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
