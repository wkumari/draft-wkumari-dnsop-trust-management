**Important:** Read CONTRIBUTING.md before submitting feedback or contributing
```




template                                                       W. Kumari
Internet-Draft                                                    Google
Intended status: Informational                        September 25, 2015
Expires: March 28, 2016


           Signalling of DNS Security (DNSSEC) Trust Anchors
                draft-wkumari-dnsop-trust-management-01

Abstract

   [ Editor note: This originally included a mechanism to actually roll
   the keys (like 5011 does), but feedback from the Prague meeting
   indicated a strong preference for signalling only. ]

   This document describes a simple method for validating recursive
   resolvers to signal their configured list of DNSSEC trust anchors.
   This mechanism allows the trust anchor maintainer to monitor the
   progress of the migration to a new trust anchor, and so predict the
   effect before decommissioning the existing trust anchor.

   It is primarily aimed at the root DNSSEC trust anchor, but should be
   applicable to trust anchors elsewhere in the DNS as well.

   [ Ed note - informal summary: One of the big issues with rolling the
   root key is that it is unclear who all is using RFC5011, who all has
   successfully fetched and installed the new key, and, most
   importantly, who all will die when the old key is revoked.  By having
   resolvers query for a special QNAME, comprised of the list of TAs
   that it knows about, we effectively signal "up stream" to the TA.  By
   querying for this name, the recursive exposes its list of TAs to the
   auth server.  This allows the TA maintainer to predict how many, and
   who all will break.]

   [ Ed note: Text inside square brackets ([]) is additional background
   information, answers to frequently asked questions, general musings,
   etc.  They will be removed before publication.]

   [ This document is being collaborated on in Github at:
   https://github.com/wkumari/draft-wkumari-dnsop-trust-management.  The
   most recent version of the document, open issues, etc should all be
   available here.  The authors (gratefully) accept pull requests ]

Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.




Kumari                   Expires March 28, 2016                 [Page 1]

Internet-Draft    draft-wkumari-dnsop-trust-management    September 2015


   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF).  Note that other groups may also distribute
   working documents as Internet-Drafts.  The list of current Internet-
   Drafts is at http://datatracker.ietf.org/drafts/current/.

   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any
   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   This Internet-Draft will expire on March 28, 2016.

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

   1.  Introduction  . . . . . . . . . . . . . . . . . . . . . . . .   3
     1.1.  Requirements notation . . . . . . . . . . . . . . . . . .   3
   2.  TDS Record Format . . . . . . . . . . . . . . . . . . . . . .   3
     2.1.  TDS Owner Name  . . . . . . . . . . . . . . . . . . . . .   3
   3.  TDS Record Processing . . . . . . . . . . . . . . . . . . . .   4
   4.  IANA Considerations . . . . . . . . . . . . . . . . . . . . .   5
   5.  Security Considerations . . . . . . . . . . . . . . . . . . .   5
   6.  Contributors  . . . . . . . . . . . . . . . . . . . . . . . .   5
   7.  Acknowledgements  . . . . . . . . . . . . . . . . . . . . . .   5
   8.  References  . . . . . . . . . . . . . . . . . . . . . . . . .   5
     8.1.  Normative References  . . . . . . . . . . . . . . . . . .   5
     8.2.  Informative References  . . . . . . . . . . . . . . . . .   6
   Appendix A.  Changes / Author Notes.  . . . . . . . . . . . . . .   6
   Author's Address  . . . . . . . . . . . . . . . . . . . . . . . .   6








Kumari                   Expires March 28, 2016                 [Page 2]

Internet-Draft    draft-wkumari-dnsop-trust-management    September 2015


1.  Introduction

   When a DNSSEC aware resolver performs validation, it requires a trust
   anchor to validate the DNSSEC chain.  An example of a trust anchor is
   the so called DNSSEC "root key".  For a variety of reasons this trust
   anchor may need to be replaced or "rolled", to a new key (potentially
   with a different algorithm, different key length, etc.).

   [RFC5011] provides a secure mechanism to do this, but operational
   experience has demonstrated a need for some additional functionality
   that was not foreseen.

   During the recent effort to roll the IANA DNSSEC "root key", it has
   become clear that, in order to predict (and minimize) outages caused
   by rolling the key, one needs to know who does not have the new key.

   This document defines a new record type, Trust Digest Signal (TDS),
   which a machanism for validating resolvers to signal their configured
   trust anchors, and some practices for using it.  Readers of this
   document are expected to be familiar with the contents of [RFC7344]
   and [RFC5011].

1.1.  Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

2.  TDS Record Format

   The TDS record does not really have a format, as it does not really
   exist.  Instead it is simply a type that can be queried for, the
   response is meaningless.

   IANA has allocated RR code TBD for the TDS resource record via Expert
   Review [DNS-TRANSPORT].  No special processing is performed by
   authoritative servers or by resolvers, when serving or resolving.

2.1.  TDS Owner Name

   The purpose of the mechanism described in this document is to allow
   the trust anchor maintainer to determine how widely deployed a given
   trust anchor is, and who is using an outdated trust anchor.  This
   information is signalled from the validating resolver to the
   authoritative servers serving the zone in which the Trust Anchor
   lives.





Kumari                   Expires March 28, 2016                 [Page 3]

Internet-Draft    draft-wkumari-dnsop-trust-management    September 2015


   This information is available from looking at queries to DNS servers
   serving the DNSKEY record for the zone; each validating resolver
   using this mechanism will periodically query the zone for a name
   encoding the list of trust anchors it is using for that zone.

   The owner name is computed as follows:

   1.  Take the Key Tags of all of the DS records corresponding to the
       TA(s) that the resolver knows / is using.

   2.  Sort this list in numerically ascending order

   3.  Prepend an underscore ('_') to each Key Tag

   4.  Concatenate the list, with each key tag being a label.

   As an example, if the resolver has a single Trust Anchor with a Key
   Tag of 4217, it would generate an owner name of _4217.  If it has two
   Trust Anchors, with Key Tags 1985 and 1776 it would generate an owner
   name of _1776._1985.

   NOTE: The generation of the TDS Name means that Key Tags MUST be
   unique, at least within "recent" history.  If (e.g during a Key
   Ceremony) a new DNSKEY is generated whose derived Key Tag collides
   with an exiting one (statistically unlikely, but not impossible) this
   DNSKEY MUST NOT be used, and a new DNSKEY MUST be generated. [ Ed
   note: This is to prevent two successive keys having the same keytag
   (e.g: 123), and then seeing "_123." - which 123 key was that?!
   RFC4034 Appendix B admonition: "Implementations MUST NOT assume that
   the key tag uniquely identifies a DNSKEY RR", but this appears to be
   targeted ad validating resolver implmentations.]

3.  TDS Record Processing

   When a compliant recursive resolver performs the "Active Refresh"
   query at port of its RFC5011 ([RFC5011] Section 2.3)) processing it
   will also send a TDS query for the TDS Owner Name.

   It will receive back either an error (e.g NoError / NoData), or a
   (nonsencial) answer.  The entire purpose of this query is to signal
   the list of trust anchors that the recursive reolver knows about to
   the nameservers that serve the zone containing the TA.  This means
   that the response to the query contains no useful information and
   MUST be ignored.







Kumari                   Expires March 28, 2016                 [Page 4]

Internet-Draft    draft-wkumari-dnsop-trust-management    September 2015


4.  IANA Considerations

   [ Ed note: This is largely a place holder.  The real IANA
   considerations section will require updating things like the DPS,
   etc.  ]

   The generation of the TDS Name means that Key Tags MUST be unique, at
   least within an interval.  If, during a Key Ceremony, a new DNSKEY is
   generated whose derived Key Tag collides with an exiting one
   (statistically unlikely, but not impossible) this DNSKEY MUST NOT be
   used, and a new DNSKEY MUST be generated.

   There will need to be some text added to the DNSSEC Ceremony to
   handle this.

5.  Security Considerations

   [ Ed note: a placeholder as well ]

   This mechanism causes a recursove resolver to disclose the list of
   Trust Anchors that it knows about to the authorative servers serving
   the zone containing the TA (or attackers able to monitor the path
   between these devices).  It is conceviable that an attacker may be
   able to use this to determine that a resolver trusts an outdated /
   revoked trust anchor and perform a MitM attack This would also
   require the attacker to have factored the private key.  This seems
   farfetched....

6.  Contributors

   A number of people contributed significantly to this document,
   including Joe Abley, Paul Wouters, Paul Hoffman.  Wes Hardaker and
   David Conrad.

7.  Acknowledgements

8.  References

8.1.  Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/
              RFC2119, March 1997,
              <http://www.rfc-editor.org/info/rfc2119>.







Kumari                   Expires March 28, 2016                 [Page 5]

Internet-Draft    draft-wkumari-dnsop-trust-management    September 2015


   [RFC4034]  Arends, R., Austein, R., Larson, M., Massey, D., and S.
              Rose, "Resource Records for the DNS Security Extensions",
              RFC 4034, DOI 10.17487/RFC4034, March 2005,
              <http://www.rfc-editor.org/info/rfc4034>.

   [RFC5011]  StJohns, M., "Automated Updates of DNS Security (DNSSEC)
              Trust Anchors", STD 74, RFC 5011, DOI 10.17487/RFC5011,
              September 2007, <http://www.rfc-editor.org/info/rfc5011>.

   [RFC7344]  Kumari, W., Gudmundsson, O., and G. Barwood, "Automating
              DNSSEC Delegation Trust Maintenance", RFC 7344, DOI
              10.17487/RFC7344, September 2014,
              <http://www.rfc-editor.org/info/rfc7344>.

8.2.  Informative References

   [I-D.ietf-sidr-iana-objects]
              Manderson, T., Vegoda, L., and S. Kent, "RPKI Objects
              issued by IANA", draft-ietf-sidr-iana-objects-03 (work in
              progress), May 2011.

Appendix A.  Changes / Author Notes.

   [RFC Editor: Please remove this section before publication ]

   From -00.1 to -00 (published):

      Integrated comments and feedback from DRC and Paul Hoffman.

      Use _ as a prefix to make clear it is meta-type (drc)

   From -00.0 to -00.1

   o  Initial draft, written in an airport lounge.

Author's Address

   Warren Kumari
   Google
   1600 Amphitheatre Parkway
   Mountain View, CA  94043
   US

   Email: warren@kumari.net







Kumari                   Expires March 28, 2016                 [Page 6]
```
