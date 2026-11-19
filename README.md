# WhatsApp Web/Desktop — Oversized Contact List Processing and Account Termination

> [!WARNING]
> This issue was reported to Meta. The affected behavior has been reported as fixed in newer versions. Updating WhatsApp Web/Desktop is recommended.
>
> Older versions may remain affected.
>
> All testing was conducted exclusively using researcher-controlled accounts.

## Overview

This research looks at an unexpected account-level termination tied to how WhatsApp Web/Desktop processes oversized contact-list messages and their serialized vCard data.

During controlled testing, a specially constructed contact-list message coincided with the recipient's account getting banned, along with the disconnection of their active Web/Desktop sessions.

The exact internal mechanism behind the ban hasn't been confirmed.

## Affected Platforms

- WhatsApp Web
- WhatsApp Desktop

The investigation focused on the Web/Desktop clients. Initial testing involved an iOS-linked account, but iOS shouldn't be assumed to be the only affected platform.

## Research Environment

Testing was done using researcher-controlled accounts and dedicated test sessions.

Runtime behavior was examined through Objective-C instrumentation and Logos-based method inspection on the iOS-linked mobile client, while the Web/Desktop sessions were watched for the resulting account state.

- **Clients tested:** WhatsApp Desktop (primary), WhatsApp Web (secondary), both linked to an iOS mobile account
- **Host OS:** Windows
- **Instrumentation:** Objective-C / Logos, used for runtime method inspection on the linked iOS client
- **Accounts:** researcher-controlled only, one-to-one conversations

## Runtime Analysis

### 1. `WAMessageContentContactList`

Handles contact-list message content.

**Relevant method:** `contacts` — returns the contact entries bundled into a contact-list message. Inspected to see how WhatsApp exposes the contact collection at runtime.

### 2. `WAMessageContentContact`

Handles individual contact entries.

**Relevant method:** `serializedVCard` — returns a given contact's vCard representation, with the usual fields:

- `BEGIN:VCARD`
- `VERSION`
- `N`
- `FN`
- `TEL`
- `waid`
- `END:VCARD`

### 3. Message Processing

We looked at how contact-list serialization relates to what happens next in message processing. Unusually large contact collections kept showing up alongside unexpected account bans, but the exact mechanism behind that is still unconfirmed.

## Instrumentation Code

The hooks below are passive inspection only, covering the two methods above. Each one calls through to the original implementation (`%orig`) and returns the result unchanged — nothing gets modified, dropped, or re-injected into the message pipeline. They just make the values WhatsApp was already computing (the contact array, each vCard string) visible at runtime, so they could be logged and lined up with the ban event.

```objc
#import <Foundation/Foundation.h>

// WAMessageContentContactList represents a contact-list message's payload.
%hook WAMessageContentContactList

// `contacts` returns the array of WAMessageContentContact entries in
// this message. We just pass the original value through — this is
// here for a breakpoint/log point, nothing else.
- (id)contacts {
    id original = %orig;
    return original;
}

%end

// WAMessageContentContact is a single entry inside that array.
%hook WAMessageContentContact

// `serializedVCard` returns the contact already encoded as a vCard
// string (BEGIN:VCARD ... END:VCARD). Same deal: observe, don't touch.
- (id)serializedVCard {
    id original = %orig;
    return original;
}

%end
```

This confirmed the contact list and vCards are built and handed off to the messaging pipeline exactly as the app normally does it — the hooking itself isn't what's causing the odd behavior, since nothing sent or received is altered.

> [!NOTE]
> The specific contact count, vCard size, and field content that reliably triggers the ban aren't included here on purpose. Older, unpatched versions of WhatsApp Web/Desktop are still affected, and publishing those exact parameters would basically be handing out a working DoS against arbitrary accounts rather than disclosing responsibly. That detail, along with logs tying specific payload sizes to the ban event, was shared privately with Meta.

## Observed Behavior

During controlled testing:

1. A contact-list message is sent.
2. The outgoing message sits in an `Awaiting` state.
3. The receiving Web/Desktop client processes it.
4. The recipient's account gets banned.
5. Linked desktop sessions disconnect.
6. Reconnecting can trigger the restriction again.

No interaction from the recipient was needed at any point.

These are just the observations from testing — they don't tell us the exact internal cause of the ban.

## Security Impact

Since no recipient interaction is required, this is more than a typical client-side parsing bug:

- Account bans hit the **recipient**, just from receiving and processing a message
- Linked WhatsApp Web/Desktop sessions get disconnected
- Access to the account is lost, and the restriction can reoccur on reconnect

It sits at the overlap between input validation (how oversized contact-list/vCard data gets parsed) and account-integrity enforcement (how that turns into a ban) — both worth a look from the vendor side.

## Evidence and Validation

The work involved runtime inspection of contact-list and vCard objects, watching message delivery states, and monitoring linked sessions through to the ban. All of it on researcher-controlled accounts. Additional evidence, including the payload parameters left out above, was shared with Meta privately for verification.

## Technical Limitations

Still unconfirmed:

- The exact internal mechanism and component behind the ban
- Whether validation happens before or after message processing
- Whether platforms other than Web/Desktop are affected
- The exact range of affected versions

## Disclosure Status

Reported to Meta; reported as fixed in newer WhatsApp Web/Desktop versions. Older versions may remain affected. Exact affected/fixed versions haven't been independently verified.

[Read the PoC Technical Analysis](<Poc Technical Analysis.md>)

## Credits

- **FbiUWu** — research and discovery
- **darkzin** — research and discovery ([github.com/darkzin-dev](https://github.com/darkzin-dev))
