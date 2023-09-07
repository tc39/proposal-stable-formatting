# An Explicit Root Locale

A TC-39 proposal to define the behaviour and properties of the `"und"` root locale.

**Stage:** 0

## Motivation

The `Intl` formatters and other interfaces defined by ECMA-402
are highly [implementation-dependent](https://tc39.es/ecma402/#sec-implementation-dependencies),
and liable to change over time.
This means that the output of these functions should not be relied upon,
as they're only intended to provide "best effort" results meant for immediate display to users.

However, this has not prevented users from relying on
string formatters having an exact output shape,
in particular with date formatting using the `en-US` locale.
Recent examples are available from
[June 2022](https://github.com/WebKit/WebKit/commit/1dc01f753d89a85ee19df8e8bd75f4aece80c594) and
[November 2022](https://bugs.chromium.org/p/v8/issues/detail?id=13494). Learn more in
[this presentation](https://docs.google.com/presentation/d/1KuIOSDQRliqCT3x3WX9Bg9H3hndfhAcH2G0aNoBKq18/edit#slide=id.p) from
[June 2023 TC39-TG2](https://github.com/tc39/ecma402/blob/master/meetings/notes-2023-06-01.md#how-to-prevent-misuse-of-localized-strings).

Separately, sometimes it is desirable to format values for an international audience,
or for other reasons use formats that are not tied to a specific locale.
The `Intl` formatters do not currently support this well.
For example, the top StackOverflow suggestion for how to format a date using ISO-8601 formatting
is to [use Swedish as the locale](https://stackoverflow.com/a/58633686).

## Possible Solution

We should define in ECMA-402 the behaviour of each of the formatters for the `"und"` root locale.
This locale identifier (short for "undetermined") is a valid BCP 47 primary language tag defined in ISO 639.2,
and it's recognised by numerous other existing systems.

Wherever possible, the `"und"` locale should use well-defined standardized behaviour,
such as using ISO-8601 for date formatting.

The "localized" output for `"und"` should avoid including actually localized text in its output,
such as fully written-out unit names or the names of months.

```js
Intl.Collator.supportedLocalesOf('und') → ['und']

new Intl.DateTimeFormat('und').format(new Date()) === '2023-09-01'

(12345.67).toLocaleString('und') === '12345.67'
```

## Prior Art

- [ISO 639-2](https://en.wikipedia.org/wiki/List_of_ISO_639-2_codes)
- [BCP 47 / RFC 5646](https://www.rfc-editor.org/rfc/rfc5646.html)
- [Unicode Technical Standard #35 (LDML)](https://unicode.org/reports/tr35/)
- [java.util.Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html)
- [PostgreSQL collation](https://www.postgresql.org/docs/current/collation.html)
