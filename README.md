# RegTracks

RegTracks lets you write readable regex, based on railroad diagrams.

See [the tutorial](./tutorial.md) for a more in-depth guide, and see [the specification](./spec.md) for the technical details.

Contents:
- [Regex is great... or is it?](#regex-is-great-or-is-it)
- [Introducing: RegTracks](#introducing-regtracks)
- [Some examples](#some-examples)
- [Reducing code duplication](#reducing-code-duplication)
- [How do I learn more about how to use RegTracks?](#how-do-i-learn-more-about-how-to-use-regtracks)
- [Available implementations](#available-implementations)
- [Syntax highlighting and extensions](#syntax-highlighting-and-extensions)

## Regex is great... or is it?

Regex can do some pretty amazing stuff. But it looks horrible:

```js
var urlRegex = /^(?:([A-Za-z]+):)?(\/{0,3})([0-9.\-A-Za-z]+)(?::(\d+))?(?:\/([^?#]*))?(?:\?([^#]*))?(?:#(.*))?$/;
```

That lovely bit of work (courtesy of Douglas Crockford) parses a url. Now imagine trying to debug it. Or don't, if you're squeamish...

Regex is powerful, but what's really needed is something with the power and conciseness of regex and the readability of normal code.

## Introducing: RegTracks

RegTracks takes its inspiration from [railroad diagrams](https://www.wikipedia.org/wiki/Syntax_diagram).

What sets it apart from regex?

- Readable patterns
- Reusablility
- Matching under identifiers
- Variables

The philosophy is simple: readable code => fewer bugs => less time spent fixing bugs => better programmes!

## Some examples

Here's how you can express regex as RegTracks. Remember that url regex back at the start? Here it is in RegTracks:

```
@url_parser                     # give our pattern a name
||                              # this begins the pattern.
start                           # makes sure we're at the start of the string
optionally {                    # braces indicate a block. The whole block is optional, here.
  any a to z, A to Z as scheme  # we match any unicode letter and store it under the identifier 'scheme'
    times forever               # we match letters as many times as possible
  (:)                           # if a character isn't a letter, parsing can continue if the character is a colon
  }
optionally (/) as slash         # match a forward-slash character and store it as 'slash'
  times 1 to 3                  # this can match up to 3 times inclusive
any digit, ., -, a to z, A to Z as host     # if we can't match a slash, match any of the specified set
  times forever
optionally {
  (:)
  any digit as port             # digit is a built-in set. There are some other built-in sets.
    times forever
  }
optionally {
  (/)
  optionally none \#, ? as path # match any unicode character that isn't a hash or question mark.
    times forever               # above, hash must be escaped, since it could otherwise denote a comment.
  }
optionally {
  (?)
  optionally none \# as query
    times forever
  }
optionally {
  (\#)
  optionally any anything as hash   # anything is another built-in set. It includes everything apart from line-ending
    times forever               # characters.
  }
end                             # this makes sure we're at the end of the string.
||
```

Now it's readable, debuggable, and generally a lot nicer than the alternative.

As can be seen, matches are attempted for rules running from top to bottom. 'Capture groups' are named - no more messing about with accessing numerical indices.

The indentation is just stylistic - it is recommended for readability, and you can use your own style - but newlines are important for showing the end of a rule and the start of a new one.

## Reducing code duplication

What else can RegTracks do? Subpatterns and defined character sets can be used to reuse code and increase readability. For example:

```
@parse_list
||
parse_identifier        # we include the parse_identifier pattern here
  times forever
    after (,)           # 'after' allows rules to be specified that must be matched before repeating
||

@parse_identifier       # we define our parse_identifier pattern here
||
any valid_name as name  # we use a character set defined below
  times forever
or any digit as number  # 'or' allows another rule or block to be matched instead of the previous one
  times forever
||

@valid_name             # here we define our character set
a to z,A to Z,_
```

Tada! It's not as compact as the regex equivalent, but you can certainly read it.

## How do I learn more about how to use RegTracks?

Please, [read the tutorial](./tutorial.md)!

## Available implementations

At the moment there are only limited implementations of RegTracks:

- [JavaScript](https://github.com/regtracks/regtracks-js) - also through [npm](https://www.npmjs.com/package/regtracks)

- Python - coming soon, I hope :)

Like RegTracks? Make your own parser for _your_ favourite language and get in touch via PR or issue. If it satisifies [the specification](./spec.md), it'll be considered for addition to this list.

If you're making your own, feel free to use the model that the JS one uses and rewrite it in a different language, or just start from scratch if you're feeling brave.

## Syntax highlighting and extensions

To highlight syntax in `.track` files, you can [install the VSCode extension for RegTracks](https://marketplace.visualstudio.com/items?itemName=jamesthistlewood.regtracks).
