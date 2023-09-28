# Stable Formatting

A TC39 proposal to bring stable Intl-inspired formatting options to ECMAScript.

**Stage:** 1

**Presentation**: [Slides](https://docs.google.com/presentation/d/1p1Xgywv1qfY54gnfHUM6PXQafqgf7wY98znxbolmP2c/edit?usp=sharing) for Stage 1 acceptance at September 2023 TC39 meeting

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

## Possible Solutions

It's entirely possible for a solution to this to be found in ECMA-262 outside `Intl`.
Two possible approaches are presented:
one that extends the `Intl` formatters
to support non-internationalization usage for the desired formatting,
and another that's a purely ECMA-262 solution.

### Add a new "null" locale

Define in ECMA-402 the behaviour of each of the formatters for the `zxx` null locale.
This locale identifier (which stands for for "no linguistic content; not applicable")
is a valid BCP 47 primary language tag defined in ISO 639.2
but its behaviour is not otherwise well defined.

For ease of use,
Intl formatters would accept `null` as an alias for the canonical `"zxx"` identifier.

Wherever possible, the `zxx` locale would use well-defined standardized behaviour,
such as using ISO-8601 for date formatting.

The "localized" output for `zxx` would avoid including actually localized text in its output,
such as fully written-out unit names or the names of months.

```js
Intl.Collator.supportedLocalesOf(null) → ['zxx']

new Intl.DateTimeFormat('zxx').format(new Date()) === '2023-09-01'

(12345.67).toLocaleString(null) === '12345.67'
```

### Add options to ECMA-262 formatters

Change

- `Date.prototype.toString()` [21.4.4.41](https://tc39.es/ecma262/#sec-date.prototype.tostring)
- `Number.prototype.toString([radix])` [21.1.3.6](https://tc39.es/ecma262/#sec-number.prototype.tostring)
- `BigInt.prototype.toString([radix])` [21.2.3.3](https://tc39.es/ecma262/#sec-bigint.prototype.tostring)

to include a new optional `options` argument:

- `Date.prototype.toString([options])`
- `Number.prototype.toString([radix] [, options])`
- `BigInt.prototype.toString([radix] [, options])`

and specify how these three functions should read the options
and create the formatted result string differently

The options read and respected by `Date.prototype.toString`
will be only a subset of what the `toLocaleString` method accepts.
For example, it will NOT read
"localeMatcher", "calendar", "numberingSystem", "hour12",
"dateStyle", and "timeStyle",
but will read "hourCycle", "timeZone".
The options listed in [Table 7](https://tc39.es/ecma402/#table-datetimeformat-components)
could be decided by the proposal to include for reading or not.

The options read and respected by `Number.prototype.toString` and `BigInt.prototype.toString`
will be only a subset of what the `toLocaleString` methods accept.
For example, they will NOT read
"localeMatcher", "numberingSystem", "style",
"currency", "currencyDisplay", "currencySign", "unit", "unitDisplay",
but will read other options listed in [Table 12](https://tc39.es/ecma402/#table-numberformat-resolvedoptions-properties).

The Number and BigInt methods could also allow for the options argument
to replace the `radix` argument, determining behaviour based on that argument's type.

This approach would not include any equivalent of the `Intl` formatters'
`formatToParts` methods.

## Alternatives

### Add an "undetermined" locale

A prior version of this proposal used the "undetermined" `und` locale instead of `zxx`.
This is also a valid ISO 639.2 language identifier,
but it is used as the canonical root locale identifier in CLDR,
which has well-defined behaviour e.g. in
[java.util.Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html).

The `und` locale is also currently supported by Safari as an alias for `en-US-u-va-posix`,
and it's recognised by Chrome and Node.js for `Intl.Locale`.

### Add new ECMA-262 formatting methods

Rather than modifying the existing `toString` methods of Date, Number and BigInt,
new methods `toFormattedString` could be added to each of these,
with an options argument as defined above.

## Prior Art

- [ISO 639-2](https://en.wikipedia.org/wiki/List_of_ISO_639-2_codes)
- [BCP 47 / RFC 5646](https://www.rfc-editor.org/rfc/rfc5646.html)
- [Unicode Technical Standard #35 (LDML)](https://unicode.org/reports/tr35/)
