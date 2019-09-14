# RegTracks

Human-readable parsing for regular languages, based on railroad diagrams.

See [the specification](./spec.md) for the technical details.

## Regex is great... or is it?

Regex can do some pretty amazing stuff. But it looks horrible:

```js
var url_exp = /^(?:([A-Za-z]+):)?(\/{0,3})([0-9.\-A-Za-z]+)(?::(\d+))?(?:\/([^?#]*))?(?:\?([^#]*))?(?:#(.*))?$/;
```

That lovely bit of work parses a url. Now imagine trying to debug it. Or don't, if you're squeamish...

Regex is powerful, but what's really needed is something with the power and conciseness of regex and the readability of normal code.

## Introducing: RegTracks

RegTracks takes its inspiration from [Railroad Diagrams](https://www.wikipedia.org/wiki/Syntax_diagram).

What sets it apart from regex?

- Readable patterns
- Reusablility

are the two main things. They're demonstrated below.

And, the best thing is, any RegTracks parsers (should) generate a regex pattern that can be used to match text. This maintains efficiency while also retaining the benefits of RegTracks. [TODO remove this]

## Some examples

Here's how you can express regex as RegTracks. Remember that url regex back at the start? Here it is in RegTracks:

```
@url_parser                     # give our pattern a name
||                              # this begins the pattern. Double pipe doesn't allow whitespace in between chars.
start                           # makes sure we're at the start of the string
optionally {                    # braces indicate a block. The whole block is optional, here.
  any a to z,A to Z as scheme   # we match any unicode letter and store it under the identifier 'scheme'
    times forever               # we match letters as many times as possible
  (:)                           # if a character isn't a letter, parsing can continue if the character is a colon
  }
(/) as slash                    # match a forward-slash character and store it as 'slash'
  times 1 to 3                  # this must match anywhere between 1 and 3 times inclusive
any digit,.,-,a to z,A to Z as host     # if we can't match a slash, match any of the specified set
  times forever
optionally {
  (:)
  digit as port                 # digit is a built-in set. There are some other built-in sets.
    times forever
  }
optionally {
  (/)
  optionally none \#,? as path  # match any unicode character that isn't a hash or question mark.
    times forever               # above, hash must be escaped, since it could otherwise denote a comment.
  }
optionally {
  (?)
  optionally none \# as query
    times forever
  }
optionally {
  (\#)
  optionally anything as hash   # anything is another built-in set. It includes everything apart from line-ending
    times forever               # characters.
  }
end                             # this makes sure we're at the end of the string.
||
```

Now it's readable, debuggable, and generally a lot nicer than the alternative.

As can be seen, matches are attempted for rules running from top to bottom. 'Capture groups' are named - no more messing about with accessing numerical indices. Literals are

The indentation is just stylistic - it is recommended for readability, and you can use your own style - but newlines are important for showing the end of a rule and the start of a new one.

What else can RegTracks do? Well, subpatterns and defined character sets can be used to decrease code duplication or increase readability. For example:

```
@parse_list
||
parse_identifier        # we include the parse_identifier pattern here
  times forever
    after (,)           # `after` allows rules to specified that must be matched before repeating
||

@parse_identifier       # we define our parse_identifier pattern here
||
any valid_name as name  # we use a character set defined below
  times forever
or digit as number      # `or` allows another rule or block to be matched instead of the previous one
  times forever
||

@valid_name             # here we define our character set
a to z,A to Z,_
```
