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
  RFC8126:
#  BCP47:
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

The format should allow implementations to specify additional
important information in addition to the bare timestamp.
This is done by defining *tags*, each with a *key* and
a *value* separated by an equals sign.
The key of a tag can be split into two parts by including a
hyphen/minus sign "`-`"; the first part (including the "`-`") can then
be used as a namespace.
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

## Namespaced

Suffix keys identify a *namespace*.
By including a hyphen/minus sign "`-`", the namespace can be separated
from the rest of the key; if no hyphen/minus sign is included, the
whole key is the namespace.

For example, if "`u-`" is a namespace for the Unicode Consortium, a
calendar as defined by that organization could be included as
`u-ca=<value>`.

An IANA registry for namespaces can be used to allocate namespaces for
specific applications, as defined in {{iana-cons}}.  Two namespaces are
allocated by the present document:

* "u-" for keys defined by the Unicode Consortium.
* "x-" for keys used within experiments.
  Such keys are not for general interchange and MUST be rejected by a
  recipient unless that is specifically enabled for an experiment.
  See {{?RFC6648}} for additional considerations about "x-" namespaces.

* In addition, for CLDR extensions: [^CLDRfn]
  <!-- What does CLDR extension mean? -->
  * There must be a `namespace-key` and it is restricted to 2
    `alphanum` characters.
  * A `suffix-value` is limited to `3*8alphanum`.

[^CLDRfn]: I don't know how this would be used, so I can't edit this text.
{: source="--- cabo"}

Additional namespaces can be registered under an Expert review policy,
providing a description for the intended use.  This may be a general
concept, or a specific organization that is intended to register keys
within this namespace.

## Registered

Actual keys are registered by supplying the information in {{record}}:

~~~~
%%
Identifier:
Description:
Comments:
Added:
RFC:
Authority:
Contact_Email:
Mailing_List:
URL:
%%
~~~~
{: #record title="Registration record for a tag key"}

'Identifier' contains the key name.
A key name ending with a hyphen/minus sign "`-`" is a namespace
that covers all timestamp tag keys beginning with that prefix.

'Description' contains the name and description of the namespace.

'Comments' is an OPTIONAL field and MAY contain a broader description
of the namespace.
<!-- Is the field optional or its contents? -->

'Added' contains the date the key's definition was published in the
"date-full" format specified in {{grammar}}.
For example: 2004-06-28 represents June 28, 2004, in the Gregorian
calendar.
<!-- That information is redundant with the RFC -->

'RFC' contains the RFC number assigned to the namespace.

'Authority' contains the name of the maintaining authority for the
namespace.

'Contact_Email' contains the email address used to contact the
maintaining authority.

'Mailing_List' contains the URL or subscription email address of the
mailing list used by the maintaining authority.

'URL' contains the URL of the registry for this namespace.


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

namespace         = 1*alphanum
namespace-key     = 1*alphanum
suffix-key        = namespace ["-" namespace-key]

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
1996-12-19T16:39:57-08:00[x-foo=bar][x-baz=bat]
~~~~
{: #date-time-private title="Adding tags in private use namespaces"}

{{date-time-private}}, based on {{rfc3339-datetime}}, utilizes the private use namespace to declare two
additional pieces of information in the suffix that can be interpreted
by any compatible implementations and ignored otherwise.

# IANA Considerations {#iana-cons}

IANA is requested to establish a registry called "Timestamp Suffix Tag Keys".
Each entry in the registry shall consist of the information described in {{registered}}.
Initial contents of the registry are specified in {{namespaced}}.

The policy is "RFC required", "Specification Required", ???[^policy]
{{RFC8126}}.

[^policy]: We need to define the policy for both namespaces and full keys.
{: source="--- cabo"}



# Security Considerations

TBD

--- back

# Acknowledgements

TBD
