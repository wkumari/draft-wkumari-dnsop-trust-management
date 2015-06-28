**Important:** Read CONTRIBUTING.md before submitting feedback or contributing
```




template                                                       W. Kumari
Internet-Draft                                                    Google
Intended status: Informational                             June 26, 2015
Expires: December 28, 2015


       Simplified Updates of DNS Security (DNSSEC) Trust Anchors
                  draft-wkumari-dnsop-trust-management

Abstract

   This document describes a simple means for automated updating of
   DNSSEC trust anchors.  This mechanism allows the trust anchor
   maintainer to monitor the progress of the migration to the new trust
   anchor, and so predict the effect before decommisioning the existing
   trust anchor.

   [ Ed note - informal summary: One of the big issues with rolling the
   root key is that it is unclear who all is using RFC5011, who all has
   successfully fetched and installed the new key, and, most
   importantly, who all will die when the old key is revoked.  A
   secondary problem is that the response sizes suddenly increase,
   potentially blowing the MTU limit.  This document describes a method
   that is basically CDS, but for the root key (or any other trust
   anchor).  Unlike the CDS record though, this record lives at a
   special name that communicates what all TAs the recursive knows.
   This allows the TA maintainer to predict how many, and who all will
   break...]

   [ Ed note: Text inside square brackets ([]) is additional background
   information, answers to frequently asked questions, general musings,
   etc.  They will be removed before publication.]

   [ This document is being collaborated on in Github at:
   https://github.com/wkumari/<TEMPLATE>.  The most recent version of
   the document, open issues, etc should all be available here.  The
   authors (gratefully) accept pull requests ]

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF).  Note that other groups may also distribute
   working documents as Internet-Drafts.  The list of current Internet-
   Drafts is at http://datatracker.ietf.org/drafts/current/.




Kumari                  Expires December 28, 2015               [Page 1]

Internet-Draft    draft-wkumari-dnsop-trust-management         June 2015


   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any
   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   This Internet-Draft will expire on December 28, 2015.

Copyright Notice

   Copyright (c) 2015 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents
   (http://trustee.ietf.org/license-info) in effect on the date of
   publication of this document.  Please review these documents
   carefully, as they describe your rights and restrictions with respect
   to this document.  Code Components extracted from this document must
   include Simplified BSD License text as described in Section 4.e of
   the Trust Legal Provisions and are provided without warranty as
   described in the Simplified BSD License.

Table of Contents

   1.  Introduction  . . . . . . . . . . . . . . . . . . . . . . . .   2
     1.1.  Requirements notation . . . . . . . . . . . . . . . . . .   3
   2.  TDS Record Format . . . . . . . . . . . . . . . . . . . . . .   3
     2.1.  TDS Owner Name  . . . . . . . . . . . . . . . . . . . . .   3
   3.  TDS Record Processing . . . . . . . . . . . . . . . . . . . .   4
   4.  IANA Considerations . . . . . . . . . . . . . . . . . . . . .   5
   5.  Security Considerations . . . . . . . . . . . . . . . . . . .   5
   6.  Acknowledgements  . . . . . . . . . . . . . . . . . . . . . .   5
   7.  References  . . . . . . . . . . . . . . . . . . . . . . . . .   5
     7.1.  Normative References  . . . . . . . . . . . . . . . . . .   5
     7.2.  Informative References  . . . . . . . . . . . . . . . . .   5
   Appendix A.  Changes / Author Notes.  . . . . . . . . . . . . . .   6
   Appendix B.  Worked example . . . . . . . . . . . . . . . . . . .   6
   Author's Address  . . . . . . . . . . . . . . . . . . . . . . . .   7

1.  Introduction

   When a DNSSEC aware resolver performs validation, it requires a trust
   anchor to validate the DNSSEC chain.  An example of a trust anchor is
   the so called DNSSEC "root key".  For a variety of reasons this trust
   anchor may need to be replaced or "rolled", to a new key (potentially
   with a different algorithm, different key length, etc.).





Kumari                  Expires December 28, 2015               [Page 2]

Internet-Draft    draft-wkumari-dnsop-trust-management         June 2015


   [RFC5011] provides a secure mechanism to do this, but operational
   experience has demonstrated a need for some additional functionality
   that was not forseen.

   During the recent effort to roll the IANA DNSSEC "root key", it has
   become clear that, in order to predict (and minimize) outages caused
   by rolling the key, one needs to know who does not have the new key.
   In addition, RFC5011 style key rolls require "double signing", which
   significantly increases the size of the responses.

   This document defines a new record type, Trust DS (TDS), which
   provides a mechanism very similar to the Child DS (CDS) [RFC7344]
   record, and some practices for using it.  Readers of this document
   are expected to be familiar with the contents of [RFC7344].

1.1.  Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

2.  TDS Record Format

   The wire and presentation format of the Trust DS (TDS) resource
   record is identical to the DS record [RFC4034].

   IANA has allocated RR code TBD for the TDS resource record via Expert
   Review [DNS-TRANSPORT].  The TDS RR uses the same registries as DS
   for its fields.  No special processing is performed by authoritative
   servers or by resolvers, when serving or resolving.

   For all practical purposes, TDS is a regular RR type.

2.1.  TDS Owner Name

   Much of the purpose of the mechanism described in this document is to
   provide a mechanism to allow the trust anchor maintainer to determine
   how widely deployed the trust anchor is, and who is using an outdated
   trust anchor.  This information is signalled from the validating
   resolver to the authoritive server serving the zone in which the
   Trust Anchor lives.

   This information is available from looking at queries to DNS servers
   serving the DNSKEY for the zone; each resolver using this mechanism
   will periodically query the zone for a name encoding the list of
   trust anchors it is using for that zone.

   This name is computed as follows:



Kumari                  Expires December 28, 2015               [Page 3]

Internet-Draft    draft-wkumari-dnsop-trust-management         June 2015


   1.  Take the Key Tags of all of the DS records corresponding to the
       TA(s) that the resolver knows / is using.

   2.  Sort this list in numerically ascending order

   3.  Concatenate the list, separating each Key Tag with an underscore
       ('_')

   As an example, if the resolver has a single Trust Anchor with a Key
   Tag of 4217, it would generate an owner name of 4217.  If it has two
   Trust Anchors, with Key Tags 4217 and 1776 it would generate an owner
   name of 1776_4217.

   NOTE: This generation of the TDS Name means that Key Tags MUST be
   unique / must not repeat within "recent" history.  If (e.g during a
   Key Ceremony) a new DNSKEY is generated whose derived Key Tag
   collides with an exiting one (statistically unlikely, but not
   impossible) this DNSKEY MUST NOT be used, and a new DNSKEY MUST be
   generated.

3.  TDS Record Processing

   A compliant recursive resolver will periodically (every 'Active
   Refresh' interval ([RFC5011] Section 2.3)) query the trust point
   domain for the TDS Owner Name.  It will receive back either an error
   (e.g NoError / NoData), or a TDS RRSet.  It will validate the TDS
   record, using standard DNSSEC logic.

   Assuming a TDS RRSet is received and validates, the resolver will
   parse the RRSet.  The RRSet will contain one or more TDS records,
   listing the DS records that correspond to DNSKEYs that may sign the
   zone.  The resolver SHOULD store this list to its configuration /
   persistent storage.

   [Ed note: See Appendix B for a worked example of performing a keyroll
   using this mechanism.  It's much less complex than this all makes it
   sound...]

   [Ed note - Corner cases.  I didn't want to spend too long writing out
   all of the handling for these until I've gotten some feedback on the
   concept:]

   1.  The TDS doesn't validate.  This is the same as in a non-TDS /
       RFC5011 world.  You entered the TA incorrectly, you are under
       attack, or similar.

   2.  There is no TDS record (you get NoError / NoData).  You have
       somehow become out of sync with the system, or someone has



Kumari                  Expires December 28, 2015               [Page 4]

Internet-Draft    draft-wkumari-dnsop-trust-management         June 2015


       bungled the keyroll in an odd way.  Panicking is a good option
       here.

4.  IANA Considerations

   This document contains no IANA considerations.Template: Fill this in!

5.  Security Considerations

   TODO: Fill this out!

6.  Acknowledgements

   This idea was discussed with Joe Abley, Paul Wouters, David Conrad
   and Ed Lewis.  I seem to remember mumbling it at a number of others
   too, but have forgotten whom.  Joe and Paul have agreed to co-author,
   but I have not listed them yet, as I want them to read this first /
   not surprise them.

7.  References

7.1.  Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, March 1997.

   [RFC4034]  Arends, R., Austein, R., Larson, M., Massey, D., and S.
              Rose, "Resource Records for the DNS Security Extensions",
              RFC 4034, March 2005.

   [RFC5011]  StJohns, M., "Automated Updates of DNS Security (DNSSEC)
              Trust Anchors", STD 74, RFC 5011, September 2007.

   [RFC7344]  Kumari, W., Gudmundsson, O., and G. Barwood, "Automating
              DNSSEC Delegation Trust Maintenance", RFC 7344, September
              2014.

7.2.  Informative References

   [I-D.ietf-sidr-iana-objects]
              Manderson, T., Vegoda, L., and S. Kent, "RPKI Objects
              issued by IANA", draft-ietf-sidr-iana-objects-03 (work in
              progress), May 2011.








Kumari                  Expires December 28, 2015               [Page 5]

Internet-Draft    draft-wkumari-dnsop-trust-management         June 2015


Appendix A.  Changes / Author Notes.

   [RFC Editor: Please remove this section before publication ]

   From -00 to -01.

   o  Nothing changed in the template!

Appendix B.  Worked example

   This section provides an example of rolling the root trust anchor
   from a DNSKEY with Key Tag 17 to one with Key Tag 42.  It is written
   informally, and will be tidied up / made more formal before
   publication.  To keep this readable, I've made key tags and hashes
   and such be short.

   The root trust anchor is RSA/SHA-1, generating a DS with SHA-1 the DS
   works out to 17 5 1 111222 #(Tag RSA/SHA1 SHA1 Key).  This DS is
   installed into a root zone in a TDS record:

   17 IN TDS 17 5 1 111222

   Compliant resolvers are configured with this information, by manually
   placing this in thier config files (in the same way resolvers are
   currently manually configred with the DNSKEY).  The resolver will
   periodically query the root for qname 17, type TDS.  It will receive
   (and validate!) this TDS record, will see that is has this key, and
   will go back to sleep.  The root TA maintainer can see that everyone
   is using the key with ID 17.

   Eventually the trust anchor maintainer withes to roll to a new DSA/
   SHA-1 key, so they generate the new key.  They compute the DS (again
   using SHA-1) and the computed DS is 42 (Tag) 3 (DSA/SHA-1) 1 (SHA1)
   333444.  They now publish TDS records as follows:

   17    IN TDS 17 5 1 111222
         IN TDS 42 3 1 333444

   17_42 IN TDS 17 5 1 111222
         IN TDS 42 3 1 333444

   A resolver who only knows about Key 17 queries for 17 and will now
   start getting 2 TDS records and will see that this does not match
   what is has configured, and so will add the 42 DS record to its
   configured list of acceptable keys (now it has 17 and 42).

   On its next scheduled check it will lookup 17_42 and see that it is
   "in sync" and will go back to sleep.  The trust anchor maintainer



Kumari                  Expires December 28, 2015               [Page 6]

Internet-Draft    draft-wkumari-dnsop-trust-management         June 2015


   will observe resolvers change from quering for 17 to querying for
   17_42.  Hopefully everyone will end up querying for 17_42, but the
   maintainer can observe who is still asking for 17 and trobleshoot
   with them to see why they have not updated yet.  At some point
   (99.9%?), the maintainer will decide enough people have moved and can
   now start using the new key, by adding it to the DNSKEY set (if they
   are really brave / concerned about MTU they could just start using it
   instead of the old key).

   The maintainer would now like to stop using the old key.  They now
   publish:

   17_42 IN TDS 42 3 1 333444

   42 IN TDS 42 3 1 333444

   Resolvers will query for 17_42 and only receive the Key 42 record.
   They will then remove the Key 17 record from thier config, leaving
   only Key 42.  They will then start querying just for 42, and see that
   they are now in symnc.

   Remember: The DS records in the TDS RRSet define the entire set that
   the trust anchor maintainer would like resolver operator to use for
   that trust point.

Author's Address

   Warren Kumari
   Google
   1600 Amphitheatre Parkway
   Mountain View, CA  94043
   US

   Email: warren@kumari.net

















Kumari                  Expires December 28, 2015               [Page 7]
```
