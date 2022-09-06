---
v: 3

docname: draft-ietf-sedate-datetime-extended-latest
cat: std
consensus: true
submissiontype: IETF
lang: en
pi:
  strict: 'yes'
  compact: 'yes'
  subcompact: 'no'
  tocdepth: '4'
  symrefs: 'yes'
  sortrefs: 'yes'
title: >
  Date and Time on the Internet: Timestamps with additional information
abbrev: Internet Extended Date/Time Fmt (IXDTF)
wg: Serialising Extended Data About Times and Events
updates: 3339

venue:
  group: Serialising Extended Data About Times and Events (SEDATE)
  mail: sedate@ietf.org
  github: ietf-wg-sedate/draft-ietf-sedate-datetime-extended

author:
- name: Ujjwal Sharma
  org: Igalia, S.L.
  street: Bugallal Marchesi, 22, 1º
  city: A Coruña
  country: Spain
  code: '15008'
  email: ryzokuken@igalia.com
-   name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org

contributor:
- name: Justin Grant
  email: justingrant.ietf.public@gmail.com
- name: Your Name Here

normative:
#  RFC2026:
#  RFC2028:
  RFC3339:
  RFC5234: abnf
#  RFC5646: # in BCP47
  RFC6838: media-types
  RFC8126:
#  BCP47:
  BCP178: RFC6648
  BCP175: RFC6557
informative:
  RFC2822:
  RFC5322:
  # obsolete, but needed for its Appendix E:
  RFC1305: ntp-old
  ISO8601:
    display: ISO8601:1988
    target: https://www.iso.org/standard/15903.html
    title: >
      Data elements and interchange formats — Information interchange —
      Representation of dates and times
    author:
    - org: International Organization for Standardization
      abbrev: ISO
    seriesinfo:
      ISO: '8601:1988'
    date: 1988-06
  ITU-R-TF.460-6:
    title: ITU-R TF.460-6. Standard-frequency and time-signal emissions
    author:
    date: 2002-02
    target: https://www.itu.int/rec/R-REC-TF.460/en
  IERS:
    target: https://www.iers.org/IERS/EN/Publications/Bulletins/bulletins.html
    title: International Earth Rotation Service Bulletins
    author:
    date: false
  JAVAZDT:
    target: https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_ZONED_DATE_TIME
    title: >
      Java SE 8, java.time.format, DateTimeFormatter: ISO_ZONED_DATE_TIME
    author:
    date: false
  CLDR:
    target: https://cldr.unicode.org
    title: Unicode CLDR Project
    date: false
    author:
  CLDR-CALENDAR:
    target: https://github.com/unicode-org/cldr/blob/main/common/bcp47/calendar.xml
    title: cldr/common/bcp47/calendar.xml
    date: false
  TZDB:
    target: https://data.iana.org/time-zones/tz-link.html
    title: Sources for time zone and daylight saving time data
    date: false
  TZDB-NAMING:
    target: https://data.iana.org/time-zones/theory.html
    title: Theory and pragmatics of the tz code and data
    date: false
  TR35:
    target: https://unicode-org.github.io/cldr/ldml/tr35-dates.html#Supplemental_Calendar_Data
    title: Unicode Technical Standard #35
    date: false
...

--- abstract


This document defines an extension to the timestamp format defined in
RFC3339 for representing additional information including a time
zone.

It updates RFC3339 in the specific interpretation of the local offset
`Z`, which is no longer understood to "imply that UTC is the preferred
reference point for the specified time"; see {{update}}.

[^status]

[^status]:
    The present version (-06) reflects the discussions at IETF 114.
    In particular, RFC 3339 is now updated with respect to the semantics
    of time zone offset `Z`.


--- middle

# Introduction {#intro}

Dates and times are used in a very diverse set of internet
applications, all the way from server-side logging to calendaring and
scheduling.

Each distinct instant in time can be represented in a descriptive text
format using a timestamp, and {{ISO8601}} standardizes a widely-adopted
timestamp format, which forms the basis of the Internet Date/Time Format {{RFC3339}}.
However, this format only allows timestamps to contain very little
additional relevant information, which means that, beyond that, any contextual
information related to a given timestamp needs to be either handled
separately or attached to it in a non-standard manner.

This is already a pressing issue for applications that handle each
instant with an associated time zone name, to take into account events
such as daylight saving time transitions.
Most of these applications attach the time zone to the timestamp in a
non-standard format, at least one of which is fairly well-adopted {{JAVAZDT}}.
Furthermore, applications might want to attach even more information to the
timestamp, including but not limited to the calendar system in which
it should be represented.

## Scope

This document defines an extension syntax for timestamps as specified
in {{RFC3339}} that has the following properties:

* The extension suffix is completely optional, making existing
  {{RFC3339}} timestamps compatible with this format.

* The format is compatible with the pre-existing popular syntax for attaching
  time zone names to timestamps {{JAVAZDT}}.

* The format provides a generalized way to attach any additional
  information to the timestamp.

We refer to this format as the Internet Extended Date/Time Format (IXDTF).

This document does not address extensions to the format where the
semantic result is no longer a fixed timestamp that is referenced to a
(past or future) UTC time.
For instance, it does not address:

* Future time given as a local time in some specified time zone, where
  changes to the definition of that time zone (e.g., a political
  decision to enact or rescind daylight saving time) affect the
  instant in time corresponding with the timestamp.
* "Floating time", i.e., a local time without information describing
  the UTC offset or time zone in which it should be interpreted.
* The use of timescales different from UTC, such as TAI.

However, additional information augmenting a fixed timestamp may be
sufficient to detect an inconsistency between intention and the actual
information in the timestamp, e.g., between the UTC offset and time zone
name in the timestamp.
For instance, such an inconsistency might arise because of:

* political decisions as discussed above, or
* errors in the applications producing and consuming such a timestamp.

While the information available is not generally sufficient to resolve
the inconsistency, it may be used to initiate some out of band
processing to obtain sufficient information for such a resolution.


In order to address some of the requirements implied here, future
related specifications might define syntax and semantics of strings
similar to {{RFC3339}}.
Note that the extension syntax defined in the present document is
designed in such a way that it can be useful for such specifications
as well.

## Definitions {#definitions}

{::boilerplate bcp14-tagged}


UTC:
: Coordinated Universal Time, as maintained since 1988 by the Bureau
  International des Poids et Mesures (BIPM) in conjunction with leap
  seconds as announced by the International Earth Rotation and
  Reference Frames Service {{IERS}}.
  From 1972 through 1987, UTC was maintained entirely by Bureau
  International de l'Heure (BIH).
  Before 1972, UTC was not generally recognized and civil time was
  determined by individual jurisdictions using different techniques
  for attempting to follow Universal Time based on measuring the
  rotation of the earth.

  UTC is often mistakenly referred to as GMT, an earlier timescale
  UTC was designed to be a useful successor for.

ABNF:
: Augmented Backus-Naur Form, a format used to represent permissible
  strings in a protocol or language, as defined in {{RFC5234}}.
  The rules defined in {{Appendix B of RFC5234}} are imported implicitly.

Internet Extended Date/Time Format (IXDTF):
: The date/time format defined in {{extended-format}} of this document.

Timestamp:
: An unambiguous representation of some instant in time.

UTC Offset:
: Difference between a given local time and UTC, usually given in
  negative or positive hours and minutes. For example, local time in New
  York in the wintertime is 5 hours behind UTC, so its UTC offset is "-05:00".

Z:
: A suffix which, when applied to a time, denotes a UTC offset of
  00:00; often spoken "Zulu" from the ICAO phonetic alphabet
  representation of the letter "Z". (Definition from {{Section 2 of RFC3339}}.)

Time Zone:
: A set of rules representing the relationship of local time to UTC
  for a particular place or region. Mathematically, a time zone can
  be thought of as a function that maps timestamps to UTC offsets.
  Time zones can deterministically convert a timestamp to local time.
  They can also be used in the reverse direction to convert local time
  to a timestamp, with the caveat that some local times may have zero
  or multiple possible timestamps due to nearby Daylight Saving Time
  changes or other changes to the UTC offset of that time zone.
  Unlike the UTC offset of a timestamp which makes no claims about
  the UTC offset of other related timestamps (and which is therefore
  unsuitable for performing local-time operations such as
  "one day later"), a time zone also defines how to derive new
  timestamps based on differences in local time.
  For example, to calculate "one day later than this
  timestamp in San Francisco", a time zone is required because the
  UTC offset of local time in San Francisco can change from one day
  to the next.

IANA Time Zone:
: A named time zone that is included in the Time Zone Database (often
  called `tz` or `zoneinfo`) maintained by IANA {{TZDB}}{{BCP175}}.
  Most IANA time zones
  are named for the largest city in a particular region that shares
  the same time zone rules, e.g. `Europe/Paris` or `Asia/Tokyo` {{TZDB-NAMING}}.
  Special IANA time zones such as `Etc/GMT+10` can be used to represent
  timestamps outside country boundaries, e.g. a buoy in the middle
  of the Pacific Ocean (note that the `Etc/GMT+10` time zone has a constant UTC
  Offset of -10:00 \[sic!]). <!-- The IANA time zone for `Z` is called
  `Etc/GMT`. Not true.  No idea which time zone name is preferred for Z. -->

  Note that the rules defined for a named IANA time zone can change
  over time.
  The use of a named IANA time zone implies that the intent is for the
  rules that are current at the time of interpretation to apply, i.e.,
  the additional information conveyed by using that time zone name is
  to change with the changed rules as recorded in the IANA time zone
  database.

Offset Time Zone:
: A time zone defined by a specific UTC offset, e.g. `+08:45` and
  serialized using as its name the same numeric UTC offset format used in an
  RFC 3339 timestamp.
  Although serialization with offset time zones is
  supported in this document for backwards compatibility with
  java.time.ZonedDateTime {{JAVAZDT}}, use of offset time zones is
  strongly discouraged.
  In particular, programs MUST NOT copy the UTC
  offset from a timestamp into an offset time zone in order to satisfy
  another program which requires a time zone annotation in its input.
  Doing this will improperly assert that the UTC offset of timestamps
  in that location will never change, which can result in incorrect
  calculations in programs that add, subtract, or otherwise derive new
  timestamps from the one provided. For example,
  `2020-01-01T00:00+01:00[Europe/Paris]` will let programs add six
  months to the timestamp while adjusting for Summer Time (Daylight Saving Time).
  But the same calculation applied to `2020-01-01T00:00+01:00[+01:00]`
  will produce an incorrect result that will be off by one hour in the
  timezone `Europe/Paris`.

CLDR:
: Common locale data repository {{CLDR}}, a project of the Unicode
  Consortium to provide locale data to applications.

For more information about timescales, see {{Appendix E of RFC1305}},
Section 3 of {{ISO8601}}, and the appropriate ITU documents
{{ITU-R-TF.460-6}}.

# Updating RFC 3339 {#update}

{{Section 4.3 of RFC3339}} states that an offset given as `Z` or
`+00:00` implies that "UTC is the preferred reference point for the
specified time".  The offset `-00:00` is provided as a way to express
that "the time in UTC is known, but the offset to local time is
unknown".

This convention mirrors a similar convention for date/time information
in email headers, described in {{Section 3.3 of RFC5322}} and introduced
earlier in {{Section 3.3 of RFC2822}}.
The latter convention is in actual use, while the former always was
handicapped by the fact that {{ISO8601}} does not actually allow `-00:00`.

Implementations that needed to express the semantics of `-00:00`
therefore tended to use `Z` as a "neutral" offset instead.

This specification updates RFC3339, aligning it with the actual
practice of interpreting the local offset `Z`: this is no longer
understood to "imply that UTC is the preferred reference point for the
specified time".

Note that the semantics of the local offset `+00:00` is not updated;
this retains the implication that UTC is the preferred reference point
for the specified time.

Note also that the fact that {{ISO8601}} does not allow `-00:00` as a
local offset reduces the level of interoperability that can be
achieved in using this feature; the present specification however does
not formally deprecate this syntax.  For the intents and purposes of
the present specification, the local offset `Z` can be used in its place.

# Internet Extended Date/Time format (IXDTF) {#date-time-format}

This section discusses desirable qualities of formats for the
timestamp extension suffix and defines such a format that extends
{{RFC3339}} for use in Internet protocols.

## Informative

The format is intended to allow implementations to specify additional
important information in addition to the bare timestamp.

This is done by defining *tags*, each with a *key* and
a *value* separated by an equals sign.
The value of a tag can be one or more items delimited by hyphen/minus signs.

Applications can build an informative timestamp *suffix* using any number of
these tags.

Keys are case-sensitive.  Values are case-sensitive unless otherwise specified.

When a suffix contains a repeated key or otherwise conflicting tags,
implementations MUST give precedence to whichever value is positioned
first. [^interop1]

[^interop1]:  I'd rather place a MU⁠ST NOT for this case, first.  This
     definitely needs to be expanded into some general text about
     error handling.
{: source="--- cabo"}

## Registered

Actual suffix tag keys are registered by supplying the information
specified in this section.  This information is modeled after that
specified for the media type registry {{RFC6838}}; if in doubt, the
provisions of this registry should be applied analogously.

Key Identifier:
: The key (conforming to `suffix-key` in {{abnf}})

Registration status:
: "Provisional" or "Permanent"

Description:
: A very brief description of the key.

Change controller:
: Who is in control of evolving the specification governing values for
  this key.  This information can include email addresses of contact
  points and discussion lists, and references to relevant web pages (URLs).

Reference:
: A reference.
  For permanent tag keys, this includes a full specification.
  For provisional tag keys, there is an expectation that some
  information is available even if that does not amount to a full
  specification; in this case, the registrant is expected to improve this
  information over time.

Key names that start with an underscore are intended for experiments
in controlled environments and cannot be registered; such keys MUST
NOT be used for interchange and MUST be rejected by implementations
not specifically configured to take part in such an experiment.
See {{BCP178}} for a discussion about the danger of experimental keys
leaking out to general production and why that MUST be prevented.

## Optionally Critical

For the format defined here, suffix tags are always *optional*: They
can be added or left out as desired by the generator of the string in
Internet Extended Date/Time Format (IXDTF).  (An application might require the presence
of specific suffix tags, though.)

Without further indication, they are also *elective*: Even if included
in the IXDTF string, the recipient is free to
ignore the suffix tag.  Reasons might include that the recipient does
not implement (or know about) the specific suffix key, or that it does
recognize the key but cannot act on the value provided.

A suffix tag may also indicate that it is *critical*: The recipient is
advised that it MUST NOT act on the Internet Extended Date/Time Format (IXDTF) string
unless it can process the suffix tag as specified.  A critical suffix
tag is indicated by following its opening bracket with an exclamation
mark (see `critical-flag` in {{abnf}}).

IXDTF strings such as:

    2022-07-08T00:14:07+01:00[Europe/Paris]

are internally inconsistent, as Europe/Paris does not use a time zone offset of `+01:00` in July 2022.
The time zone hint given in the suffix tag is elective, though, so the recipient is not
required to act on the inconsistency; it can treat the Internet
Date/Time Format string as if it were:

    2022-07-08T00:14:07+01:00

Similar with:

    2022-07-08T00:14:07+01:00[knort=blargel]

(assuming that the recipient does not understand the suffix key `knort`).

<aside markdown="1">

Note that:

    2022-07-08T00:14:07Z[Europe/Paris]

does not exhibit such an inconsistency, as the local offset of `Z`
does not imply a specific preferred time zone of interpretation.
With the knowledge of how time zone offsets applied to Europe/Paris in
the summer of 2022, it is equivalent to:

    2022-07-08T02:14:07+02:00[Europe/Paris]

</aside>

In contrast to this elective use of a suffix tag,

    2022-07-08T00:14:07+01:00[!Europe/Paris]
    2022-07-08T00:14:07Z[!knort=blargel]

have an internal inconsistency or an unrecognized suffix key/value
that are marked as critical, so a recipient MUST treat the IXDTF
string as erroneous.

Note that this does not mean that an application is disallowed to
perform additional processing on elective suffix tags, e.g., asking
the user how to resolve the inconsistency.
It means it is not required to do so with elective suffix tags, but is
required to reject or perform some other error handling when
encountering inconsistent or unrecognized suffix tags marked as
critical.

# Syntax Extensions to RFC 3339 {#extended-format}

## ABNF

The following rules extend the ABNF syntax defined in {{RFC3339}} in
order to allow the inclusion of an optional suffix.

The Internet Extended Date/Time Format (IXDTF) is described by the
rule `date-time-ext`.

`date-time` and `time-numoffset` are imported from {{Section 5.6 of
RFC3339}}, `ALPHA` and `DIGIT` from {{Section B.1 of RFC5234}}.

~~~~ abnf
time-zone-initial = ALPHA / "." / "_"
time-zone-char    = time-zone-initial / DIGIT / "-" / "+"
time-zone-part    = time-zone-initial *13(time-zone-char)
                    ; but not "." or ".."
time-zone-name    = time-zone-part *("/" time-zone-part)
time-zone         = "[" critical-flag
                        time-zone-name / time-numoffset "]"

key-initial       = ALPHA / "_"
key-char          = key-initial / DIGIT / "-"
suffix-key        = key-initial *key-char

suffix-value      = 1*alphanum
suffix-values     = suffix-value *("-" suffix-value)
suffix-tag        = "[" critical-flag
                        suffix-key "=" suffix-values "]"
suffix            = [time-zone] *suffix-tag

date-time-ext     = date-time suffix

critical-flag     = [ "!" ]

alphanum          = ALPHA / DIGIT
~~~~
{: #grammar title="ABNF grammar of extensions to RFC 3339"}

Note that a `time-zone` is syntactically similar to a `suffix-tag`,
but does not include an equals sign.
This special case is only available for time zone tags.

## Examples {#date-time-examples}

Here are some examples of Internet Extended Date/Time Format (IXDTF).

~~~~
1996-12-19T16:39:57-08:00
~~~~
{: #rfc3339-datetime title="RFC 3339 date-time with time zone offset"}

{{rfc3339-datetime}} represents 39 minutes and 57 seconds after the 16th hour of
December 19th, 1996 with an offset of -08:00 from UTC.
Note that this is the same instant in time as `1996-12-20T00:39:57Z`, expressed in UTC.

~~~~
1996-12-19T16:39:57-08:00[America/Los_Angeles]
~~~~
{: #datetime-tzname title="Adding a time zone name"}

{{datetime-tzname}} represents the exact same instant as the previous example but
additionally specifies the human time zone associated with it
("Pacific Time") for time-zone-aware implementations to take into
account.

~~~~
1996-12-19T16:39:57-08:00[America/Los_Angeles][u-ca=hebrew]
~~~~
{: #date-time-hebrew title="Projecting to the Hebrew calendar"}

{{date-time-hebrew}} represents the exact same instant, but it informs calendar-aware
implementations (see {{calendar}}) that they should project it to the Hebrew calendar.

~~~~
1996-12-19T16:39:57-08:00[_foo=bar][_baz=bat]
~~~~
{: #date-time-private title="Adding experimental tags"}

{{date-time-private}}, based on {{rfc3339-datetime}}, utilizes keys
identified as experimental by a leading underscore to declare two additional pieces of
information in the suffix; these can be interpreted by implementations
that take part in the controlled experiment making use of these tag keys.

# The u-ca Suffix Key: Calendar Awareness {#calendar}

Out of the possible suffix keys, the suffix key `u-ca` is allocated to
indicate the calendar in which the date/time is preferably presented.

A calendar is a set of rules defining how dates are counted and
consumed by implementations.  The set of suffix values allowed for
this suffix key is as defined by the {{CLDR}} data for {{TR35}}.
At the time of writing, this information is collected in {{CLDR-CALENDAR}}.


# IANA Considerations {#iana-cons}

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFCthis with the RFC
    number of this RFC and remove this note.

IANA is requested to establish a registry called "Timestamp Suffix Tag Keys".
Each entry in the registry shall consist of the information described in {{registered}}.
Initial contents of the registry are specified in {{tab-registered}}.

| Key Identifier | Registration status | Description:                        | Change controller | Reference             |
| u-ca           | Permanent           | Preferred Calendar for Presentation | IESG              | {{calendar}} of RFCthis |
{: #tab-registered title="Initial Content of Timestamp Suffix Tag Keys registry"}

The registration policy {{RFC8126}} is "Specification Required" for
permanent entries, and "Expert Review" for provisional ones.
In the second case, the expert is instructed to ascertain that a basic
specification does exist, even if not complete or published yet.

# Security Considerations

TBD

--- back

# Acknowledgements
{:unnumbered}

Richard Gibson provided some editorial improvements.
