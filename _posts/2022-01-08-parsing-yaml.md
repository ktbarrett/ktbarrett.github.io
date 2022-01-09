---
layout: post
title: Complexity in YAML
---

I was recently skimming [Drew Devault's blog](https://drewdevault.com/) and came across [a wish-list for a YAML replacement](https://drewdevault.com/2021/07/28/The-next-YAML.html).
I tend to agree with what he is saying.
One of his arguments against YAML is the complexity of the language.
YAML being complex leads to higher chance of bugs or exploits, and many of those complex features are not often *-- if ever --* used in the wild.

I also recently interacted with [Patrick Lehmann](https://github.com/Paebbels) who mentioned he was interested in writing a YAML parser for VHDL.
YAML is a complex language and writing a fully-featured parser in VHDL would be a Herculean task.
If there were a simpler language that offered the same benefits of YAML, without the complexity, without the unused and dangerous "features", that might be a more reasonable target for a parser written in VHDL.

So I took this as a challenge.
I set out to create a language that was both effective at both program-to-program and programmer-to-program data interchange (AKA configuration), that was simpler than YAML.
My first thought was that I could create a language that was a superset of JSON, which is an reasonably effective program-to-program data interchange;
and a subset of YAML, which is an effective configuration language that also makes up for the few holes in JSON's design.
This would allow existing YAML parsers to load this new language.
It would also allow users to easily port existing YAML files to this new format.

To best understand what subset to select to prevent choosing features which are difficult to parse,
I first had to understand what about YAML made it difficult to parse.
This article explores the difficulty in parsing YAML.

## Plain Scalars

Syntactic noise sucks.
It's a visual manifestation of accidental complexity.
The YAML authors seemed to think that delimiting strings with quotes was unnecessary syntactic noise.
So in YAML any sequence of characters that isn't punctuation or a literal is a string.
While this is visually pleasing, this really does a number on language complexity.
As Drew Devault pointed out, this makes determining document schema difficult,
because some strings aren't strings, [sometimes they are `false`](https://medium.com/@lefloh/lessons-learned-about-yaml-and-norway-13ba26df680).
If I know anything about programmers, it's that they love special cases...

Some of those special cases on plain scalars:
* Cannot contain control codes (probably a good thing).
* Cannot contain escape codes.
* Can contain the `\` character as long as it isn't followed by something that could be parsed as an escape code.
* Cannot start with the characters `[`, `]`, `{`, `}` `!`, `@`, `#`, `%`, `&`, `*`, `|`, `,`, `"`, and `'`, but they are allowed in the rest of the string.
* Can start with `-`, `?`, and `:`, but only if they are not immediately followed with a space.
* Cannot contain the sequence `: `.
* Plain scalars inside flow sequences cannot contain `[`, `]`, `{`, `}`, or `,` at all, otherwise it would confuse the parser (yes, even parsing strings is context-dependent in YAML).
* Anything that correctly parses as a number is a number.
* Anything that correctly parses as a date is a date.
* Anything that *looks* like a number or date, but doesn't correctly parse is a string (`+.00101e4` is a string).
* Even if it is a valid number or date, if it isn't followed by optional whitespace then punctuation, it will continue to be parsed as a plain scalar (`-123 bad` is a string).
* Can't be a literal of another type, including but not limited to: `null`, `NULL`, `~`, `True`, `false`, `FALSE`, `+.inf`, `.Inf`, `.NaN`, and many different capitalization forms of those words.
* In YAML < 1.2 strings in the form `"[0-9]"+ (":" "[0-5]"? "[0-9]")+` are sexagesimal number literals, which probably includes your MAC address.
* In YAML < 1.2 the scalars `yes`, `no`, `on`, and `off` are bools.

All this to avoid a couple quotation marks...

This does complicate the parser, predictive parsers don't work well here.
You have no idea when you see a `-` if it should be parsed as a list, a number, a string, or an error;
and deciding that requires unlimited lookahead.
Instead it is better to consume everything between two punctuation marks and then post-process the string to decide what it is.

I understand this is what the YAML authors were going for.
Scalar values can be parsed in any "schema", including JSON-compatible schemas, and extended schemas.
I'm not entirely sure if there is anyone taking advantage of this feature.
You can load YAML in a schema-less format in some YAML parsers (like PyYAML and ruamel.yaml) using the "base" loader.

## Significant Indentation and Blocks

Significant indentation is a great way cut down on syntactic noise (block-end delimiters being "noise" is debatable).
The popularity of YAML vs JSON as a configuration language shows that people tend to prefer it.
Significant indentation has been seen before YAML in programming languages like Python.
It's a fairly well understood problem in terms of parsing complexity, but it deserves a few words, especially since YAML does take it a step farther than languages like Python do.

Significant indentation is a context-dependent language feature;
however, that complexity can be hand-waved away with indescribable INDENT rules in your EBNF or PEG grammar.
This is typically manifested as a lexer which maintains state to produce INDENT tokens, which are consumed by an otherwise typical parser.

In Python, the lexer must maintain a stack of the current indentation level of the source code.
Increases in indentation compared to the previous non-empty line of code produces an INDENT token.
Decreases in indentation compared to the previous non-empty line of code produces potentially multiple DEDENT tokens,
depending on how many indentation levels are being ended on the same line of code.
And all this INDENT/DEDENT logic also needs to be turned off when inside parentheses or brackets.

However, YAML is a bit different and more difficult; see the following example for why.

```yaml
- - a: 1
    b: 2
  -
    a: 3
```

Multiple blocks can start on the same line, meaning that block indentation is not based on whitespace count before the start of the line,
but on the character position in the line of the first element of the block.
So Python's INDENT and DEDENT tokens are not quite sufficient.

This problem is compounded by the fact that it is also more difficult to decide you are starting a block.
This is because of the way block mappings are written: you don't know you are starting a new mapping block or just parsing a scalar until you see `: `.
So you can't generate an INDENT or some kind of BLOCK_START token unless you first lookahead to find a `: ` token, which requires unlimited lookahead.
Or you instead wait until you are done lexing a scalar and lookahead to see the `: ` sequence to emit a BLOCK_START before emitting the scalar.
Or one of many other possible workarounds.

## Special Cases on Blocks

There are also some special cases with nesting blocks in YAML:

* You can't start a new block mapping or block sequence on the same line a block mapping is started.
* Although, block scalars are fine.
* You don't have to indent a block sequence nested in a block mapping.
* But you can only do this once before the rules change.

```yaml
a: |
  ok

- |
  ok 

b: - invalid

c: d: invalid

- a: ok

- - ok     

b:
- this is ok

-
a: this is not ok

b:
- this is:   # notice start of block
- not ok
# specifically this is interpreted {"b": [{"this is": null}, "not"]}
```

## Block Scalars 

Block scalars are, along with significant indentation, one of the nicest features of YAML.
Drew Devault also notes this in his pros for YAML, asserting that it is one of the reason that YAML still dominates CI configurations:

> **Easily embeds documents written in other formats.** This is the chief reason that YAML still dominates in CI configuration: the ability to trivially write scripts directly into config file, without escaping anything or otherwise molesting the script.

However, block scalars are also a source of language complexity.

There are two kinds of block scalars in YAML, literal (`|`) and folded (`>`).
Literal keeps newlines as they are, while folded replaces newlines on adjacent non-empty lines with spaces.
In addition there are mixin behaviors that allow the user to change where the indentation of the block starts and how to handle trailing newlines.
There a number of features here, and of course only a couple are ever used.

The indentation number is necessary because the indentation of the first non-empty line in the block scalar is used to determine what the indentation for the rest of the block is.
If you needed to start your literal with spaces, you are SOL.
So a number was added to allow the user to say where the literal starts.
This solution is fragile to changes in indentation in the file (e.g. going from 4 spaces to 2).

The chomping operator exists because there is no way to distinguish intended empty lines that are a part of the literal, and spacing between piece of your YAML file.
The default chomping operator leaves 1 newline at the end of the string of your literal.
The `+` chomping operator leaves however many newlines there are before the DEDENT.
If you need somewhere in between all or one, you're still SOL.

To prove these features are accidental complexity due to the design, lets consider an alternative design and see if it has comparable complexity.
Consider the following proposed syntax:

```
a:
  | for i in range(20):
  |     if i % 2 == 0:
  |     
  |         print(i)
  |
  |

b:
  > Merge lines like
  > is done in markdown
  > and LaTeX.
  >
  > But only on adjacent
  > lines.
```

This syntax removes the need to use the indentation number;
and the need for the `+` chomping operator, while also being more flexible.
It is also more readable for long literals when the starting indentation goes off the page and you can't so easily see what your literal's indentation is.
The existence of this alternative syntax may prove that the language complexity of block scalars is accidental and unnecessary.

## Anchors and Aliases

Anchors and aliases seemed to be designed to prevent code duplication.
Why this is needed in a configuration language is unknown to me[^2].
This seems to be an attempt by the YAML authors to provide language features to compete with XML.
Good idea or not, it is susceptible to the "Billion Laughs" attack seen below (code taken from [Wikipedia](https://en.wikipedia.org/wiki/Billion_laughs_attack#Variations)).

```yaml
a: &a ["lol","lol","lol","lol","lol","lol","lol","lol","lol"]
b: &b [*a,*a,*a,*a,*a,*a,*a,*a,*a]
c: &c [*b,*b,*b,*b,*b,*b,*b,*b,*b]
d: &d [*c,*c,*c,*c,*c,*c,*c,*c,*c]
e: &e [*d,*d,*d,*d,*d,*d,*d,*d,*d]
f: &f [*e,*e,*e,*e,*e,*e,*e,*e,*e]
g: &g [*f,*f,*f,*f,*f,*f,*f,*f,*f]
h: &h [*g,*g,*g,*g,*g,*g,*g,*g,*g]
i: &i [*h,*h,*h,*h,*h,*h,*h,*h,*h]
```

Kubernetes' and Github Actions' configurations do not support anchors and aliases for exactly this reason.

## Tags

Tags are a way to mark a node to be constructed into a destination-native type.
Tags can be applied to *any* node: scalar, block, or flow style.
A secondary feature, TAG directives, were added to ease their use.
I have yet to see any examples of global URI tags and TAG directives being used in the wild.
This feature also seems to have been added by the YAML authors so that YAML could compete with XML.
The YAML cheat sheet does a pretty good job at showing the uses of tags.

```yaml
Tag property: # Usually unspecified.
  none    : Unspecified tag (automatically resolved by application).
  '!'     : Non-specific tag (by default, "!!map"/"!!seq"/"!!str").
  '!foo'  : Primary (by convention, means a local "!foo" tag).
  '!!foo' : Secondary (by convention, means "tag:yaml.org,2002:foo").
  '!h!foo': Requires "%TAG !h! <prefix>" (and then means "<prefix>foo").
  '!<foo>': Verbatim tag (always means "foo").
```

Most languages support a common set of primitive data types and data structures, that are generally sufficient to describe any kind of data.
This set of types is seen again and again in JSON, YAML *without tags*, Python, Lua, Go, etc. etc.
Differentiating type when multiple types are supported in a given location can be serviced by the idiom of using an additional layer of mapping to associate a tag with a value[^3];
tags are not necessary for this.

Application-specific schemas -- as opposed to document-specific schemas -- are likely sufficient for configuration tasks.
[JSON Schema](https://json-schema.org/learn/) is proof that describing and validating schema does not need to built into the core language.

I hope this proves that tags are a complexity, and an unnecessary complexity, for the task of YAML acting as either program-to-program or programmer-to-program data interchange format.

## Conclusions

YAML is a complex language, especially when compared to JSON.
And most of this complexity is found in unneeded and often unused features;
or found in accidental complexity due to design decisions.
It is certainly possible to carve out a simple subset of YAML, but a better solution might be found by breaking compatibility (especially wrt. block scalars).

## Footnotes

[^2]: Data compression algorithms would be a better form of compression over the wire. However, it can theoretically help the speed of serialization/deserialization by having the dumper/loader dump/load an anchor and alias pair when the same reference is seen multiple times, instead of spending the time to duplicate the value.

[^3]: See the [ReadTheDocs configuration](https://docs.readthedocs.io/en/stable/config-file/v2.html#python-install). They use a mapping to differentiate the kinds of installation requirements: requirements file or pip dependency string.
