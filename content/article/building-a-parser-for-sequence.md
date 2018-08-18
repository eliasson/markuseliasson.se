+++
Categories = ["Software Development", "Code"]
Description = "Now that we have defined what we want the Sequence language to look like, we need to formalize this in a grammar and then build a parser to parse our input into an Abstract Syntax Tree"
Tags = ["Software Development", "Sequence", "DSL"]
date = "2018-08-18T13:00:00+01:00"
title = "Building a parser for Sequence"
draft = true
+++

Previously, in [Defining the Sequence language](/article/defining-the-sequence-language/) we drafted a version
of the **Sequence** language that were happy with, now we need to formalize this language and build a parser for it.

The language, as we left it looks like:

```
Name "Sequence demo"

Actor User
Object Alpha
Object Bravo

Sequence Test
    User ask Alpha "A synchronous message"
    Alpha tell Bravo "An asynchronous message"
    Bravo replies Alpha "A response message"
    Alpha replies User """
        This is a multi-line
        message text!
        """
```

_For new readers, Sequence is a tiny domain specific language used to describe sequence diagrams. I use it as basis
for a series of blog posts covering how to build such a tool together with a Visual Studio Code extension. See the
initial post [here](/article/building-your-own-dsl/)._


## Scanning, lexing, tokenizing and parsing

There are many different terms used for the process of getting from a stream of characters (our source code) into a structure
that reflects the grammar of our language. **Scanning** or **lexing** is the first part of this process - converting the stream
of characters into a stream of **tokens**. This might also be referred to as _lexical analysis_, or by some, _tokenizing_.

Tokenizing works by recognizing patterns of characters (e.g. by using regular expressions) into tokens that carries a type and the
matched word (which is called _lexeme_).

A **parser** will take this stream of **tokens** and transform into a tree-form data structure according to the _grammar_ of the
language.

```
# Small piece of Sequence code
Sequence Test
    User ask Alpha "Hi"

# The fragment above is really just a sequence of characters:
characters = ['S', 'e', 'q', 'u', 'e', 'n', 'c', 'e', ' ', 'T', 'e', 's', 't', '\n', ' ', ' ', ' ', ' ', 'U', 's', 'e', 'r', ' ', 'a', 's', 'k', ' ', 'A', 'l', 'p', 'h', 'a', ' ', '"', 'H', 'i', '"']

# Which gets converted into tokens
tokens = [
    {
        "type": "sequence",
        "lexeme": "Sequence"
    },
    {
        "type": "identifier",
        "lexeme": "Test"
    },
    {
        "type": "identifier",
        "lexeme": "User"
    },
    {
        "type": "ask",
        "lexeme": "ask"
    },
    {
        "type": "identifier",
        "lexeme": "Alpha"
    },
    {
        "type": "string",
        "lexeme": "Hi!"
    }
]

# We convert that into a tree structure of nodes (called AST)
tree = {
    "type": "root",
    "children": [
        {
            "type": "sequence",
            "value": "Test",
            "children": [
                {
                    "type": "interaction"
                    "value": "",
                    "children": [
                        {
                            "type": "source"
                            "value": "User",
                            "children": []
                        },
                        {
                            "type": "ask"
                            "value": "",
                            "children": []
                        }
                        {
                            "type": "source"
                            "value": "User",
                            "children": []
                        },
                        {
                            "type": "destination"
                            "value": "Alpha",
                            "children": []
                        },
                        {
                            "type": "message"
                            "value": "Hi!",
                            "children": []
                        }
                    ]
                }
            ]
        }
    ]
}
```

There are multiple ways of implementing a parser in order to generate this tree, three common approaches are:

**Hand-written** - Unsurprisingly, is when you write all parts by yourself. This can be quite challenging
depending on how your language is constructed (left recursion, etc.), but also gives you unlimited possibilities
for error handling. I recommend Ruslan Spivak’s article series
[Let’s Build A Simple Interpreter](https://ruslanspivak.com/lsbasi-part1/) if you want to read up on how to build one yourself.

<br/>
**Parser combinator** - A parser combinator provides a library of small functions that you can compose to
write a complex parser in (often using functional composition). See [this article](https://medium.com/@chetcorcos/introduction-to-parsers-644d1b5d7f3d)
for a short introduction to parser combinators and examples using a JavaScript parser combinator.

<br/>
**Parser generator** - A parser generator, is a separate program (a compiler actually) that generates a parser
given a formal grammar definition. Often you describe your grammar in a format resembling
[Backus-Naur Form](https://en.wikipedia.org/wiki/Backus–Naur_form) and run a tool that generates the source
code for a parser that you can use in your program.

_I will not explain lexical theory in this post as there are tons of good articles available online which do
that better than I ever could. Instead this article will give you an overview and refer to a small implementation
with code available on [GitHub](https://github.com/eliasson/sequence)._


## Generating a parser

For Sequence I settled on using a parser generator as I did not want to hand write a parser. I chose
[ANTLR4](http://www.antlr.org), this is one of the most well-known parser generators out there. It has
an active community, good documentation (including a few books) and generates
[fairly performant parsers](https://sap.github.io/chevrotain/performance/).

ANTLR, is a Java program but is not limited to you using Java, at it can output generators in several different
programming languages – one of which is `JavaScript`. As Visual Studio Code extensions are implemented
in `TypeScript` or `JavaScript` this fits perfectly.

> Also, I do believe that you do not have to be a compiler nerd to implement your own language - or at
least I am not one. Based on you language and your needs a parser generator will be good enough and save
you a lot of trouble. Of course, people on the Internet tends to disagree on this matter.

In ANTLR you define your lexical rules, which patterns or stream of characters that should be recognized
and then you use these lexical tokens when defining the grammar (the rules of token composition). The
grammar rules (try reading the grammar rules as tree definitions).

The grammar definition in ANTLR is also a DSL and for the Sequence language, a stripped-down version of
the grammar definition looks like this:

```
grammar Sequence;

// Defines two custom tokens to deal with the whitespace significance, which is omitted
// from this example
tokens { INDENT, DEDENT }

// The root rule is where it all starts. This rule states what constructs that
// is allowed on the root level. The EOF indicates that the full input should
// be read and matched by the parser, else an error will be thrown.
root
    : NEWLINE* documentMeta definitions EOF
    ;

documentMeta
    : documentName
    ;

documentName
    : DOCUMENT_NAME string NEWLINE?
    ;

definitions
    : (NEWLINE | actorDefinition | objectDefinition | sequenceDefinition)*
    ;

actorDefinition
    : ACTOR IDENTIFIER IS string NEWLINE?
    | ACTOR IDENTIFIER NEWLINE?
    ;

objectDefinition
    : OBJECT IDENTIFIER IS string NEWLINE?
    | OBJECT IDENTIFIER NEWLINE?
    ;

sequenceDefinition
    : SEQUENCE IDENTIFIER sequenceBlock
    ;

sequenceBlock
    : NEWLINE INDENT sequenceMessage* DEDENT
    ;

sequenceMessage
    : IDENTIFIER (ASK|TELL|REPLIES) IDENTIFIER string NEWLINE?
    | IDENTIFIER (ASK|TELL|REPLIES) IDENTIFIER NEWLINE?
    ;

// String exposed as a grammar rule to make it testable directly
// without piggy-back on another rule.
string
    : STRING
    ;

// Definition of string tokens
DOCUMENT_NAME                                           : 'Name';
ACTOR                                                   : 'Actor';
OBJECT                                                  : 'Object';
IS                                                      : 'is';
SEQUENCE                                                : 'Sequence';
ASK                                                     : 'ask';
TELL                                                    : 'tell';
REPLIES                                                 : 'replies';

// Captures any word that is not a keyword. Keep this after the keywords
// are defined
IDENTIFIER
    : LETTER+ LETTERORDIGIT*
    ;

COMMENT
    : '#' .+? (NEWLINE | EOF) -> skip
    ;

STRING
    : (SINGLE_LINE_STRING | MULTI_LINE_STRING)
    ;

SKIPPING
    : SPACES -> skip
    ;

// A fragment itself will (cannot) be interpreted as a token, it is only meant
// to be reused from other lexer rules.

fragment SPACES
    : [ \t]+
    ;

fragment LETTER
    : [a-zA-Z]
    ;

fragment LETTERORDIGIT
    : [a-zA-Z0-9]
    ;

fragment DIGIT
    : [0-9]
    ;

fragment SINGLE_LINE_STRING
    : '"' (. | ~[\r\n"])*? '"'
    | '\'' (. | ~[\r\n'])*? '\''
    ;

fragment MULTI_LINE_STRING
    : '"""' (.)*? '"""'
    | '\'\'\'' (.)*? '\'\'\''
    ;
```

Lexical rules are in `UPPER-CASE` and grammar must start with a `lower-case` character. Other than that, I think
the ANTLR-grammar in this example is pretty easy to understand. (ANTLR has quite a few features, I am only using
the very basic).

The full grammar is available at [`parser/Sequence.g4`](https://github.com/eliasson/sequence/blob/master/parser/Sequence.g4)

### Feedback loop

Defining all the grammar at once is not what I recommend, I argue that you should TDD your language just like
any other implementation. The cycle I used for developing this language was:

1.	Implement the test for the new language construct
2.	Update the ANTLR grammar file with the new construct
3.	Generate a new parser (i.e. use the ANTLR compiler to generate source code)
4.	Run your tests

As stated in (3) a new parser will be generated several times, overwriting the existing parser. For this reason,
the generated parser is separated from the other source code, and no modifications should be made to the generated code.


### Using the ANTLR generated parser

Generate the Sequence parser by running the following commands:

```bash
# The following command use the alias defined when installing ANTLR
$ antlr4 -Dlanguage=JavaScript ./parser/Sequence.g4

# Which is the equvivalent of this command
$ java -Xmx500M -cp "/usr/local/lib/antlr-4.7-complete.jar:$CLASSPATH" org.antlr.v4.Tool -Dlanguage=JavaScript ./parser/Sequence.g4
```

ANTLR will output the source code for the parser next to where the `.g4` file is located. In the case
of Sequence you can find them in the directory [./parser/](https://github.com/eliasson/sequence/tree/master/parser).

We will import these modules in our code and use them to scan, lex, and parse the input to what is called a _Parser Tree_.

A **Parser Tree** is very similar to an *Abstract Syntax Tree* - but is closer to the actual tokens. E.g.
a parenthesis might be included in a parser tree to denote the start of a list, but is is not included
in the AST (which might only contain a `List` and `ListItems`).

```javascript
import * as antlr4 from 'antlr4';
import { SequenceLexer } from '../parser/SequenceLexer';
import { SequenceParser } from '../parser/SequenceParser';

const chars = new antlr4.InputStream(source);
const lexer = new SequenceLexer(chars);
const tokens  = new antlr4.CommonTokenStream(lexer);
const parser = new SequenceParser(tokens);
parser.buildParseTrees = true;

// Initiate the parsing, and begin with the root rule.
const parserResult = parser.root();
```

For Sequence there are two places where the generated parser is used, once for the entry point in [`src/index.js`](https://github.com/eliasson/sequence/blob/master/src/index.js)
of our _compiler_ (transpiler, generator or what you might call it). And another place is in the test utils at [`__tests__/test-utils.js`](https://github.com/eliasson/sequence/blob/master/__tests__/test-utils.js), so that we can test individual grammar rules.

One nice thing with the generated parser is that all the grammar rules are accessible. Notice the call made to
`parser.root()` above? This will parse the input according to the grammar defined by `root` in the `.g4` file. This
makes is super easy to write tests for only parts of your grammar.

E.g. if we were to write a test to make sure that an actor definition can be successfully parsed, we could have
the test only parse the input using the grammar defined in `parser.definitions()` (or if we wanted to be even
more specific `parser.actorDefinition()`.


### Unit-test

Sometimes I write unit-tests for the grammar only - this gives the benefit of having early tests that there is no error or
ambiguity in the grammar definition (the `.g4` files) but comes at a cost of some duplication of tests.

Other times I test both the grammar parsing and construction of AST in a single test. This only benefit is that you need to
write less tests.

To start with I recommend having separate unit-tests for parsing the grammar without exercising your own code.

Here is an example of a small from [`__tests__/grammar/definitions-grammar.spec.js`](https://github.com/eliasson/sequence/blob/master/__tests__/grammar/definitions-grammar.spec.js)
that asserts that an actor definition can be parsed (targetting a single grammar rule as
described earlier):

```javascript
const parse = (src) => createParser(src, false).definitions();

describe('parse single Actor definition', () => {
    let result;
    beforeEach(() => {
        result = parse(
            'Actor Jane\n' +
            '');
    });

    it('should parse without errors', () => {
        expect(result.parser._syntaxErrors).toEqual(0);
    });

    it('should have "Jane" as the symbol name', () => {
        expect(result.children[0].children[1].symbol.text).toEqual('Jane');
    });
});
```

Not that I even check that the Parser Tree contains the correct element. This is _not something I recommend_, instead leave
these tests to check of errors only. Later when building your **Abstract Syntax Tree** you will you will implicitly test
the parser tree as well. By testing explicitly, like above, you will be more fragile to grammatical changes in your test -
treat parser tree as an internal implementation detail.


### Errors and warnings

What you eventually will notice is that ANTLR prints errors to standard out, both for parser and lexer-errors. I recommend you to
substitute the built-in error handler with your own so that you can capture errors, this gives you the possibility to build better
tests as well as improving how errors are presented to the user.

```javascript
const chars = new antlr4.InputStream(source);
const lexer = new SequenceLexer(chars);
const tokens  = new antlr4.CommonTokenStream(lexer);
const parser = new SequenceParser(tokens);
parser.buildParseTrees = true;

// Remove the standard error listener since we do not want to
// print to stdout, but to capture the errors ourself to present
// a nice and tidy error message for the end-user.
parser.removeErrorListeners();
lexer.removeErrorListeners();

// Register our listener used to build the AST while the parsing
// is being done.
const errorListener = new AntlrErrorListener();  // <-- This is our own class
parser.addErrorListener(errorListener);

// Initiate the parsing, and begin with the root rule.
const parserResult = parser.root();
```


### Building your Abstract Syntax Tree

As said, the parser will generate a Parser Tree, which is closely related to the grammar definitions, while you can use this, it is a
bit raw, a higher abstraction is generally what you want – this is what is known as the Abstract Syntax Tree (or AST for short).

Our AST will be used to represent the participants and sequence of messages, we will use it later to produce the output
SVG and we will use it to perform semantic analysis (such as not being able to address a message to a non-existing participant).

Therefore, the AST should contain entities as actor, object, message, etc. as well as their associated identifier or value – but not
lexical details such as parentheses, line-breaks. I have often found that it is enough to keep the parser details to location (file,
line and column) and the symbol (lexeme) to be able to produce adequate diagnostic messages. My suggestion is to put as few things
in there as possible, as you always should do when building things.

For Sequence only a few types of nodes are needed, below is their definition in JavaScript.

```javascript

/**
 * The root node represents the sequence document, containing the meta-data
 * information, actors and the defined sequence flows.
 */
export class RootNode extends Node { ... }

/**
 * The declaration of the document name.
 *
 *     Name "My sequence flow"
 */
export class NameNode extends Node { ... }

/**
 * Represents an identifier of something (e.g. a sequence or a participant).
 *
 * E.g. the name of the actor is an identifier
 *
 *     Actor Alice
 */
export class IdentifierNode extends Node { ... }

/**
 * The definition of an actor, where the string at the end is the description.
 *
 *     Actor Alice is "A user"
 */
 export class ActorNode extends NodeWithIdentifier { ... }

/**
 * The definition of an object, where the string at the end is the description.
 *
 *     Object Bob is "A fake system"
 */
export class ObjectNode extends NodeWithIdentifier { ... }

/**
 * A Sequence of messages between one or more participants. This is what makes
 * a sequence diagram.
 *
 *      Sequence Hello
 *          Alice tell Bob "Sudo, make me a sandwich"
 *          Alice tell Charlie "Hello"
 */
export class SequenceNode extends NodeWithIdentifier { ... }

/**
 * The message that is exchanged by two participants, both of which are refered
 * to by their identifiers (names).
 *
 *      Alice ask Bob "Sudo, make me a sandwich"
 */
export class MessageNode extends Node { ... }

/**
 * Represents a string value, that can either be from a single line or a
 * multi-line string. This is what is between the quotes in a message.
 *
 *     Alice ask Bob "Sudo, make me a sandwich"
 *
 */
export class StringNode extends Node { ... } 
```

If you're curious, go and have a look at the file [`src/ast.js`](https://github.com/eliasson/sequence/blob/master/src/ast.js)
for a full view on what the AST nodes looks like.


### Visit your parser tree

So, how exactly is this parser tree of ours transformed into a AST?

When generating the parser ANTLR also generates a listener that defines `enter` and `exit` methods
for all grammar rules defined, if you open the file [`parser/SequenceListener.js`](https://github.com/eliasson/sequence/blob/master/parser/SequenceListener.js)
you will see a bunch of methods declared on the `SequenceListener` prototype looking like:

```javascript
// Enter a parse tree produced by SequenceParser#actorDefinition.
SequenceListener.prototype.enterActorDefinition = function(ctx) {
};

// Exit a parse tree produced by SequenceParser#actorDefinition.
SequenceListener.prototype.exitActorDefinition = function(ctx) {
};
```

Since we do not want to touch the generated code, we do not modify this file directly, instead
we declare our own `IntSequenceListener` which is derived from this one. The listener is then instantiated
and registered in the parser in the code that sets up the parser (which should be familiar by now).

```javascript
// Register our listener used to build the AST while the parsing is being done.
const errorListener = new AntlrErrorListener();
const listener = new IntSequenceListener();      // <-- This is our listener
parser.addErrorListener(errorListener);
parser.addParseListener(listener);                // <-- Get notified  during the parsing

// Initiate the parsing, and begin with the root rule.
const parserResult = parser.root();
```

ANTLR will call the `enter` method with the contents of the parser tree as argument when it _starts_
parse that grammar rule, and `exit` once that rule, and _all_ its inner rules have been parsed. I.e.
often there will calls to `enter` for other nodes before the current one is `exit`-ed.

Let's look at the order of calls for a tiny example:

```
Name "foo"
```

With only the the interesting part of the grammar definition cut out:

```
root
    : NEWLINE* documentMeta definitions EOF
    ;

documentMeta
    : documentName
    ;

documentName
    : DOCUMENT_NAME string NEWLINE?
    ;

string
    : STRING
    ;
```

1. The first grammar rule is the `root` grammar, thus a call to `listener.enterRoot(...)` is made
2. The `Name "foo"` will match the `documentMeta` grammar, a call to `listener.enterDocumentMeta(...)` is made
3. The documentMeta uses the `documentName` grammar, a call to `listener.enterDocumentName(...)` is made
4. The document meta uses the `string` grammar to match `"foo"`,a call to `listener.enterString(...)` is made
5. String only use lexical tokens, no additional call is made
6. A call to `listener.exitString(...)` is made
7. A call to `listener.exitDocumentName(...)` is made
8. A call to `listener.exitDocumentMeta(...)` is made
9. A call to `listener.exitRoot(...)` is made

Our listener `IntSequenceListener` is keeping an internal stack (or `Array` in JavaScript) where it pushes
all nodes as they are entered, when exited each node will attach itself to its parent - thus forming a tree.

It is as simple as that.

Inside the `exit` call the tokens in the parser tree is accessed to get the values needed (during exit, all
tokes are present, during enter it is not fully parsed yet). It is also here that we fetch the source location
and store in each node.

Sequence implements this listener in the same file as we define our AST nodes, in
[`src/ast.js`](https://github.com/eliasson/sequence/blob/master/src/ast.js)


### Input and output

The AST is also easy to test, you just parse the input and validate the presence of the expected nodes
in the output - there is no such thing as a side effect.

The difference is that the AST (at least for Sequence) cannot handle a delta or branch of Sequence code as
used for the grammar unit-test. Instead the main program entry is being tested, which use the `root` grammar,
so you have to specify "full" sequence descriptions in your test.

The AST tests is located at [`__tests__/ast/ast.spec.js`](https://github.com/eliasson/sequence/blob/master/__tests__/ast/ast.spec.js)


## Conclusions

A **parser generator** - is a good way of creating a custom parser, without the need to do the implement
a parser by hand (which of course might be needed sometimes, or can be a fun exercise).

**Separate the parser code from the AST**, you want the smartness of your program to work
on the AST as much as possible - it is a smaller data structure than the parser tree, and it reflects the
semantics of your domain better.

Parser and compilers are really well suited for **TDD**, it's all about input and output.

The next step is to use the generated AST and verify that it is semantically correnct, not only syntactically,
which is known as _static analysis_ or _semantic analysis_.

Happy hacking!
