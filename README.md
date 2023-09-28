# PEVR: The Proper Email Validation RegEx

Any web developer worth their salt has probably implemented a regular expression to validate e-mail addresses at one point or another. Most of those will probably amount to something along the lines of ``[A-Za-z0-9\.\-]+@[A-Za-z0-9\.\-]+\.[A-Za-z]+``, simply matching something in front of the @, then some domain name after that. This is generally fine, but will fail to validate a lot of e-mail addresses that would be legal as per [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.html#section-4.1.2).

PEVR aims to solve this by offering a regular expression that will *every* legal e-mail address as specified in [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.html#section-4.1.2). The RegEx looks like this (with line breaks added for something resembling readability):

```regex
^
(
([a-zA-Z0-9\!#-'\*\+\-\/\=\?\^-`{-~]+(\.[a-zA-Z0-9\!#-'\*\+\-\/\=\?\^-`{-~]+)*)|("((\\[ -~])|([ \!\#-\[\]-~]))*")
)
@
(
([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?(\.[a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?)*)|
(
\[
(
(\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?(\.(\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?)){3})|
(
IPv6:
(
([0-9a-f]{1,4}(:[0-9a-f]{1,4}){1,7})|
(([0-9a-f]{1,4}:){1,1}(:[0-9a-f]{1,4}){1,6})|
(([0-9a-f]{1,4}:){1,2}(:[0-9a-f]{1,4}){1,5})|
(([0-9a-f]{1,4}:){1,3}(:[0-9a-f]{1,4}){1,4})|
(([0-9a-f]{1,4}:){1,4}(:[0-9a-f]{1,4}){1,3})|
(([0-9a-f]{1,4}:){1,5}(:[0-9a-f]{1,4}){1,2})|
(([0-9a-f]{1,4}:){1,6}(:[0-9a-f]{1,4}){1,1})|
((([0-9a-f]{1,4}:){1,7}|:):)|
(:(:[0-9a-f]{1,4}){1,7})|
(((([0-9a-f]{1,4}:){6})(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3}))|
((([0-9a-f]{1,4}:){5}[0-9a-f]{1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3}))|
(([0-9a-f]{1,4}:){5}:[0-9a-f]{1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,1}(:[0-9a-f]{1,4}){1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,2}(:[0-9a-f]{1,4}){1,3}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,3}(:[0-9a-f]{1,4}){1,2}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,4}(:[0-9a-f]{1,4}){1,1}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
((([0-9a-f]{1,4}:){1,5}|:):(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(:(:[0-9a-f]{1,4}){1,5}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})
)
)
)
\]
)
)
$
```

A detailed breakdown of how this RegEx works can be found below. Test cases that will validate the RegEx's functionality can be found in [tests.txt](./tests.txt), Test cases that shouldn't be validated can be found in [tests-negative.txt](./tests-negative.txt). I don't think the test cases have 100% coverage, feel free to submit a PR to add tests if you can think of any missing ones.

Note that ***PEVR SHOULD NOT BE USED IN PRODUCTION SYSTEMS***. This is purely made for comedic purposes, and most major e-mail providers will not be able to deal with the more unusual kinds of e-mail addresses [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.html#section-4.1.2) and, by extension, PEVR allows.

This project was inspired by Dylan Beattie's talk ["Email vs Capitalism, or, Why We Can't Have Nice Things"](https://www.youtube.com/watch?v=mrGfahzt-4Q&t=990s) ([privacy friendly link](https://piped.video/watch?v=mrGfahzt-4Q&t=990s)), where he points out some of the things [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.html#section-4.1.2) allows that most e-mail providers don't.

## Breakdown

[RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.html#section-4.1.2) defines a valid SMTP mailbox as ``{{Local-part}}@({{Domain}}|{{address-literal}})``. Note that the words in double curly braces are placeholders for parts defined further down below. Local-part and Domain/address-literal will be addressed separately.

A ``^`` at the beginning and ``$`` at the end are also added to ensure the entire string matches the RegEx, not just part of it.

### Local-part

The local part can be either a ``Dot-string`` or a ``Quoted-string``.

A Dot-string is defined as ``[a-zA-Z0-9\!#-'\*\+\-\/\=\?\^-`{-~]+(\.[a-zA-Z0-9\!#-'\*\+\-\/\=\?\^-`{-~]+)*``, a string of all printable ASCII characters excluding special ones.

A Quoted-string is a lot more open, defined as ``"({{quoted-pairSMTP}}|{{qtextSMTP}})*"``, with ``quoted-pairSMTP`` as ``\\[ -~]`` and ``qtextSMTP`` as ``[ \!\#-\[\]-~]``. The full segment for Quoted-string is thus ``"((\\[ -~])|([ \!\#-\[\]-~]))*"``.

The full segment for the Local-part is thus

```regex
(
([a-zA-Z0-9\!#-'\*\+\-\/\=\?\^-`{-~]+(\.[a-zA-Z0-9\!#-'\*\+\-\/\=\?\^-`{-~]+)*)|("((\\[ -~])|([ \!\#-\[\]-~]))*")
)
```

### Domain

Domains are defined as ``{{sub-domain}}(\.{{sub-domain}})*``. Each sub-domain is defined as ``[a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?``.

The full segment for domains is thus ``[a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?(\.[a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?)*``

### address-literal

An address literal is simply defined as ``\[{{IPv4-address-literal}}|{{IPv6-address-literal}}|{{General-address-literal}}\]``.

An ``IPv4-address-literal`` is, well, an IPv4 address, defined as ``{{IPv4-segment}}(\.{{IPv4-segment}}){3}``. A single ``IPv4-segment`` looks like this: ``(\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?)`` (from [iHateRegex](https://ihateregex.io/expr/ip/)), which just matches any number between 0 to 255. Note that this doesn't allow leading zeroes, although RFC 5321 doesn't make a clear statement on whether they should be allowed. TODO: Allow leading zeros. The full segment for IPv4 is thus ``\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?(\.\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?){3}``.

An ``IPv6-address-literal`` is an IPv6-address, defined as ``IPv6:({{IPv6-full}}|{{IPv6-comp}}|{{IPv6v4-full}}|{{IPv6v4-comp}})``. The ``Ipv6v4-*`` placeholders are equivalent to their corresponding ``IPv6-*`` placeholders, except the last two groups are replaced by an ``IPv4-address-literal``. ``IPv6-comp`` allows for replacing empty parts of the address with ``::``. To make life a little easier, this uses [an IPv6 RegEx by Vernon Mauery](https://web.archive.org/web/20120302212319/https://vernon.mauery.com/content/projects/linux/ipv6_regex) that looks like this:

```regex
IPv6:
(
([0-9a-f]{1,4}(:[0-9a-f]{1,4}){1,7})|
(([0-9a-f]{1,4}:){1,1}(:[0-9a-f]{1,4}){1,6})|
(([0-9a-f]{1,4}:){1,2}(:[0-9a-f]{1,4}){1,5})|
(([0-9a-f]{1,4}:){1,3}(:[0-9a-f]{1,4}){1,4})|
(([0-9a-f]{1,4}:){1,4}(:[0-9a-f]{1,4}){1,3})|
(([0-9a-f]{1,4}:){1,5}(:[0-9a-f]{1,4}){1,2})|
(([0-9a-f]{1,4}:){1,6}(:[0-9a-f]{1,4}){1,1})|
((([0-9a-f]{1,4}:){1,7}|:):)|
(:(:[0-9a-f]{1,4}){1,7})|
(((([0-9a-f]{1,4}:){6})(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3}))|
((([0-9a-f]{1,4}:){5}[0-9a-f]{1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3}))|
(([0-9a-f]{1,4}:){5}:[0-9a-f]{1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,1}(:[0-9a-f]{1,4}){1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,2}(:[0-9a-f]{1,4}){1,3}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,3}(:[0-9a-f]{1,4}){1,2}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,4}(:[0-9a-f]{1,4}){1,1}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
((([0-9a-f]{1,4}:){1,5}|:):(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(:(:[0-9a-f]{1,4}){1,5}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})
)
```

The only things changed from the original are removal of the ``\A`` and ``\Z`` as well as adding the first case (because the original regex had no case for fully formed IPv6 addresses).

A ``General-address-literal`` serves as a stand-in for any protocols defined in the future, defined as ``{{Standardized-tag}}:({{dcontent}})+``. ``Standardized-tag`` can be anything matching ``([A-Za-z0-9\-])*[A-Za-z0-9]``, but it is defined that "Standardized-tag MUST be specified in a Standards-Track RFC and registered with IANA". Since this RegEx must work in the future too and protocols might be added in the future, any tag will be allowed. ``dcontent`` is simply defined as any printable US-ASCII except ``[``, ``]`` and ``\``, so ``([!-Z]|[\^-~])``. The full segment for General address literals is thus ``([A-Za-z0-9\-])*[A-Za-z0-9]:([!-Z]|[\^-~])+``.

While General-address-literals are interesting, they will be excluded from the RegEx as I am not aware of any actual other new protocols accepted by IANA and it currently leads to invalidly formatted IPv6 addresses being accepted.

The full segment for ``address-literal`` can thus be boiled down to

```regex
\[
(
(\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?(\.\b25[0-5]|\b2[0-4][0-9]|\b[01]?[0-9][0-9]?){3})|
(
IPv6:
(
(([0-9a-f]{1,4}:){1,1}(:[0-9a-f]{1,4}){1,6})|
(([0-9a-f]{1,4}:){1,2}(:[0-9a-f]{1,4}){1,5})|
(([0-9a-f]{1,4}:){1,3}(:[0-9a-f]{1,4}){1,4})|
(([0-9a-f]{1,4}:){1,4}(:[0-9a-f]{1,4}){1,3})|
(([0-9a-f]{1,4}:){1,5}(:[0-9a-f]{1,4}){1,2})|
(([0-9a-f]{1,4}:){1,6}(:[0-9a-f]{1,4}){1,1})|
((([0-9a-f]{1,4}:){1,7}|:):)|
(:(:[0-9a-f]{1,4}){1,7})|
(((([0-9a-f]{1,4}:){6})(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3}))|
((([0-9a-f]{1,4}:){5}[0-9a-f]{1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3}))|
(([0-9a-f]{1,4}:){5}:[0-9a-f]{1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,1}(:[0-9a-f]{1,4}){1,4}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,2}(:[0-9a-f]{1,4}){1,3}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,3}(:[0-9a-f]{1,4}){1,2}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(([0-9a-f]{1,4}:){1,4}(:[0-9a-f]{1,4}){1,1}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
((([0-9a-f]{1,4}:){1,5}|:):(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})|
(:(:[0-9a-f]{1,4}){1,5}:(25[0-5]|2[0-4]\d|[0-1]?\d?\d)(\.(25[0-5]|2[0-4]\d|[0-1]?\d?\d)){3})
)
)
)
\]
```
