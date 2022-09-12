# Autoplay Policy Detection - Security and Privacy Questionnaire

This document answers the [W3C Security and Privacy
Questionnaire](https://w3ctag.github.io/security-questionnaire/) for the
Autoplay Policy Detection specification.

Last Update: 2022-09-12

**What information might this feature expose to Web sites or other parties, and
for what purposes is that exposure necessary?**

This API exposes information to allow websites detect if autoplaying media is
allowed, which help them make actions, such as selecting alternate content or
improving the user experience while media is not allowed to autoplay.

Example query:
Is this media element allowed to autoplay?

Example answer:
The queried media element is not allowed to autoplay.

If the user agent does not allow any autoplay media, then websites could stop
loading media resources and related tasks to save the bandwidth and CPU usage
for users.

**Do features in your specification expose the minimum amount of information
necessary to enable their intended uses?**

Yes. The API will return different results, such as `allowed`, `allowed-muted`
and `disallowed`, to answer websites' question.

**How do the features in your specification deal with personal information,
personally-identifiable information (PII), or information derived from them?**

This specification does not deal with PII.

**How do the features in your specification deal with sensitive information?**

This specification does not deal with sensitive information.

**Do the features in your specification introduce new state for an origin that
persists across browsing sessions?**

No.

**Do the features in your specification expose information about the underlying
platform to origins?**

No. The information about whether autoplay is not allowed is not platform
specific. The result doesn't describe anything which can be used to deduce the
underlying platform.

**Do features in this specification allow an origin access to sensors on a
user’s device?**

No.

**What data do the features in this specification expose to an origin? Please
also document what data is identical to data exposed by other features, in the
same or different contexts.**

3 enums, "allowed", "allowed-muted" and "disallowed", which are used to answer
the question for knowing the status for autoplay.

There is no other API can directly answer the status of whether autoplay is
allowed. However, for media element, there is a API could answer the question
indirectly. But for the audio context, there is no way to know the status.

Eg. `HTMLMediaElement.play()`, will return a promise. If autoplay is not
allowed, the play promise will be rejected, and the element will receive an
unsupported error.

**Do features in this specification enable new script execution/loading
mechanisms?**

No.

**Do features in this specification allow an origin to access other devices?**

No.

**Do features in this specification allow an origin some measure of control over
a user agent’s native UI?**

No.

**What temporary identifiers do the features in this specification create or
expose to the web?**

No.

**How does this specification distinguish between behavior in first-party and
third-party contexts?**

It does not distinguish.

**How do the features in this specification work in the context of a browser’s
Private Browsing or Incognito mode?**

This specification does not treat Private Browsing and Incognito mode in a
special way. They should all work the same as normal browsing mode.

Unless the user agent implements something specially which would return
different answers for the same origin under the same situation.

**Does this specification have both "Security Considerations" and "Privacy
Considerations" sections?**

Yes, this specification has a [Security and Privacy Considerations](https://w3c.github.io/autoplay/#security-and-privacy)
section already.

**Do features in your specification enable downgrading default security
characteristics?**

No.
