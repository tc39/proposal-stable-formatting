# An Explicit Null Locale

A TC-39 proposal to define the behaviour and properties of the `zxx` null locale.

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
[November 2022](https://bugs.chromium.org/p/v8/issues/detail?id=13494).

Separately, sometimes it is desirable to format values for an international audience,
or for other reasons use formats that are not tied to a specific locale.
The `Intl` formatters do not currently support this well.
For example, the top StackOverflow suggestion for how to format a date using ISO-8601 formatting
is to [use Swedish as the locale](https://stackoverflow.com/a/58633686).

## Possible Solution

We should define in ECMA-402 the behaviour of each of the formatters for the `zxx` null locale.
This locale identifier (which stands for for "no linguistic content; not applicable")
is a valid BCP 47 primary language tag defined in ISO 639.2
but its behaviour is not otherwise well defined.

For ease of use,
Intl formatters should accept `null` as an alias for the canonical `"zxx"` identifier.

Wherever possible, the `zxx` locale should use well-defined standardized behaviour,
such as using ISO-8601 for date formatting.

The "localized" output for `zxx` should avoid including actually localized text in its output,
such as fully written-out unit names or the names of months.

```js
Intl.Collator.supportedLocalesOf(null) → ['zxx']

new Intl.DateTimeFormat('zxx').format(new Date()) === '2023-09-01'

(12345.67).toLocaleString(null) === '12345.67'
```

## Alternatives

A prior version of this proposal used the "undetermined" `und` locale instead of `zxx`.
This is also a valid ISO 639.2 language identifier,
but it is used as the canonical root locale identifier in CLDR,
which has well-defined behaviour e.g. in
[java.util.Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html).

The `und` locale is also currently supported by Safari as an alias for `en-US-u-va-posix`,
and it's recognised by Chrome and Node.js for `Intl.Locale`.

## Prior Art

- [ISO 639-2](https://en.wikipedia.org/wiki/List_of_ISO_639-2_codes)
- [BCP 47 / RFC 5646](https://www.rfc-editor.org/rfc/rfc5646.html)
- [Unicode Technical Standard #35 (LDML)](https://unicode.org/reports/tr35/)
