# CA1 Reflection — Kobo Scanner

## 1. Number scanning

In `src/scanner.rs`, the `number` function checks whether the current
character is a decimal point and whether the character after it is a
digit. It only consumes the decimal point when both conditions are true.
This means `5.25` is scanned as one number, while `5.` is scanned as the
number `5` followed by a separate period. Similarly, `.5` does not
become a number because the scanner starts number scanning only when it
encounters a digit. I used `peek()` and `peek_next()` to make this
decision without consuming characters too early. This follows section
1.4 of the language specification, which says that a fractional part
needs digits after the decimal point.

**Code reference:** `src/scanner.rs`, the `number` function (the
`if self.peek() == '.' && self.peek_next().is_ascii_digit()` condition).

## 2. Line tracking and EOF

I increase the line counter when the scanner reads a newline in
`scan_token`. I also increase it inside `string` when a newline occurs
within a string, because strings are allowed to span multiple lines. At
the end of scanning, `run` sets the EOF token's line to the line of the
last token scanned, or to line 1 if there are no tokens. Therefore, if a
file ends with two blank lines after its last token, EOF carries the
last token's line number, not the physical final line of the file. This
is the EOF rule described in section 6.1 of the specification.

**Code references:** `src/scanner.rs`, the newline arm in `scan_token`;
the newline check in `string`; and the EOF line assignment in `run`.

## 3. A debugging experience

I encountered a failure with `invalid/unterminated_string.kobo`. The
expected error was `[line 1] Error: String is never closed.` The issue
taught me that an unterminated string must report the line where the
opening quote appeared, even if scanning inside the string changes the
current line. In my scanner, I save the starting line in `start_line`
and use it when reporting this error.

My earlier commit was `6f2c6d1`, where the relevant code was the
original unterminated-string error handling that reported the current
scanner line instead of the line where the string started. My fixing
commit was `3a99bd2`, where I changed it to save the starting line of
the string in `start_line` and use that line when reporting the error.

The misunderstanding I corrected was that I initially thought the
scanner should report the line it was currently on when it discovered
that a string was not closed. However, the specification requires the
error to be reported using the line where the opening quote appeared.
This became important for strings that contain newlines. After making
this change, the `invalid/unterminated_string.kobo` test passed, giving
me 14/14 passing tests.