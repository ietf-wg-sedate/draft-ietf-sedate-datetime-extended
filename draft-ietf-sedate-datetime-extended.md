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
- name: Carsten Bormann
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

normative:
#  RFC2026:
#  RFC2028:
  RFC3339:
  RFC5234: abnf
#  RFC5646: # in BCP47
  RFC6838: media-types
  BCP26: RFC8126
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
    ann: Also available from <⁠<https://nvlpubs.nist.gov/nistpubs/Legacy/FIPS/fipspub4-1-1991.pdf>>.
  ISO8601-2000:
    display: 'ISO8601:2000'
    target: https://www.iso.org/standard/26780.html
    title: >
      Data elements and interchange formats — Information interchange —
      Representation of dates and times
    author:
    - org: International Organization for Standardization
      abbrev: ISO
    seriesinfo:
      ISO: '8601:2000'
    date: 2000-12
  ISO8601-2019:
    display: 'ISO8601-1:2019'
    target: https://www.iso.org/standard/70907.html
    title: >
      Date and time — Representations for information interchange —
      Part 1: Basic rules
    author:
    - org: International Organization for Standardization
      abbrev: ISO
    seriesinfo:
      ISO: '8601-1:2019'
    date: 2019-02
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
  CLDR-LINKS:
    target: https://cldr.unicode.org/stable-links-info
    title: Stable Links Info
    author:
      org: Unicode CLDR
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
  # target: https://unicode-org.github.io/cldr/ldml/tr35-dates.html#Supplemental_Calendar_Data
    # target: https://www.unicode.org/reports/tr35/
    target: https://www.unicode.org/reports/tr35/#UnicodeCalendarIdentifier
    title: >
      Unicode Technical Standard #35:
      Unicode Locale Data Markup Language (LDML)
    date: false
  ICAO-PA:
    title: >
      Annex 10 to the Convention on International Civil Aviation:
      Aeronautical Telecommunications;
      Volume II Communication Procedures including those with PANS status (7th ed.)
    author:
      org: International Civil Aviation Organization
    date: July 2016
    target: https://store.icao.int/annex-10-aeronautical-telecommunications-volume-ii-communication-procedures-including-those-with-pans-status

# points to https://github.com/unicode-org/cldr/blob/main/common/bcp47/calendar.xml
  DATA-MINIMIZATION: I-D.arkko-iab-data-minimization-principle
...

--- abstract


This document defines an extension to the timestamp format defined in
RFC3339 for representing additional information including a time
zone.

It updates RFC3339 in the specific interpretation of the local offset
`Z`, which is no longer understood to "imply that UTC is the preferred
reference point for the specified time"; see {{update}}.

[^status]

[^status]:  (This "cref" paragraph will be removed by the RFC editor:)\\
    The present version (-09) addresses comments received during IETF
    last call.


--- middle

# Introduction {#intro}

Dates and times are used in a very diverse set of internet
applications, all the way from server-side logging to calendaring and
scheduling.

Each distinct instant in time can be represented in a descriptive text
format using a timestamp.
{{ISO8601-2019}} standardizes a widely-adopted
timestamp format, an earlier version of which {{ISO8601}} formed the
basis of the Internet Date/Time Format {{RFC3339}}.
However, this format allows timestamps to contain only very little
additional relevant information.
Beyond that, any contextual
information related to a given timestamp needs to be either handled
separately or attached to it in a non-standard manner.

This is a pressing issue for applications that handle each
such instant in time with an associated time zone name, in order to take into account events
such as daylight saving time transitions.
Many of these applications attach the time zone to the timestamp in a
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

* The format provides a generalized way to attach additional
  information to the timestamp.

We refer to this format as the Internet Extended Date/Time Format (IXDTF).

This document does not address extensions to the format where the
semantic result is no longer a fixed timestamp that is referenced to a
(past or future) UTC time.
For instance, it does not address:

* Future time given as a local time in some specified time zone, where
  changes to the definition of that time zone (such as a political
  decision to enact or rescind daylight saving time) affect the
  instant in time represented by the timestamp.
* "Floating time", i.e., a local time without information describing
  the UTC offset or time zone in which it should be interpreted.
* The use of timescales different from UTC, such as International Atomic
  Time (TAI).

However, additional information augmenting a fixed timestamp may be
sufficient to detect an inconsistency between intention and the actual
information in the timestamp, such as between the UTC offset and time zone
name.
For instance, inconsistencies might arise because of:

* political decisions as discussed above, or
* updates to time zone definitions being applied at different times
  by timestamp producers and receivers, or
* errors in programs producing and consuming timestamps.

While the information available in an IXDTF string is not generally sufficient to resolve
an inconsistency, it may be used to initiate some out of band
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
  for which UTC was designed to be a useful successor.

ABNF:
: Augmented Backus-Naur Form, a format used to represent permissible
  strings in a protocol or language, as defined in {{RFC5234}}.
  The rules defined in {{Appendix B of RFC5234}} are imported implicitly.

Internet Extended Date/Time Format (IXDTF):
: The date/time format defined in {{extended-format}} of this document.

Timestamp:
: An unambiguous representation of a particular instant in time.

UTC Offset:
: Difference between a given local time and UTC, usually given in
  negative or positive hours and minutes. For example, local time in
  the city of New York, NY, USA, in the wintertime in 2023, is 5 hours
  behind UTC, so its UTC offset is "-05:00".

Z:
: A suffix which, when applied to a time, denotes a UTC offset of
  00:00; often spoken "Zulu" from the ICAO phonetic alphabet
  representation of the letter "Z".
  (Definition from {{Section 2 of RFC3339}}; see {{ICAO-PA}} for the
  phonetic alphabet.)

Time Zone:
: A set of rules representing the relationship of local time to UTC
  for a particular place or region. Mathematically, a time zone can
  be thought of as a function that maps timestamps to UTC offsets.
  Time zones can deterministically convert a timestamp to local time.
  They can also be used in the reverse direction to convert local time
  to a timestamp, with the caveat that some local times may have zero
  or multiple possible timestamps due to nearby daylight saving time
  changes or other changes to the UTC offset of that time zone.
  Unlike the UTC offset of a timestamp which makes no claims about
  the UTC offset of other related timestamps (and which is therefore
  unsuitable for performing local-time operations such as
  "one day later"), a time zone also defines how to derive new
  timestamps based on differences in local time.
  For example, to calculate "one day later than this
  timestamp in San Francisco, California", a time zone is required because the
  UTC offset of local time in San Francisco can change from one day
  to the next.

IANA Time Zone:
: A named time zone that is included in the Time Zone Database (often
  called `tz` or `zoneinfo`) maintained by IANA {{TZDB}}{{BCP175}}.
  Most IANA time zones
  are named for the largest city in a particular region that shares
  the same time zone rules, e.g., `Europe/Paris` or `Asia/Tokyo` {{TZDB-NAMING}}.

  The rules defined for a named IANA time zone can change
  over time.
  The use of a named IANA time zone implies that the intent is for the
  rules to apply that are current at the time of interpretation:
  the additional information conveyed by using that time zone name is
  to change with any rule changes as recorded in the IANA time zone
  database.

Offset Time Zone:
: A time zone defined by a specific UTC offset, e.g. `+08:45`, and
  serialized using as its name the same numeric UTC offset format used in an
  RFC 3339 timestamp, for example:

      2022-07-08T00:14:07+08:45[+08:45]

  An offset in the suffix that does not repeat the offset of the
  timestamp is inconsistent (see {{inconsistent}}).

  Although serialization with offset time zones is
  supported in this document for backwards compatibility with
  `java.time.ZonedDateTime` {{JAVAZDT}}, use of offset time zones is
  strongly discouraged.
  In particular, programs MUST NOT copy the UTC
  offset from a timestamp into an offset time zone in order to satisfy
  another program which requires a time zone suffix in its input.
  Doing this will improperly assert that the UTC offset of timestamps
  in that location will never change, which can result in incorrect
  calculations in programs that add, subtract, or otherwise derive new
  timestamps from the one provided. For example,
  `2020-01-01T00:00+01:00[Europe/Paris]` will let programs add six
  months to the timestamp while adjusting for Summer Time (daylight saving time).
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

## Background

{{Section 4.3 of RFC3339}} states that an offset given as `Z` or
`+00:00` implies that "UTC is the preferred reference point for the
specified time".  The offset `-00:00` is provided as a way to express
that "the time in UTC is known, but the offset to local time is
unknown".

This convention mirrors a similar convention for date/time information
in email headers, described in {{Section 3.3 of RFC5322}} and introduced
earlier in {{Section 3.3 of RFC2822}}.
This email header convention is in actual use, while its adaptation into
{{RFC3339}} was always
compromised by the fact that {{ISO8601-2000}} and later versions do not actually allow `-00:00`.

Implementations that needed to express the semantics of `-00:00`
therefore tended to use `Z` instead.

## Update to RFC 3339

This specification updates {{Section 4.3 of RFC3339}}, aligning it with the actual
practice of interpreting the offset `Z` to mean the same as`-00:00`:
"the time in UTC is known, but the offset to local time is unknown".

{{Section 4.3 of RFC3339}} is revised to read as follows:

{:quote}
>
   If the time in UTC is known, but the offset to local time is unknown,
   this can be represented with an offset of "Z".
   (The original version of this specification provided "-00:00" for
   this purpose, which is not allowed by {{ISO8601-2000}} and therefore
   is less interoperable; {{Section 3.3 of RFC5322}} describes a related
   convention for email which does not have this problem).
   This differs semantically from an offset of "+00:00", which implies
   that UTC is the preferred reference point for the specified time.

## Notes

Note that the semantics of the local offset `+00:00` is not updated;
this retains the implication that UTC is the preferred reference point
for the specified time.

Note also that the fact that {{ISO8601-2000}} and later do not allow `-00:00` as a
local offset reduces the level of interoperability that can be
achieved in using this feature; the present specification however does
not formally deprecate this syntax.
With the update to RFC 3339, the local offset `Z` should now be used in its
place.

# Internet Extended Date/Time format (IXDTF) {#date-time-format}

This section discusses desirable qualities of formats for the
timestamp extension suffix and defines the IXDTF format, which extends
{{RFC3339}} for use in Internet protocols.

## Format of Extended Information

The format allows applications to specify additional
important information in addition to a bare {{RFC3339}} timestamp.

This is done by defining *tags*, each with a *key* and
a *value* separated by an equals sign.
The value of a tag can be one or more items delimited by hyphen/minus signs.

Applications can build an informative timestamp *suffix* using any number of
these tags.

Keys are lower-case only.  Values are case-sensitive unless otherwise specified.

See {{optionally-critical}} for the handling of inconsistent information
in a suffix.

## Registering Keys for Extended Information Tags {#registered}

Suffix tag keys are registered by supplying the information
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
leaking out to general production and why that must be prevented.

## Optional Generation, Elective vs. Critical Consumption {#optionally-critical}

For the IXDTF format, suffix tags are always *optional*: They
can be added or left out as desired by the generator of the string.
(An application might require the presence
of specific suffix tags, though.)

Without further indication, suffix tags are also *elective*:
The recipient is free to ignore any suffix tag included in an IXDTF
string.
Reasons might include that the recipient does
not implement (or know about) the specific suffix key, or that it does
recognize the key but cannot act on the value provided.

A suffix tag may also indicate that it is *critical*: The recipient is
advised that it MUST NOT act on the Internet Extended Date/Time Format (IXDTF) string
unless it can process the suffix tag as specified.  A critical suffix
tag is indicated by following its opening bracket with an exclamation
mark (see `critical-flag` in {{abnf}}).

For example, IXDTF strings such as:

    2022-07-08T00:14:07+01:00[Europe/Paris]

are internally inconsistent (see {{inconsistent}}), because Europe/Paris did not
use a time zone offset of `+01:00` in July 2022.
The time zone hint given in the suffix tag is elective, though, so the recipient is not
required to act on the inconsistency; it can treat the Internet
Date/Time Format string as if it were:

    2022-07-08T00:14:07+01:00

<aside markdown="1">

Note that as per {{update}} (see also {{inconsistent}}), the IXDTF string:

    2022-07-08T00:14:07Z[Europe/Paris]

does not exhibit such an inconsistency, as the local offset of `Z`
does not imply a specific preferred time zone of interpretation.
Using the Time Zone Database rules for Europe/Paris in
the summer of 2022, it is equivalent to:

    2022-07-08T02:14:07+02:00[Europe/Paris]

</aside>

Similarly, an unknown suffix may be entirely ignored:

    2022-07-08T00:14:07+01:00[knort=blargel]

(assuming that the recipient does not understand the suffix key `knort`).

In contrast to this elective use of a suffix tag,

    2022-07-08T00:14:07+01:00[!Europe/Paris]
    2022-07-08T00:14:07Z[!u-ca=chinese][u-ca=japanese]
    2022-07-08T00:14:07Z[u-ca=chinese][!u-ca=japanese]
    2022-07-08T00:14:07Z[!knort=blargel]

each have an internal inconsistency or an unrecognized suffix key/value
that are marked as critical, so a recipient MUST treat these IXDTF
strings as erroneous.
This means that the application MUST reject the data, or perform some
other error handling, such as asking the user how to resolve the
inconsistency (see {{inconsistent}}).

Note that applications MAY also perform additional processing on
inconsistent or unrecognized elective suffix tags, such as asking the
user how to resolve the inconsistency.
While they are not required to do so with elective suffix tags, they are
required to reject or perform some other error handling when
encountering inconsistent or unrecognized suffix tags marked as
critical.

An application that encounters duplicate use of a suffix key in
elective suffixes and does not want to perform additional processing
on this inconsistency MUST choose the first suffix that has that key,
i.e.,

    2022-07-08T00:14:07Z[u-ca=chinese][u-ca=japanese]
    2022-07-08T00:14:07Z[u-ca=chinese]

are then treated the same.

## Inconsistent `time-offset`/Time-Zone Information {#inconsistent}

An RFC 3339 timestamp can contain a `time-offset` value that indicates
the offset between local time and UTC (see {{Section 4 of RFC3339}},
noting that {{update}} of the present specification updates {{Section 4.3
of RFC3339}}).

The information given in such a `time-offset` value can be
inconsistent with the information provided in a time zone suffix for an
IXDTF timestamp.

For example, a calendar application could store an IXDTF string representing a
far-future meeting in a particular time zone. If that time zone's definition is
subsequently changed to abolish daylight saving time, IXDTF strings that were
originally consistent may now be inconsistent.

In case of inconsistent `time-offset` and time zone suffix, if the
critical flag is used on the time zone suffix, an application MUST act
on the inconsistency.
If the critical flag is not used, it MAY act on the inconsistency.
Acting on the inconsistency may involve rejecting the timestamp, or
resolving the inconsistency via additional information such as user input
and/or programmed behavior.

For example, the IXDTF timestamps in {{example-inconsistent}} represent
00:14:07 UTC, indicating a local time with a `time-offset` of +00:00.
However, because Europe/London used offset +01:00 in July 2022, the
timestamps are inconsistent in {{example-inconsistent}}, where the first
case is one where the application MUST act on the inconsistency (the
time zone suffix is marked critical), and the second case is one where
it MAY act on it (time zone suffix is elective).

    2022-07-08T00:14:07+00:00[!Europe/London]
    2022-07-08T00:14:07+00:00[Europe/London]
{: #example-inconsistent title="Inconsistent IXDTF timestamps"}

As per {{Section 4.3 of RFC3339}} as updated by {{update}}, IXDTF
timestamps may also forego indicating local time information in their
{{RFC3339}} part by using `Z` instead of a numeric time zone offset.
The IXDTF timestamps in {{example-consistent}} (which represent the same
instant in time as the strings in {{example-inconsistent}}) are not
inconsistent because they do not assert any particular local time nor
local offset in their {{RFC3339}} part.
Instead, applications that receive these strings can calculate the
local offset and local time using the rules of the time zone suffix,
such as Europe/London in the example in {{example-consistent}}, which
like {{example-inconsistent}} has a case with a time zone suffix marked
critical, i.e., the intention is that the application must understand
the time zone information, and one marked elective, which then only is
provided as additional information.

    2022-07-08T00:14:07Z[!Europe/London]
    2022-07-08T00:14:07Z[Europe/London]
{: #example-consistent title="No inconsistency in IXDTF timestamps"}

Note that `-00:00` may be used instead of `Z`, because they have the
same meaning according to {{update}}, but `-00:00` is not allowed by
{{ISO8601-2000}} and later so `Z` is preferred.

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
time-zone-part    = time-zone-initial *time-zone-char
                    ; but not "." or ".."
time-zone-name    = time-zone-part *("/" time-zone-part)
time-zone         = "[" critical-flag
                        time-zone-name / time-numoffset "]"

key-initial       = lcalpha / "_"
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
lcalpha           = %x61-7A
~~~~
{: #grammar title="ABNF grammar of extensions to RFC 3339" sourcecode-name="date-time-ext.abnf"}

Note that a `time-zone` is syntactically similar to a `suffix-tag`,
but does not include an equals sign.
This special case is only available for time zone tags.

The ABNF definition of `time-zone-part` matches "." and "..", which
however both are explicitly excluded (see also comment on
`time-zone-part`).

`time-zone-name` is intended to be the name of an IANA Time Zone.
As generator and recipient may be using different revisions of the
Time Zone Database, recipients may not be aware of such an IANA Time
Zone name and should treat such a situation as any other inconsistency.

{:aside}
>
Note: At the time of writing, the length of each `time-zone-part` is
limited to a maximum of 14 characters by the rules in {{TZDB-NAMING}}.
One platform is known to enforce this limit, an entry in a timezone
database on another platform is known to exceed this limit.
As the `time-zone-name` will ultimately have to be looked up in the
database, which therefore has control over the length, the
`time-zone-part` production in {{grammar}} is deliberately permissive.

## Examples {#date-time-examples}

Here are some examples of Internet Extended Date/Time Format (IXDTF).

~~~~ ixdtf
1996-12-19T16:39:57-08:00
~~~~
{: #rfc3339-datetime title="RFC 3339 date-time with time zone offset"}

{{rfc3339-datetime}} represents 39 minutes and 57 seconds after the 16th hour of
December 19th, 1996 with an offset of -08:00 from UTC.
Note that this is the same instant in time as `1996-12-20T00:39:57Z`, expressed in UTC.

~~~~ ixdtf
1996-12-19T16:39:57-08:00[America/Los_Angeles]
~~~~
{: #datetime-tzname title="Adding a time zone name"}

{{datetime-tzname}} represents the exact same instant in time as the previous example but
additionally specifies the human time zone associated with it
("Pacific Time") for time-zone-aware applications to take into
account.

~~~~ ixdtf
1996-12-19T16:39:57-08:00[America/Los_Angeles][u-ca=hebrew]
~~~~
{: #date-time-hebrew title="Projecting to the Hebrew calendar"}

{{date-time-hebrew}} represents the exact same instant in time, but it informs calendar-aware
applications (see {{calendar}}) that they should project it to the Hebrew calendar.

~~~~ ixdtf
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
consumed by implementations.
The set of suffix values allowed for this suffix key is the set of
values defined for the Unicode Calendar Identifier {{TR35}}.
A resource that has been built to provide links into the most recent
stable and development {{CLDR}} information about that is provided by
{{CLDR-LINKS}}.

# IANA Considerations {#iana-cons}

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFCthis with the RFC
    number of this RFC and remove this note.

IANA is requested to establish a registry called "Timestamp Suffix Tag
Keys" in a new registry group "Internet Date/Time Format".
Each entry in the registry shall consist of the information described in {{registered}}.
Initial contents of the registry are specified in {{tab-registered}}.

| Key Identifier | Registration status | Description:                        | Change controller | Reference             |
| u-ca           | Permanent           | Preferred Calendar for Presentation | IESG              | {{calendar}} of RFCthis |
{: #tab-registered title="Initial Content of Timestamp Suffix Tag Keys registry"}

The registration policy {{BCP26}} is "Specification Required" for
permanent entries, and "Expert Review" for provisional ones.
In the second case, the expert is instructed to ascertain that a basic
specification does exist, even if not complete or published yet.

# Security Considerations

## Excessive Disclosure

The ability to include various pieces of ancillary information with a
timestamp might lead to excessive disclosure.
An example for possibly excessive disclosure is given in {{Section 7 of
RFC3339}}.
Similarly, divulging information about the calendar system or the
language of choice may provide more information about the originator
of a timestamp than the data minimization principle would permit
{{DATA-MINIMIZATION}}.
More generally speaking, generators of IXDTF timestamps need to
consider whether information to be added to the timestamp is
appropriate to divulge to the recipients of this information, and need
to err on the side of minimizing such disclosure if the set of
recipients is not under control of the originator.

## Data Format Implementation Vulnerabilities

As usual when extending the syntax of a data format, this can lead to
new vulnerabilities in implementations parsing and processing the
format.
No considerations are known for the IXDTF syntax that would pose
concerns that are out of the ordinary.

## Operating with Inconsistent Data

Information provided in the various parts of an IXDTF string may be
inconsistent in interesting ways, both with the extensions defined in
this specification (see for instance {{inconsistent}}) and with future
extensions still to be defined.
Where consistent interpretation between multiple actors is required
for security purposes (e.g., where timestamps are embedded as
parameters in access control information), only such extensions can be
employed that have a defined resolution of such inconsistent data.

--- back

# Acknowledgements
{:unnumbered}

Richard Gibson and Justin Grant provided editorial improvements.
The authors would like to thank Francesca Palombini for her AD review.
