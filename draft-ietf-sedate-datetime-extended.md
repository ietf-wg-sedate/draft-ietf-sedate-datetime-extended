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
  RFC2026:
  RFC2028:
  RFC3339:
  RFC5234: abnf
#  RFC5646: # in BCP47
  RFC8126:
  BCP47:
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
instant with an associated time zone name, to take into account things
like DST transitions.
Most of these applications attach the timezone to the timestamp in a
non-standard format, at least one of which is fairly well-adopted {{JAVAZDT}}.
Furthermore, applications might want to attach even more information to the
timestamp, including but not limited to the calendar system it needs
to be represented in.

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
semantic result no longer is a fixed timestamp that is referenced to a
(past or future) UTC time.
For instance, it does not address:

* Future time given as a local time in some specified time zone, where
  changes to the definition of that time zone (e.g., a political
  decision to enact or rescind daylight savings time) changes the
  actual time in UTC time.
* Time given as a "floating time", i.e., a local time without
  information as to which time zone this local time refers to.
* The use of time scales different from UTC, such as TAI.

However, the additional information provided that augments a fixed
timestamp may be sufficient to detect an inconsistency between
intention and the actual information given in the timestamp, e.g.,
between the additional timezone information given and the timezone
offset recorded in the timestamp.
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
  From 1972 through 1987 UTC was maintained entirely by Bureau
  International de l'Heure (BIH).
  Before 1972 UTC was not generally recognized and civil time was
  determined by individual jurisdictions using different techniques
  for attempting to follow Universal Time based on measuring the
  rotation of the earth.

  UTC is often mistakenly referred to as GMT, an earlier timescale
  UTC was designed to be a useful successor for.

ABNF:
: Augmented Backus-Naur Form, a format used to represent permissible
  strings in a protocol or language, as defined in {{RFC5234}}.
  The rules defined in {{Appendix B of RFC5234}} are imported implicitly.

Internet Date/Time Format:
: The date/time format defined in section 3 of this document.

Timestamp:
: This term is used in this document to refer to an unambiguous
  representation of some instant in time.

Z:
: A suffix which, when applied to a time, denotes a UTC offset of
  00:00; often spoken "Zulu" from the ICAO phonetic alphabet
  representation of the letter "Z".
  <!-- Shouldn't we just import this term from RFC 3339? -->

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
a *value* separated by an equals sign, and
allowing implementations to include an informative
*suffix* at the end with as many tags as required.
The value of a tag can be a hyphen delimited list of multiple values.

In case a key is repeated or conflicted, implementations MUST give
precedence to whichever value is positioned first.
<!-- needs example and/or defn of "conflicted" -->

## Namespaced

Since tags can include all sorts of additional information,
different standards bodies/organizations need a way to identify which
part adheres to their standards.
For this, all information needs to be namespaced.
Each key is therefore divided into two hyphen-separated sections: the
namespace and the key.
For example, the calendar as defined by the Unicode consortium could
be included as `u-ca=<value>`.

All single-character namespaces are reserved for {{BCP47}} extensions
recorded in the BCP47 extensions registry.
<!-- What is "the BCP47 extensions registry"? -->
For these namespaces:
<!-- Which of these are just for one-alnum namespaces, which are more general? -->

* Case differences are ignored.
  <!-- everywhere?  Use "case-insensitive" as a term.  -->

* The namespace is restricted to single alphanum, corresponding to
  extension singletons ('x' can be used for a private use extension).
  <!-- need to explain that we are using ABNF terms in plain text -->

* In addition, for CLDR extensions:
  <!-- What does CLDR extension mean? -->
  * There must be a `namespace-key` and it is restricted to 2
    `alphanum` characters.
  * A `suffix-value` is limited to `3*8alphanum`.

## Multi-character Namespaces {#multi-char}

Multi-character namespaces can be registered specifically for use in
this format, see {{iana-cons}}.
The registration policy requires the development of an RFC,
which SHALL define the
name, purpose, processes, and procedures for maintaining the tags
using the namespace registered.

(This subsection uses BCP 14 language to describe the requirements on
the information interchanged indirectly by providing requirements on
the RFC registering a namespace and the principles of its evolution.)

The maintaining or registering authority, including name, contact
email, discussion list email, and URL location of the registry, MUST
be indicated clearly in the RFC.
<!-- Can we mandate a "discussion list email"? -->
The RFC MUST specify each of the following (directly or included by reference):

* The specification MUST reference the specific version or revision of
  this document that governs its creation and MUST reference this
  section of this document.

* The specification and all keys defined by the specification MUST
  follow the ABNF and other rules for the formation of keys as defined
  in this document.
  <!-- I hope, not just the keys, but also the values -->
  In particular, it MUST specify that case is not significant and that
  keys MUST NOT exceed eight characters in length.

* The specification MUST specify a canonical representation.
  <!-- What does that mean?  -->

* The specification of valid keys MUST be available over the Internet
  and at no cost.
  <!-- There is no such thing as "no cost".  More importantly, also
  access to the specification cannot require agreeing to onerous legal
  requirements such as an NDA. -->

* The specification MUST be in the public domain or available via a
  royalty-free license acceptable to the IETF and specified in the
  RFC.
  <!-- Public domain is not a concept available to an international
  organization.  How is "acceptable to the IETF" defined?  I believe
  that this item talks about copyright, not other forms of licensing
  such as trademark or patent licensing.  -->

* The specification MUST be versioned, and each version of the
  specification MUST be numbered, dated, and stable.
  <!-- Do we have any opinion what that version is being used for? -->

* The specification MUST be stable.
  That is, namespace keys, once defined by a specification, MUST NOT
  be retracted or change in meaning in any substantial way.

* The specification MUST include, in a separate section, the
  registration form reproduced in this section (below) to be used in
  registering the namespace upon publication as an RFC.
  <!-- Do you mean the IANA registration template? -->

* IANA MUST be informed of changes to the contact information and URL
  for the specification.
  <!-- That is not even a requirement on the RFC. -->

<!-- The following also aren't requirements on the RFC. -->

IANA will maintain a registry of allocated multi-character namespaces.
This registry MUST use the record-jar format described by the ABNF in
{{BCP47}}.
<!-- explain what that means -->
<!-- The MUST appears to be a requirement on IANA, which is not the -->
<!-- way this works... -->
Upon publication of a namespace as an RFC, the maintaining authority
defined in the RFC MUST forward this registration form to
\<[](mailto:iesg@ietf.org)>, who MUST forward the request to
\<[](mailto:iana@iana.org)>.
<!-- wait, didn't we just publish an RFC - - why not put the form in there? -->
The maintaining authority of the namespace MUST maintain the accuracy
of the record by sending an updated full copy of the record to
\<[](mailto:iana@iana.org)> with the subject line "TIMESTAMP FORMAT
NAMESPACE UPDATE" whenever content changes.
Only the 'Comments', 'Contact_Email', 'Mailing_List', and 'URL' fields
MAY be modified in these updates.

Failure to maintain this record, maintain the corresponding registry,
or meet other conditions imposed by this section of this document MAY
be appealed to the IESG {{RFC2028}} under the same rules as other IETF
decisions (see {{RFC2026}}) and MAY result in the authority to maintain
the extension being withdrawn or reassigned by the IESG.

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
{: #record title="Registration record for a multi-character namespace"}

'Identifier' contains the multi-character sequence assigned to the
namespace.
The Internet-Draft submitted to define the namespace SHOULD specify
which sequence to use, although the IESG MAY change the assignment
when approving the RFC.
<!-- This is way too specific.  Obviously, the IESG has final control -->
<!-- over that name. -->

'Description' contains the name and description of the namespace.

'Comments' is an OPTIONAL field and MAY contain a broader description
of the namespace.
<!-- Is the field optional or its contents? -->

'Added' contains the date the namespace's RFC was published in the
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

The determination of whether an Internet-Draft meets the above
conditions and the decision to grant or withhold such authority rests
solely with the IESG and is subject to the normal review and appeals
process associated with the RFC process.
<!-- Again, we shouldn't second-guess RFC 2026. -->

## Syntax Extensions to RFC 3339

The following rules extend the ABNF syntax defined in {{RFC3339}} in
order to allow the inclusion of an optional suffix: the extended
date/time format is described by the rule `date-time-ext`.

~~~~
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


## Examples {#date-time-examples}

Here are some examples of Internet extended date/time format.
<!-- We need to agree a name for the baby. -->

~~~~
1996-12-19T16:39:57-08:00
~~~~
{: #rfc3339-datetime title="RFC 3339 date-time with timezone offset"}

{{rfc3339-datetime}} represents 39 minutes and 57 seconds after the 16th hour of
December 19th, 1996 with an offset of -08:00 from UTC.
Note that this is the same instant in time as `1996-12-20T00:39:57Z`, expressed in UTC.

~~~~
1996-12-19T16:39:57-08:00[America/Los_Angeles]
~~~~
{: #datetime-tzname title="Adding a timezone name"}

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

Multi-character namespaces are assigned by IANA using the "IETF
Review" policy defined by {{RFC8126}}; the IETF review process needs to
be based on the requirements laid out in {{multi-char}}.

# Security Considerations

TBD

--- back

# Acknowledgements

TBD
