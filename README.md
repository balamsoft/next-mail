# next-mail (v0.0.1)

Next-Mail specification version 0.0.1.

## Payload format

### Emails payload format

A JSON UTF-8 string without comments.

```
{
  "version": "next-mail-0.0.1",
  "message-id": "9e279631-0055-44be-b297-00d0ed6aae46",
  "message-id-algorithm": "uuid-v4",

  "from": "john-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "to": ["sandra-doe@example.com", "sales@example.com"],
  "cc": [],
  "bcc": [],
  "sent-time": "2020-12-22T00:04:12Z",

  "subject": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt.",
  "font-family": "example-com-serif",
  "default-font-family": "sans-serif",

  "body":"
    [H1]Monthly Report January - 2021[/H1]
    
    Hi Sandra, I'm sending you the sales report for January.
    You will find the report in the [i]attachments section[/i].
  ",
  "signature": {
    "full-name": "John H. Doe",
    "company-name": "Example Dot Com SA",
    "address-line-1": "77 Massachusetts Ave",
    "address-line-2": null,
    "city": "Cambridge",
    "state": "MA",
    "zip-code": "02139",
    "company-url": "www.example.com",
    "phone-number-1": "202-555-5555",
    "phone-number-1-type": "office",
    "phone-number-2": "202-555-0719",
    "phone-number-2-type": "mobile",
    "country-code": "1"
  }
  "attachments": [
    {
      "file-name": "Monthly report January 2021.pdf",
      "mime-type": "application/pdf",
      "data": "iVBORw0KGgoAAAANSUhEUgAAA4QAAAMdCAYAAADDJ7iwAAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAB3RJTUUH5AoFFic1qLKyIwAAIABJREFUeNrsvduTJdd55ffbec8896rq6uoLGt0NgKSGkiySEw6GzAe9jEejGD067Aj/M3qcZ9sxM2GPHzQOeSwrwpJt2RpQ1IWSOOSQFCASFEEBTaKbfauu27nkyXvu3H7Is3dnNUFKIjAyCe4f4kRVnzp1TmZWNbpWre9bSyilFBaLxWL5mUMphRDiB+4bMvz4i48f/lkpZf784mO6rkMIgeM45nGO4wDQdR1KKVzXBUBKSVmW1HWN67o4jsNmsyEIAnzfp65rPM+jrmuUUiRJguM4ZFlG0zREUUQURQghzOPKskRKie/7BEFA13XUdY3v++b1hRC4rmte03EcPM8z9w+PVZ/H8Bq933V88T6LxW"
    }
  ]
}
```

### Replies payload format

Unlike with traditional emails replies do not include the original message, instead, they reference it:

```
{
  "version": "next-mail-0.0.1",
  "reply-id": "9e279631-0055-44be-b297-00d0ed6aae46",
  "reply-id-algorithm": "uuid-v4",
  "original-message-id": "9e279631-0055-44be-b297-00d0ed6aae46",

  "from": "sandra-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "sent-time": "2020-12-22T00:04:23Z",

  "subject": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",

  "body":"Thanks John, I'll check it.",
  "signature": {
    "full-name": "Sandra Doe",
    "company-name": "Example Dot Com SA",
    "address-line-1": "77 Massachusetts Ave",
    "address-line-2": null,
    "city": "Cambridge",
    "state": "MA",
    "zip-code": "02139",
    "company-url": "www.example.com",
    "phone-number-1": "202-555-5556",
    "phone-number-1-type": "office",
    "country-code": "1"
  }
  "attachments": []
}
```

The attribute original-message-id points to the original email sent.

To identify JSON messages as emails or replies next-mail clients should check the following attributes:

- Emails: `version`, `message-id`, `message-id-algorithm`
- Replies: `version`, `reply-id`, `reply-id-algorithm`, `original-message-id`

Any other combinations should be considered as an invalid message. For example, if a message contains both a `message-id` and a `reply-id` it should be considered invalid. Also, if an email or reply misses any of the fields listed above it should be considered invalid.


## Sender identification

### Verification request

An HTTP request is sent to the original domain including the original sender's address and the `from-fingerprint` and the `message-id` values as follows:

```
{
  "from": "john-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "message-id": "9e279631-0055-44be-b297-00d0ed6aae46"
}
```

Following the example above, the request will be sent to `nmail.example.com/verify` which will do the following:

1. An account exists for the sender.
2. The hash code is valid against the sender's secret code, which is a randomized unique string generated for that user when the account was created.
3. The account did send a message that matches the given `message-id`, and the generated `from-fingerprint` hash code in that event matches as well.

Next-mail servers should allow user's to renew their secret code, if and only if, any previous valid secret code are kept. If previous valid secret codes
are deleted then any previous emails sent from that email account will be rendered as "unverified sender" by mail clients.

Next-mail clients should keep indexes for values: `from`, `from-fingerprint` and `message-id`. Mail clients should actively refuse any emails received with a duplicate `from-fingerprint` or `message-id` values since those values are meant to be unique at all times. Re-sending or forwarding emails should provide new values for those two fields at the moment the message is sent.

### Verification endpoint

The next-mail verification URL will be constructed from the domain in the sender's mail address. The company should publish a subdomain called `nmail` which in turn should provide an endpoint called `verify`.

Custom verification URLs should not be allowed since that could bypass this verification mechanism really easily. For example, an attacker could impersonate a CEO of a company by using his/her email address in the `from` field and then providing a verification URL to a server like `2y6yy9Hglsjza2EBqAADgNq24naGAZo4EepM0Nli.biz/verify` that simply replies successfully to all requests it receives.

### Verification response

The verification response should include all the fields received in the request plus two more fields called `verified` and `verification-date`. Example:

```
{
  "from": "john-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "message-id": "9e279631-0055-44be-b297-00d0ed6aae46",
  "verified": true,
  "verification-date": "2020-12-22T00:04:22Z"
}
```


## Dates format

ISO 8601 - Limited to UTC only, clients should convert the dates to the local date and time according to the user's timezone.

```
"sent-time": "2020-12-22T00:04:12Z"
```


## Rich text markup

For security reasons we should prevent the inclusion of HTML and Javascript code in messages. For these reasons a custom markup language will be used to format text in emails.

### Headers

```
[H1]Lorem ipsum[/H1]
[H2]Lorem ipsum[/H2]
[H3]Lorem ipsum[/H3]
[H4]Lorem ipsum[/H4]
[H5]Lorem ipsum[/H5]
[H6]Lorem ipsum[/H6]
```

### Basic text formating

```
[b]Bold text[/b]
[i]Italics[/i]
[u]Underline[/u]
[s]Striked text[/s]
[super]super index[/super]
[sub]subindex[/sub]
[left]left aligned[/left]
[center]centered text[/center]
[right]right aligned[/right]
[justify]justified text[/justify]
```

### Font size

```
[{font-size}]{text}[/{font-size}]
```

Example:
```
[14]Lorem ipsum dolor sit amet, consectetur adipiscing elit[/14]
```

The text above would be shown in 14 points size font.

### Font family

Fonts cannot be attached or included in emails. Instead, fonts will be resolved by the next-mail client based on the fonts installed in the operating system. The emaill will be rendered with the font specified by `font-family` and when this one doesn't exist it will default to `default-font-family` and, in last instance, to a font specified in the next-mail client settings.

```
"font-family": "example-com-serif",
"default-font-family": "sans-serif"
```

Both attributes are optional, when both `font-family` and `default-font-family` are undefined the client will render the email in the font specified in the next-mail client settings.

### Font color

```
[rgb:{hex-number}]{text}[/rgb]
[rgb:{r},{g},{b}]{text}[/rgb]
```

Examples:
```
[rgb:EBE100]Lorem ipsum dolor sit amet, consectetur adipiscing elit[/rgb]
[rgb:235,255,0]Lorem ipsum dolor sit amet, consectetur adipiscing elit[/rgb]
```

### Images

External images:
For security reasons including external images is not supported.

Inline images:
```
[img]image/png;iVBORw0KGgoAAAANSUhEUgAAAA4AAAAQCAYAAAAmlE46AAAAAXNSR0IArs4c6QAAAIRlWElmTU0AKgAAAAgABQESAAMAAAABAAEAAAEaAAUAAAABAAAASgEbAAUAAAABAAAAUgEoAAMAAAABAAIAAIdpAAQAAAABAAAAWgAAAAAAAABgAAAAAQAAAGAAAAABAAOgAQADAAAAAQABAACgAgAEAAAAAQAAAA6gAwAEAAAAAQAAABAAAAAAfkeR3QAAAAlwSFlzAAAOxAAADsQBlSsOGwAAAVlpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IlhNUCBDb3JlIDUuNC4wIj4KICAgPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4KICAgICAgPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIKICAgICAgICAgICAgeG1sbnM6dGlmZj0iaHR0cDovL25zLmFkb2JlLmNvbS90aWZmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICA8L3JkZjpEZXNjcmlwdGlvbj4KICAgPC9yZGY6UkRGPgo8L3g6eG1wbWV0YT4KTMInWQAAAB5JREFUKBVj/A8EDGQAJjL0gLWMasQTcqOBM8IDBwBlLAQctY7BrAAAAABJRU5ErkJggg==[/img]
```

### Hyperlinks

```
[mailto:{Hyperlink text}]{Email Address}[/mailto]
[a:{Hyperlink text}]{URL}[/a]
```

Examples:
```
[mailto:John Doe, Finance]john-doe@example.com[/mailto]
[a:http://www.example.com/events/2021/March/security-awareness-meeting]Security Awareness Meeting[/a]
```

Next-mail servers and clients should implement a mechanism that verifies URLs and renders them unclickable to all users when blacklisted. Before visiting the link the next-mail client will create a shortened version of it, the short URL will be opened instead of the original one.

Hyperlinks executing `javascript` code will simply not be allowed.

### Tables

TODO

### Escape codes

Next-mail escape sequences:

|Sequence |Description                      |
|---------|---------------------------------|
|\\[      |to escape left square brackets   |
|\\]      |to escape right square brackets  |
|\\"      |to escape double quotes          |
|\\\\     |to escape backslash              |

## Attachments

The attachments data value is also a Base64 encoded string.

```
  ...
  "attachments": [
    {
      "file-name": "Monthly report January 2021.pdf",
      "mime-type": "application/pdf",
      "data": "iVBORw0KGgoAAAANSUhEUgAAA4QAAAMdCAYAAADDJ7iwAAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAB3RJTUUH5AoFFic1qLKyIwAAIABJREFUeNrsvduTJdd55ffbec8896rq6uoLGt0NgKSGkiySEw6GzAe9jEejGD067Aj/M3qcZ9sxM2GPHzQOeSwrwpJt2RpQ1IWSOOSQFCASFEEBTaKbfauu27nkyXvu3H7Is3dnNUFKIjAyCe4f4kRVnzp1TmZWNbpWre9bSyilFBaLxWL5mUMphRDiB+4bMvz4i48f/lkpZf784mO6rkMIgeM45nGO4wDQdR1KKVzXBUBKSVmW1HWN67o4jsNmsyEIAnzfp65rPM+jrmuUUiRJguM4ZFlG0zREUUQURQghzOPKskRKie/7BEFA13XUdY3v++b1hRC4rmte03EcPM8z9w+PVZ/H8Bq933V88T6LxW"
    }
    ...
```