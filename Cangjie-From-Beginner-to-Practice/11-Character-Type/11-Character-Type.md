# 11-Character Type

The character type is represented by `Rune /ruːn/` and can represent all characters in the Unicode character set.

Unicode, also called Universal Code, was developed by the Unicode Consortium and is an industry standard in the field of computer science, including character sets, encoding schemes, etc.

Unicode was created to address the limitations of traditional character encoding schemes. It establishes a unified and **unique binary code** for each character in each language to meet the requirements for text conversion and processing across languages and **platforms**.

## Character Type Literals

Character type literals have three forms: single character, escape character, and universal character. A `Rune` literal begins with the character `r`, followed by a character enclosed in a pair of single or double quotes.

Examples of single character literals:

```javascript
let a: Rune = r'a'
let b: Rune = r"b"
```

Escape characters are characters that interpret the following character differently in a character sequence. Escape characters use the escape symbol `\` followed by the character to be escaped. Examples are as follows:

```javascript
let slash: Rune = r'\\'
let newLine: Rune = r'\n'
let tab: Rune = r'\t'
```

Universal characters begin with `\u` followed by 1-8 hexadecimal digits defined within a pair of curly braces, representing the character corresponding to the Unicode value. Examples are as follows:

```javascript
main() {
    let he: Rune = r'\u{4f60}'
    let llo: Rune = r'\u{597d}'
    print(he)
    print(llo)
}
```

Compiling and executing the above code, the output is:

```text
你好
```

## [Operations Supported by Character Type](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/basic_data_type/characters.html#字符类型支持的操作)

The character type only supports relational operators: less than (`<`), greater than (`>`), less than or equal to (`<=`), greater than or equal to (`>=`), equal to (`==`), not equal to (`!=`). The comparison is based on the Unicode value of the characters.

`Rune` can be converted to `UInt32`, and integer types can be converted to `Rune`. For specific type conversion syntax and rules, please refer to [`Rune` to `UInt32` and Integer Type to `Rune` Conversion](https://docs.cangjie-lang.cn/docs/0.53.13/user_manual/source_zh_cn/class_and_interface/typecast.html#rune-到-uint32-和整数类型到-rune-的转换).



## Relationship between Unicode and ASCII

The relationship between Unicode and ASCII is mainly reflected in that Unicode is a superset of ASCII, containing all ASCII characters and being able to represent many more characters.

## ASCII

Size rules:****

Common ASCII code size rules: numbers < [uppercase letters](https://baike.baidu.com/item/大写字母/4666274?fromModule=lemma_inlink) < lowercase letters.

1. Numbers are smaller than letters. For example, "7"<"F";

2. The digit 0 is smaller than the digit 9, and increases sequentially from 0 to 9. For example, "3"<"8";

3. The letter A is smaller than the letter Z, and increases sequentially from A to Z. For example, "A"<"Z";

4. The uppercase letter is 32 less than the same lowercase letter. For example, "A"<"a".

ASCII codes for some common letters: "A" is 65; "a" is 97; "0" is 48

| Bin(binary) | Oct(octal) | Dec(decimal) | Hex(hexadecimal) | Abbreviation/Character       | Explanation    |
| ----------- | ----------- | ----------- | ------------- | --------------------------- | ------------ |
| 0000 0000   | 00          | 0           | 0x00          | NUL(null)                   | Null character       |
| 0000 0001   | 01          | 1           | 0x01          | SOH(start of headline)      | Start of headline     |
| 0000 0010   | 02          | 2           | 0x02          | STX (start of text)         | Start of text     |
| 0000 0011   | 03          | 3           | 0x03          | ETX (end of text)           | End of text     |
| 0000 0100   | 04          | 4           | 0x04          | EOT (end of transmission)   | End of transmission     |
| 0000 0101   | 05          | 5           | 0x05          | ENQ (enquiry)               | Enquiry         |
| 0000 0110   | 06          | 6           | 0x06          | ACK (acknowledge)           | Acknowledge     |
| 0000 0111   | 07          | 7           | 0x07          | BEL (bell)                  | Bell         |
| 0000 1000   | 010         | 8           | 0x08          | BS (backspace)              | Backspace         |
| 0000 1001   | 011         | 9           | 0x09          | HT (horizontal tab)         | Horizontal tab   |
| 0000 1010   | 012         | 10          | 0x0A          | LF (NL line feed, new line) | Line feed       |
| 0000 1011   | 013         | 11          | 0x0B          | VT (vertical tab)           | Vertical tab   |
| 0000 1100   | 014         | 12          | 0x0C          | FF (NP form feed, new page) | Form feed       |
| 0000 1101   | 015         | 13          | 0x0D          | CR (carriage return)        | Carriage return       |
| 0000 1110   | 016         | 14          | 0x0E          | SO (shift out)              | Shift out     |
| 0000 1111   | 017         | 15          | 0x0F          | SI (shift in)               | Shift in     |
| 0001 0000   | 020         | 16          | 0x10          | DLE (data link escape)      | Data link escape |
| 0001 0001   | 021         | 17          | 0x11          | DC1 (device control 1)      | Device control 1    |
| 0001 0010   | 022         | 18          | 0x12          | DC2 (device control 2)      | Device control 2    |
| 0001 0011   | 023         | 19          | 0x13          | DC3 (device control 3)      | Device control 3    |
| 0001 0100   | 024         | 20          | 0x14          | DC4 (device control 4)      | Device control 4    |
| 0001 0101   | 025         | 21          | 0x15          | NAK (negative acknowledge)  | Negative acknowledge     |
| 0001 0110   | 026         | 22          | 0x16          | SYN (synchronous idle)      | Synchronous idle     |
| 0001 0111   | 027         | 23          | 0x17          | ETB (end of trans. block)   | End of transmission block   |
| 0001 1000   | 030         | 24          | 0x18          | CAN (cancel)                | Cancel         |
| 0001 1001   | 031         | 25          | 0x19          | EM (end of medium)          | End of medium     |
| 0001 1010   | 032         | 26          | 0x1A          | SUB (substitute)            | Substitute         |
| 0001 1011   | 033         | 27          | 0x1B          | ESC (escape)                | Escape   |
| 0001 1100   | 034         | 28          | 0x1C          | FS (file separator)         | File separator   |
| 0001 1101   | 035         | 29          | 0x1D          | GS (group separator)        | Group separator       |
| 0001 1110   | 036         | 30          | 0x1E          | RS (record separator)       | Record separator   |
| 0001 1111   | 037         | 31          | 0x1F          | US (unit separator)         | Unit separator   |
| 0010 0000   | 040         | 32          | 0x20          | (space)                     | Space         |
| 0010 0001   | 041         | 33          | 0x21          | !                           | Exclamation mark         |
| 0010 0010   | 042         | 34          | 0x22          | "                           | Double quote       |
| 0010 0011   | 043         | 35          | 0x23          | #                           | Hash         |
| 0010 0100   | 044         | 36          | 0x24          | $                           | Dollar sign       |
| 0010 0101   | 045         | 37          | 0x25          | %                           | Percent sign       |
| 0010 0110   | 046         | 38          | 0x26          | &                           | Ampersand         |
| 0010 0111   | 047         | 39          | 0x27          | '                           | Single quote       |
| 0010 1000   | 050         | 40          | 0x28          | (                           | Opening parenthesis       |
| 0010 1001   | 051         | 41          | 0x29          | )                           | Closing parenthesis       |
| 0010 1010   | 052         | 42          | 0x2A          | *                           | Asterisk         |
| 0010 1011   | 053         | 43          | 0x2B          | +                           | Plus sign         |
| 0010 1100   | 054         | 44          | 0x2C          | ,                           | Comma         |
| 0010 1101   | 055         | 45          | 0x2D          | -                           | Minus sign/Hyphen  |
| 0010 1110   | 056         | 46          | 0x2E          | .                           | Period         |
| 0010 1111   | 057         | 47          | 0x2F          | /                           | Forward slash         |
| 0011 0000   | 060         | 48          | 0x30          | 0                           | Character 0        |
| 0011 0001   | 061         | 49          | 0x31          | 1                           | Character 1        |
| 0011 0010   | 062         | 50          | 0x32          | 2                           | Character 2        |
| 0011 0011   | 063         | 51          | 0x33          | 3                           | Character 3        |
| 0011 0100   | 064         | 52          | 0x34          | 4                           | Character 4        |
| 0011 0101   | 065         | 53          | 0x35          | 5                           | Character 5        |
| 0011 0110   | 066         | 54          | 0x36          | 6                           | Character 6        |
| 0011 0111   | 067         | 55          | 0x37          | 7                           | Character 7        |
| 0011 1000   | 070         | 56          | 0x38          | 8                           | Character 8        |
| 0011 1001   | 071         | 57          | 0x39          | 9                           | Character 9        |
| 0011 1010   | 072         | 58          | 0x3A          | :                           | Colon         |
| 0011 1011   | 073         | 59          | 0x3B          | ;                           | Semicolon         |
| 0011 1100   | 074         | 60          | 0x3C          | <                           | Less than         |
| 0011 1101   | 075         | 61          | 0x3D          | =                           | Equals sign         |
| 0011 1110   | 076         | 62          | 0x3E          | >                           | Greater than         |
| 0011 1111   | 077         | 63          | 0x3F          | ?                           | Question mark         |
| 0100 0000   | 0100        | 64          | 0x40          | @                           | At sign |
| 0100 0001   | 0101        | 65          | 0x41          | A                           | Uppercase A    |
| 0100 0010   | 0102        | 66          | 0x42          | B                           | Uppercase B    |
| 0100 0011   | 0103        | 67          | 0x43          | C                           | Uppercase C    |
| 0100 0100   | 0104        | 68          | 0x44          | D                           | Uppercase D    |
| 0100 0101   | 0105        | 69          | 0x45          | E                           | Uppercase E    |
| 0100 0110   | 0106        | 70          | 0x46          | F                           | Uppercase F    |
| 0100 0111   | 0107        | 71          | 0x47          | G                           | Uppercase G    |
| 0100 1000   | 0110        | 72          | 0x48          | H                           | Uppercase H    |
| 0100 1001   | 0111        | 73          | 0x49          | I                           | Uppercase I    |
| 01001010    | 0112        | 74          | 0x4A          | J                           | Uppercase J    |
| 0100 1011   | 0113        | 75          | 0x4B          | K                           | Uppercase K    |
| 0100 1100   | 0114        | 76          | 0x4C          | L                           | Uppercase L    |
| 0100 1101   | 0115        | 77          | 0x4D          | M                           | Uppercase M    |
| 0100 1110   | 0116        | 78          | 0x4E          | N                           | Uppercase N    |
| 0100 1111   | 0117        | 79          | 0x4F          | O                           | Uppercase O    |
| 0101 0000   | 0120        | 80          | 0x50          | P                           | Uppercase P    |
| 0101 0001   | 0121        | 81          | 0x51          | Q                           | Uppercase Q    |
| 0101 0010   | 0122        | 82          | 0x52          | R                           | Uppercase R    |
| 0101 0011   | 0123        | 83          | 0x53          | S                           | Uppercase S    |
| 0101 0100   | 0124        | 84          | 0x54          | T                           | Uppercase T    |
| 0101 0101   | 0125        | 85          | 0x55          | U                           | Uppercase U    |
| 0101 0110   | 0126        | 86          | 0x56          | V                           | Uppercase V    |
| 0101 0111   | 0127        | 87          | 0x57          | W                           | Uppercase W    |
| 0101 1000   | 0130        | 88          | 0x58          | X                           | Uppercase X    |
| 0101 1001   | 0131        | 89          | 0x59          | Y                           | Uppercase Y    |
| 0101 1010   | 0132        | 90          | 0x5A          | Z                           | Uppercase Z    |
| 0101 1011   | 0133        | 91          | 0x5B          | [                           | Opening square bracket     |
| 0101 1100   | 0134        | 92          | 0x5C          | \                           | Backslash       |
| 0101 1101   | 0135        | 93          | 0x5D          | ]                           | Closing square bracket     |
| 0101 1110   | 0136        | 94          | 0x5E          | ^                           | Caret       |
| 0101 1111   | 0137        | 95          | 0x5F          | _                           | Underscore       |
| 0110 0000   | 0140        | 96          | 0x60          | `                           | Backtick     |
| 0110 0001   | 0141        | 97          | 0x61          | a                           | Lowercase a    |
| 0110 0010   | 0142        | 98          | 0x62          | b                           | Lowercase b    |
| 0110 0011   | 0143        | 99          | 0x63          | c                           | Lowercase c    |
| 0110 0100   | 0144        | 100         | 0x64          | d                           | Lowercase d    |
| 0110 0101   | 0145        | 101         | 0x65          | e                           | Lowercase e    |
| 0110 0110   | 0146        | 102         | 0x66          | f                           | Lowercase f    |
| 0110 0111   | 0147        | 103         | 0x67          | g                           | Lowercase g    |
| 0110 1000   | 0150        | 104         | 0x68          | h                           | Lowercase h    |
| 0110 1001   | 0151        | 105         | 0x69          | i                           | Lowercase i    |
| 0110 1010   | 0152        | 106         | 0x6A          | j                           | Lowercase j    |
| 0110 1011   | 0153        | 107         | 0x6B          | k                           | Lowercase k    |
| 0110 1100   | 0154        | 108         | 0x6C          | l                           | Lowercase l    |
| 0110 1101   | 0155        | 109         | 0x6D          | m                           | Lowercase m    |
| 0110 1110   | 0156        | 110         | 0x6E          | n                           | Lowercase n    |
| 0110 1111   | 0157        | 111         | 0x6F          | o                           | Lowercase o    |
| 0111 0000   | 0160        | 112         | 0x70          | p                           | Lowercase p    |
| 0111 0001   | 0161        | 113         | 0x71          | q                           | Lowercase q    |
| 0111 0010   | 0162        | 114         | 0x72          | r                           | Lowercase r    |
| 0111 0011   | 0163        | 115         | 0x73          | s                           | Lowercase s    |
| 0111 0100   | 0164        | 116         | 0x74          | t                           | Lowercase t    |
| 0111 0101   | 0165        | 117         | 0x75          | u                           | Lowercase u    |
| 0111 0110   | 0166        | 118         | 0x76          | v                           | Lowercase v    |
| 0111 0111   | 0167        | 119         | 0x77          | w                           | Lowercase w    |
| 0111 1000   | 0170        | 120         | 0x78          | x                           | Lowercase x    |
| 0111 1001   | 0171        | 121         | 0x79          | y                           | Lowercase y    |
| 0111 1010   | 0172        | 122         | 0x7A          | z                           | Lowercase z    |
| 0111 1011   | 0173        | 123         | 0x7B          | {                           | Opening curly brace     |
| 0111 1100   | 0174        | 124         | 0x7C          | |                           | Vertical bar     |
| 0111 1101   | 0175        | 125         | 0x7D          | }                           | Closing curly brace     |
| 0111 1110   | 0176        | 126         | 0x7E          | ~                           | Tilde     |
| 0111 1111   | 0177        | 127         | 0x7F          | DEL (delete)                | Delete     |

10000–1FFFF: Plane 1, Supplementary Multilingual Plane (SMP) [1]

20000–2FFFF: Plane 2, **[Ideographic](https://baike.baidu.com/item/表意文字/3769313?fromModule=lemma_inlink)** **Supplementary Plane** (SIP) [1]

30000–3FFFF: Plane 3, **[Ideographic](https://baike.baidu.com/item/表意文字/3769313?fromModule=lemma_inlink)** **Tertiary Plane** (TIP)

40000–DFFFF: Planes 4-13, Not yet used

E0000–EFFFF: Plane 14, Supplementary Special-purpose Plane (SSP)

F0000–FFFFF: Plane 15, Reserved for [Private Use Area](https://baike.baidu.com/item/私人使用区/61727452?fromModule=lemma_inlink) (PUA, [PUA](https://baike.baidu.com/item/PUA/23356576?fromModule=lemma_inlink))

100000–10FFFF: Plane 16, Reserved for Private Use Area (PUA) [1]