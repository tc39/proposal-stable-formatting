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

## Proposed Solution

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

### Intl.Collator

When the `zxx` locale is used, [CLDR root collation](https://www.unicode.org/reports/tr35/tr35-collation.html#Root_Collation)
is used, with unified ideographs ordered either by block and then by code point, or by radical-stroke
(See [issue #13](https://github.com/tc39/proposal-stable-formatting/issues/13)).

### Intl.DateTimeFormat

When the `zxx` locale is used, the formatted output matches that used by Temporal.
To achieve that, the following default option values are applied:

```js
{
  calendar: 'gregory',
  numberingSystem: 'latn',
  hour12: false,
  hourCycle: 'h23'
}
```

In the formatted output, an RFC 9557 serialization of the input value is used when appropriate,
e.g. `2006-01-02`, `15:04:05`, `2006-01-02T15:04:05.999+01:00[Europe/Paris]`.
Only numerical representations of time and date values are used, as in:

```js
const dtf = new Intl.DateTimeFormat(null, { month: "long" });
dtf.format(new Date("2006-01-02")) === "1";
```

The `hour12` and `hourCycle` options are validated but ignored,
and formatting is always done as if `hour12: false` was set.

The `dayPeriod`, `weekday`, `era` options are validated but ignored.

The `month` option values `'long'` and `'short'` are considered equivalent to its `'2-digit'` value,
and `'narrow'` is considered equivalent to its `'numeric'` value.

If the `timeZoneName` option has a valid value,
the canonical IANA time zone identifier is used for the timezone, irrespective of the option value.

If the `dateStyle` option has a valid value,
the formatted output always starts with a date formatted like `2006-01-02`.

If the `timeStyle` option has a valid value, the formatted output always ends with a time formatted as follows:

- `'full'` or `'long'`: `15:04:05+01:00[Europe/Paris]`
- `'medium'`: `15:04:05`
- `'short'`: `15:04`

If both `dateStyle` and `timeStyle` options are set, the formatted output consists of
the formatted date, followed by `T` (U+0054), followed by the formatted time.

### Intl.DisplayNames

When the `zxx` locale is used with valid formatting options,
calling the `of(code)` method with structurally valid input
will behave as if no matching display name is available,
and return either the requested code or `undefined`,
depending on the `fallback` option.

### Intl.DurationFormat

When the `zxx` locale is used with valid formatting options,
the formatted duration is an ISO 8601-2 duration,
such as `P2Y` (2 years), `PT2H30M` (2 hours and 30 minutes), or `P5DT0.001S` (5 days and 1 millisecond).

### Intl.ListFormat

When the `zxx` locale is used, the `type` option value is validated but ignored,
and the output is determined by the `style` option:

- `'long'` or `'short'`: list items are separated by a comma followed by a space `, ` (U+002C U+0020)
- `'narrow'`: list items are separated by a space (U+0020)

### Intl.Locale

TBD

### Intl.NumberFormat

When the `zxx` locale is used, the numerical part of the formatted output
always satisfies the [_StrNumericLiteral_](https://tc39.es/ecma262/#prod-StrNumericLiteral) grammar symbol.
In the locale options, `'latn'` is used as default value for the `numberingSystem` option.

When used together with the `style: 'currency'` option,
the output includes the numerical value, followed by a space (U+0020), followed by the ISO currency code.
The `currencyDisplay` option value is validated but ignored.
If the [intl-currency-display-choices](https://github.com/tc39/proposal-intl-currency-display-choices) proposal
is accepted, using the `currencyDisplay: 'never'` option leaves out all but the numerical value from the output.

When used together with the `style: 'percent'` option,
the output includes the numerical value followed by the U+0025 Percent Sign character.

When used together with the `style: 'unit'` option,
the output is determined by the `unitDisplay` option:

- `'short'`: numerical value, followed by a space (U+0020), followed by the short unit identifier
- `'narrow'`: numerical value, followed by the short unit identifier
- `'long'`: numerical value, followed by a space (U+0020), followed by the long unit identifier

The "long unit identifier" is the `unit` option value.
The "short unit identifier" is a locale-independent string derived from the `unit` option value
which will need to be explicitly defined from SI units and otherwise,
with e.g. `l` for `litre` and `TB` for `terabyte`.
Compound units are formed by replacing the `-per-` with a solidus `/` (U+002F)
and by mapping the unit parts separately to their short unit identifiers.

When used together with the `notation: 'compact'` option,
the output includes the numerical value followed by an appropriate SI prefix:

- 10<sup>12</sup>: `T` (U+0054)
- 10<sup>9</sup>: `G` (U+0047)
- 10<sup>6</sup>: `M` (U+004D)
- 10<sup>3</sup>: `k` (U+006B)
- 10<sup>-3</sup>: `m` (U+006D)
- 10<sup>-6</sup>: `μ` (U+03BC)
- 10<sup>-9</sup>: `n` (U+006E)
- 10<sup>-12</sup>: `p` (U+0070)

The `compactDisplay` option is validated but ignored.

The `useGrouping` option is validated but ignored,
and grouping separators are never included in the output.

### Intl.PluralRules

When the `zxx` locale is used with valid formatting options,
calling the `select(number)` and `selectRange(startRange, endRange)`
methods with structurally valid inputs will always return `'other'`.

### Intl.RelativeTimeFormat

When the `zxx` locale is used with valid formatting options,
the formatted relative time is an ISO 8601-2 duration
with either a Plus Sign `+` (U+002B) or a Hyphen-Minus `-` (U+002D) as its first character,
such as `+P2Y` (in 2 years), `-P1D` (yesterday), or `+PT10S` (in 10 seconds).

Quarters are expressed in months.

### Intl.Segmenter

When the `zxx` locale is used, [UAX #29](https://unicode.org/reports/tr29/) segmentation
with extended grapheme clusters is used, without tailorings
(See [issue #13](https://github.com/tc39/proposal-stable-formatting/issues/13)).

### Array.prototype.toLocaleString

When the `zxx` locale is used, array items are concatenated with a comma `,` (U+002C)
as a separator.

### String.prototype.toLocaleLowerCase & String.prototype.toLocaleUpperCase

When the `zxx` locale is used, the string is converted to the appropriate case
using the Unicode Default Case Conversion algorithm.

## Alternatives

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
