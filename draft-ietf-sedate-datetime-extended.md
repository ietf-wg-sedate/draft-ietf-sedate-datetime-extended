---
stand_alone: true
ipr: trust200902
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
abbrev: Timestamps Extended
wg: Serialising Extended Data About Times and Events

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

normative:
#  RFC2026:
#  RFC2028:
  RFC3339:
  RFC5234: abnf
#  RFC5646: # in BCP47
  RFC6838: media-types
  RFC8126:
#  BCP47:
  BCP178:
informative:
  # obsolete, but needed for its Appendix E:
  RFC1305: ntp-old
  ISO8601:
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

--- abstract


This document defines an extension to the timestamp format defined in
RFC3339 for representing additional information including a time
zone.

--- middle

# Introduction {#intro}

Dates and times are used in a very diverse set of internet
applications, all the way from server-side logging to calendaring and
scheduling.

Each distinct instant in time can be represented in a descriptive text
format using a timestamp, and {{ISO8601}} standardizes a widely-adopted
timestamp format, which forms the basis of {{RFC3339}}.
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
  time zone names to timestamps ({{JAVAZDT}}).

* The format provides a generalized way to attach any additional
  information to the timestamp.


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
* The use of time scales different from UTC, such as TAI.

However, additional information augmenting a fixed timestamp may be
sufficient to detect an inconsistency between intention and the actual
information in the timestamp, e.g., between the UTC offset and time zone
name in the timestamp.
For instance, such an inconsistency might arise because of:

* Political decisions as discussed above, or
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

  UTC is often mistakenly referred to as GMT, an earlier time scale
  UTC was designed to be a useful successor for.

ABNF:
: Augmented Backus-Naur Form, a format used to represent permissible
  strings in a protocol or language, as defined in {{RFC5234}}.
  The rules defined in {{Appendix B of RFC5234}} are imported implicitly.

Internet Date/Time Format:
: The date/time format defined in section 3 of this document.

Timestamp:
: An unambiguous representation of some instant in time.

Z:
: A suffix which, when applied to a time, denotes a UTC offset of
  00:00; often spoken "Zulu" from the ICAO phonetic alphabet
  representation of the letter "Z" (from {{Section 2 of RFC3339}}).

Time Zone:
: A time zone that is included in the Time Zone Database (often
  called `tz` or `zoneinfo`) maintained by IANA.
  <!-- ref needed -->

CLDR:
: Common locale data repository {{CLDR}}, a project of the Unicode
  Consortium to provide locale data to applications.

For more information about time scales, see {{Appendix E of RFC1305}},
Section 3 of {{ISO8601}}, and the appropriate ITU documents
{{ITU-R-TF.460-6}}.

# Extended Date/Time format {#date-time-format}

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
: The key.

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

# Syntax Extensions to RFC 3339

## ABNF

The following rules extend the ABNF syntax defined in {{RFC3339}} in
order to allow the inclusion of an optional suffix.

The extended date/time format is described by the rule
`date-time-ext`.
`date-time` is imported from {{Section 5.6 of RFC3339}}, `ALPHA` and
`DIGIT` from {{Section B.1 of RFC5234}}.

~~~~ abnf
time-zone-initial = ALPHA / "." / "_"
time-zone-char    = time-zone-initial / DIGIT / "-" / "+"
time-zone-part    = time-zone-initial *13(time-zone-char)
                    ; but not "." or ".."
time-zone-name    = time-zone-part *("/" time-zone-part)
time-zone         = "[" time-zone-name "]"

key-initial       = ALPHA / "_"
key-char          = key-initial / DIGIT / "-"
suffix-key        = key-initial *key-char

suffix-value      = 1*alphanum
suffix-values     = suffix-value *("-" suffix-value)
suffix-tag        = "[" suffix-key "=" suffix-values "]"
suffix            = [time-zone] *suffix-tag

date-time-ext     = date-time suffix

alphanum          = ALPHA / DIGIT
~~~~
{: #grammar title="ABNF grammar of extensions to RFC 3339"}

Note that a `time-zone` is syntactically similar to a `suffix-tag`,
but does not include an equals sign.
This special case is only available for time zone tags.

## Examples {#date-time-examples}

Here are some examples of Internet extended date/time format.
<!-- We need to agree a name for the baby. -->

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
implementations that they should project it to the Hebrew calendar.

~~~~
1996-12-19T16:39:57-08:00[_foo=bar][_baz=bat]
~~~~
{: #date-time-private title="Adding experimental tags"}

{{date-time-private}}, based on {{rfc3339-datetime}}, utilizes keys
identified as experimental by a leading underscore to declare two additional pieces of
information in the suffix; these can be interpreted by implementations
that take part in the controlled experiment making use of these tag keys.

# IANA Considerations {#iana-cons}

IANA is requested to establish a registry called "Timestamp Suffix Tag Keys".
Each entry in the registry shall consist of the information described in {{registered}}.
Initial contents of the registry are specified in {{registered}}.

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

