# Base67

Base67 is a binary-to-text encoding designed to travel unescaped through URLs and file names. It is really **Base64 plus 3 signaling symbols**: 64 symbols carry data (6 bits each), and 3 extra symbols sit outside the data to split or mark parts of a message.

## Alphabet

```
0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz_-.~!
```

The same symbols as ASCII bytes in hexadecimal, in index order:

```
30 31 32 33 34 35 36 37 38 39
41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50 51 52 53 54 55 56 57 58 59 5A
61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74 75 76 77 78 79 7A
5F 2D 2E 7E 21
```

| Index | Symbols | Role |
|------:|---------|------|
| 0–9   | `0`–`9` | data |
| 10–35 | `A`–`Z` | data |
| 36–61 | `a`–`z` | data |
| 62    | `_`     | data |
| 63    | `-`     | data |
| 64    | `.`     | signal |
| 65    | `~`     | signal |
| 66    | `!`     | signal |

## Design

### Data: 64 symbols, 6 bits each

Indexes 0–63 work just like Base64: each symbol represents 6 bits. No big-number arithmetic or special block sizes are needed.

### Signals: 3 symbols outside the data

`.` `~` `!` never represent data. Because they cannot appear inside an encoded block, they can split or mark a message unambiguously, with no escaping and no length prefixes. This is the same principle JWT uses when it joins base64url blocks with `.`.

### Starts at `0`, not `A`

Standard Base64 orders its alphabet as `A-Z a-z 0-9`. Base67 starts with the digits, so that:

- `0`–`9` and `A`–`F` are worth 0–15, the same as in hexadecimal.
- It follows the convention of hex and Base36: digits first, letters after.
- The first 62 symbols are in ASCII order.

For the same reason, Base67 is **not compatible** with Base64 or base64url: the same characters have different values.

## Why these symbols

Besides letters and digits, only 17 ASCII characters can appear literally in a URL under RFC 3986. Of those, these five have the fewest trade-offs across URLs, file names and the command line:

| Symbol | In URLs | In file names |
|--------|---------|---------------|
| `_` | never escaped | no restrictions |
| `-` | never escaped | at the start, the command line reads it as an option |
| `.` | never escaped | Windows strips a trailing dot; a leading dot hides the file on Linux/macOS |
| `~` | escaped by forms (`%7E`) | at the start, the shell expands it to the home directory |
| `!` | escaped by forms (`%21`) | bash uses it for history expansion |

The two symbols with no trade-offs (`_` and `-`) were assigned to data. The three with some trade-off are the signals, which only appear where the format decides.

### Rejected symbols

| Symbol | Reason |
|--------|--------|
| `*` `:` `\|` `<` `>` `\` `/` `?` `"` | forbidden in Windows file names |
| `#` | starts the URL fragment; whatever follows never reaches the server |
| `%` | is the URL escape character |
| `+` | read as a space in the query string |
| `&` `=` | separate parameters and key/value pairs in the query string |
| `$` `,` `;` `@` | escaped by URL encoders |
| `(` `)` `'` | need quoting in the shell; `)` breaks Markdown links |
| `[` `]` `{` `}` `^` `` ` `` | not allowed in URLs |

## Compatibility

- **URLs:** no symbol is escaped in the path or the query string, nor by `encodeURIComponent`. In forms (`application/x-www-form-urlencoded`, `URLSearchParams`) only `~` and `!` are escaped.
- **File names:** every symbol is valid on Windows, Linux and macOS.

## Known limitations

- **Case sensitivity:** Windows and macOS (by default) do not distinguish case in file names. `aB` and `Ab` are different Base67 values, but they would be the same file.
- **Symbol position:** a value should not start with `-`, `.` or `~`, nor end with `.` or `!`. Using the signals as separators (between blocks) rather than terminators avoids almost every case.
- **Windows reserved names:** a value could match `CON`, `NUL`, `AUX`, `PRN`, `COM1`–`COM9` or `LPT1`–`LPT9`. A fixed prefix or extension avoids this.
- **Ordering:** the last five symbols are not in ASCII order, so sorting encoded text is not the same as sorting the original data.

## To be defined

- The meaning of each of the three signals (`.` `~` `!`).
- Padding when the byte count is not a multiple of 3 (omit it, like base64url, or use a signal).
- Bit order within each 6-bit group.
