# PENIS

Practical, Easy, Nested and Isolated Serialization  
Created by Jimmy Cushnie

**Note:** PENIS will change as we put it through its paces in a production environment. The following document refers to version 0.1 of the spec.

---

PENIS is a data serialization language designed for configuration files in video games. It is similar to YAML, but a little more specialized.

PENIS is practical. Its rules make it inherently fast and simple to use in a project.  
PENIS is easy. Easy to write, easy to read, and easy to build a parser for.  
PENIS is nested. Through the magic of indents, almost any kind of data can be serialized.  
PENIS is isolated. A value pair does not have an inherent type of data assosiated with it; `123` could be an integer, a float, a byte, or a signed byte, depending on how the parser is instructed to interpret it.

Example
---

```yaml
# This is a PENIS file that could store the settings for a game.

############
# GRAPHICS #
############

Resolution:
    x: 1920
    y: 1080
Window Mode: Fullscreen # unlike most DSLs, PENIS supports enumerated types
Max FPS: 60
VSync: singleBuffered
Anti-Aliasing: 8x

#########
# SOUND #
#########

Global Volume       : 100           # whitespace is, for the most part, ignored,
Music Volume        : 100           # so you can format your data however you wish.
SFX Volume          : 100
Subtitles           : English

############
# CONTROLS #
############

Mouse Sensitivity:
    x: 20
    y: 20
    
Walk Forwards: W
Walk Backwards: S
Jump: Space

############
# GAMEPLAY #
############

Forbidden Numbers:
    - 4
    - 2700
    - -2
    
Forbidden Haiku: """
    I looked to the skies
    But I saw only darkness
    Later, the sun rose
    """
```

Spec
---

* a PENIS file is UTF-8 encoded Unicode.
* the only valid whitespace character in PENIS is space (U+0020)
* new lines in both Unix and Windows formats are accepted.

Non-data lines
---

If a line contains no characters, only spaces, or if the first non-space character is the comment indicator `#`, the line contains no data and should be ignored by the data parser. The line must be preserved when the file is modified, however. Take the following example:

```PENIS
# TODO: investigate the possibility of funnier numbers

Funny Number: 69

DontChangeThis: false
```
If a PENIS parser modifies the value of DontChangeThis, it must preserve the structure of the document, with its one comment line and two whitespace lines.

Data lines
---

Any line which does NOT match the qualifications above is assumed to contain data. All data lines have a part called the "value". There are three types of data line: key, list, and multi-line string.

### Key lines

Key lines represent unordered data indexed by name. A key line contains a key part and a value part. These are separated by the character `:`.

All spaces surrounding the `:` character are ignored. All spaces at the beginning and end of the line are ignored. The following examples all result in a key of `key` and a value of `value`:

* `key:value`
* `key: value`
* `key : value`
* <pre>key      :       value           </pre>

A key may contain any unicode character except for newline characters, `:` and `#`. A key's first character may not be `-`. A key may not start or end with a space. A key must contain at least one character.

### List lines

List lines represent ordered data. A list line's first non-space character is `-`. Excluding any spaces between `-` and the next non-space character and any trailing spaces, the rest of the line is the value. Each of the following examples is a list node with a value of `heck`:

* `- heck`
* `-heck`
* <pre>-    heck        </pre>

### Multi-line strings

Strings can be represented over multiple lines, like so:

```PENIS
HorribleLimerick: """
    I once knew a man from Nantucket
    Whose dick was so long he could suck it
    He said with a grin
    And with cum on his chin
    'If my ear were a pussy, I'd fuck it!'
    """
```

The lines between the two `"""`s *do* contain data and should be interpreted by the parser. More details on their implementation in the String section.

---

If a data line contains the comment indicator `#`, that character and everything after it is ignored. For example, in the key line `opinion: I like #PENIS`, the value is `I like`, not `I like #PENIS`.

A line which does not meet the criteria to be a non-data line or any type of data line is invalid PENIS, and the parser should throw an error.

Nesting
---

Nesting is a core part of PENIS, and the reason it can represent such a wide variety of data. Nesting means that a data line can have child lines, and children can have children as well. A data line can only have children if it does *not* have a value; its children are interpreted as its value.

The *indentation* of a data line is the number of spaces at its beginning. A data line's children all have the same indentation, which is greater than the indentation of their parent.

Here is a simple example of indentation: a parent data line with three children.

```PENIS
Parent:
    Child1: Suzan
    Child2: David
    Child3: Francis
```

Here is a more complex example, representing a list of lists of lists of strings:

```PENIS
list of lists of lists of strings:
    -
        -
            - aaaaa
            - bbbbb
            - abababab
        -
            - string!
            - striiiiiiiiiiing.
        -
            - There are, right now, about 30 million slaves in the world.
            - Every day, hundreds of species go extinct.
            - Every year, a million people take their own life.
    -
        -
            - boobies
            - peepee
        -
            - masturbation is a sin.
            - """
                this string
                has several lines.
                That's cool. We don't judge.
                :)
                """
```

Here's another example showcasing how PENIS can serialize complex data in an easily human-readable format:

```PENIS
Torso:
    Chest:
        Left Nipple:
            Temperature: 42
            Color:
                r: 255
                g: 0
                b: 255
            Notes: 
                - "Very round."
                - "Appears to be looking at you."
                - "Soft."
        Right Nipple:
            Temperature: 16
            
    Abdomen:
        Notes: 
            - "Firm, but somewhat bloated."
```

A data line may only have one type of child: key, line, or multi-line string. The following, for example, is invalid PENIS and should cause the parser to throw an error:

```PENIS
Boi:
    key: 69
    - 69 #BAD
```

Structure
---

A PENIS file contains only the following:

* non-data lines
* top-level key lines - key lines with an indentation of 0
* the children/grandchildren/ect of the top-level key lines

A PENIS parser's job is to read, modify, add and remove the top-level key lines of a given file, without disturbing the non-data content.

String
---

A string is a piece of text. Strings in PENIS are interpreted fairly literally, where the interpreted string is the same as the value of the data line. The exception is that if a value's first and last character are both `"`, the first and last character are trimmed from the string. This allows you to save strings that have leading or trailing spaces, since those would otherwise be trimmed when interpreting what the value of the data line is.

If a string contains newlines, it is serialized in a special way: the data line's value is set to `"""`, each line of the string is made a child data line, and one final data line is added with the value `"""`. For example, here is how the first verse of Poe's *The Raven* is serialized in PENIS:

```PENIS
Verse1: """
    Once upon a midnight dreary, while I pondered weak and weary,
    Over many a quaint and curious volume of forgotten lore,            # snap snap snap
    While I nodded, nearly napping, suddenly there came a tapping,
    As of some one gently rapping, rapping at my chamber door.
    "Tis some visitor," I muttered, "tapping at my chamber door -
    Only this, and nothing more."
    """
```

If one line of a multi-line string contains leading or trailing spaces, that line can be individually encased in `"`s just like a single-line string, and the `"`s will be trimmed when interpreting the string.

Strings in PENIS may NOT contain the comment indicator character `#`. If a user tries to save this value, the parser should replace it with the `🍆` character and log an apologetic warning.

Integer
---

Integers are whole numbers. In PENIS, they are represented by the digits `0`-`9`, optionally preceded by the `-` character to indicate a negative number. For example, here is a serialized list of integers:

```PENIS
IntList:
    - 1
    - 0
    - -22
    - 69
    - -69
    - 696969
```

Float
---

Floats are numbers which can have a decimal place in them. In PENIS, they consist of two parts: an integer part, which follows the rules described above, and an optional subsequent fractional part, which consists of a decimal place `.` character followed by more digits. For example, here is a serialized list of floats:

```PENIS
FloatList:
    - 1.0
    - -2.7
    - 3333333333333333.33333
```

Floats also have a few special values:

* `infinity` is infinity.
* `-infinity` is negative infinity.
* `nan` is the Not a Number value.

None of those values are case sensitive.

Note that floats in PENIS do NOT allow scientific notation (i.e. you must use `0.0000001` instead of `1E-07`).

Boolean
---

A boolean is a value that can be true or false. In PENIS, this is represented by either `true` or `false`, without being case sensitive. For example, each element in the following serialized list of booleans should parse as true:

```PENIS
BoolList:
    - true
    - True
    - TRUE
    - tRUe  # valid, but strongly discouraged
```

Date and Time
---

You can represent a date and time in PENIS with the format `yyyy-MM-dd HH:mm:ss`. PENIS uses 24 hour time. PENIS times have only seconds precision. It is up to the user of the parser whether to interpret a Date and Time as global or in a local timezone.

Here is a serialized list of date and times:

```PENIS
DateTimeList:
    - 2018-09-22 11:33:00
    - 2000-07-21 00:00:00
    - 1990-01-09 22:03:22
```

Byte
---

A byte is an 8-bit binary value. In PENIS, they are represented the same way as an integer, but with a value range of 0 to 255. Anything outside of that range should have the parser throw an error.

Signed Byte
---

In a signed byte, the first bit is a sign to determine if the remaining 7 bits represent a postive or negative value. Signed bytes in PENIS are represented the same way as an integer, but with a range of -128 to 127. Anything outside of that range should have the parser throw an error.

Enumerated Types
---

If you would like to serialize an enumerated type, PENIS uses the name of that enumeration, without any information about any namespaces or classes that type might belong to.

File Extension
---

PENIS files should use the extension `.PENIS`.

Pronounciation
---

PENIS is not written in all caps only because it is an acronym. When speaking the name aloud, it should be at a noticeably higher volume than the rest of the conversation.

Wiki
---

We have an [Official PENIS Wiki](https://github.com/JimmyCushnie/PENIS/wiki) which catalogs PENIS parsers and projects using PENIS. If you've made one of those things, please add it there!
