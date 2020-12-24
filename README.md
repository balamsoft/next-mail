# next-mail (v0.0.1)

Next-Mail specification version 0.0.1.

## Payload format

A JSON UTF-8 string without comments.

### Emails payload format

```
{
  "version": "0.0.1",
  "message-id": "9e279631-0055-44be-b297-00d0ed6aae46",
  "message-id-algorithm": "uuid-v4",

  "from": "john-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "to": ["sandra-doe@example.com", "sales@example.com"],
  "cc": [],
  "bcc": [],
  "sent-time": "2020-12-22T00:04:12Z",

  "subject": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt.",

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
    "phone-numbers": [
        {"number": "202-555-5555", "type": "OFFICE", "country-code": 1},
        {"number": "202-555-0719", "type": "MOBILE", "country-code": 1}
    ]
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
  "version": "0.0.1",
  "reply-id": "9e279631-0055-44be-b297-00d0ed6aae46",
  "reply-id-algorithm": "uuid-v4",
  "original-message-id": "9e279631-0055-44be-b297-00d0ed6aae46",

  "from": "sandra-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "sent-time": "2020-12-22T00:04:23Z",

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
    "phone-numbers": [
        {"number": "202-555-5556", "type": "OFFICE", "country-code": 1}
    ]
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

### Verification request

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

### Verification endpoint

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

### Basic text formatting

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
[div]...[/div]
[span]...[/span]
[code]...[/code]
[cite]...[/cite]
[blockquote]...[/blockquote]
[p]...[/p]
[pre]...[/pre]
[ul]
    [li]Milk[/li]
    [li]Coffee[/li]
[/ul]
```

The list is not meant to be extensive; it is only meant to support a subset of HTML tags sufficient for creating good-looking emails.
Tags like `iframe`,`script`,`form`,`input`,`select`,`embed`and `canvas`, among others, won't and shouldn't be allowed in the future. This means no tracking/analytics scripts in the emails you receive.

### Images

External images:
For security reasons external images are not supported.
(TODO: consider allowing external images, then fetching the image files in the back end and translating them into Base64 strings before being sent to the front end)

Inline images:
```
[img]image/png;iVBORw0KGgoAAAANSUhEUgAAAA4AAAAQCAYAAAAmlE46AAAAAXNSR0IArs4c6QAAAIRlWElmTU0AKgAAAAgABQESAAMAAAABAAEAAAEaAAUAAAABAAAASgEbAAUAAAABAAAAUgEoAAMAAAABAAIAAIdpAAQAAAABAAAAWgAAAAAAAABgAAAAAQAAAGAAAAABAAOgAQADAAAAAQABAACgAgAEAAAAAQAAAA6gAwAEAAAAAQAAABAAAAAAfkeR3QAAAAlwSFlzAAAOxAAADsQBlSsOGwAAAVlpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IlhNUCBDb3JlIDUuNC4wIj4KICAgPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4KICAgICAgPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIKICAgICAgICAgICAgeG1sbnM6dGlmZj0iaHR0cDovL25zLmFkb2JlLmNvbS90aWZmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICA8L3JkZjpEZXNjcmlwdGlvbj4KICAgPC9yZGY6UkRGPgo8L3g6eG1wbWV0YT4KTMInWQAAAB5JREFUKBVj/A8EDGQAJjL0gLWMasQTcqOBM8IDBwBlLAQctY7BrAAAAABJRU5ErkJggg==[/img]
```

### Hyperlinks

```
[mailto:{Hyperlink text}]{Email Address}[/mailto]
[url:{Hyperlink text}]{URL}[/url]
```

Examples:
```
[mailto:John Doe, Finance]john-doe@example.com[/mailto]
[url:Security Awareness Meeting]http://www.example.com/events/2021/March/security-awareness-meeting[/url]
```

Next-mail servers and clients should implement a mechanism that verifies URLs and renders them unclickable to all users when blacklisted. Before visiting the link the next-mail client will create a shortened version of it, the short URL will be opened instead of the original one.

Hyperlinks executing `javascript` code will simply not be allowed.

### Tables

```
[table;class:balance-sheet]
  [row;class:banner-row]
    [col]
      [img]image/png; ... [/img]
    [/col]
  [/row]
  [row;class:header-row]
    [col]Number[col]
    [col]Description[col]
    [col]Amount ($USD)[col]
  [/row]
  [row]
    [col]1[/col]
    [col]Net income[/col]
    [col]$14.564M[/col]
  [row]
  ...
[/table]
```

### CSS Formatting

As you could see in the `tables` section you can use class names. The CSS code for a next-mail 
email is defined in the `css` attribute at the root node.

```
{
  ...
  "body": "[div;class=mail-title]Some text[/div]",
  "css": ".mail-title {
    text-align: center;
    font-size: 14px;
    font-weight: bold;
    color: #EE0022;
  }"
  ...
}
```

You can also define classes in other tags seen before. Use a semicolon (;) to separate the class definition from the rest of the tag as follows:

```
[div;class:main-section]...[/div]
[code;class:code-block-java]...[/code]
[mailto:John Doe, Finance;class:mailto-link]john-doe@example.com[/mailto]
[ul;class:numbered-list]...[/ul]
[12;class:simple-text]...[/12]
```

Also you can add styles to all tags of the same type just like in HTML:

```
{
    ...
    "body": "
        [div]This is a div[/div]
        [div]This is another div[/div]
        [span]This is a span[/span]
    ",
    "css": "div {
        display: block;
        font-size: 14pt;
        color: black;
    }
    span {
        display: inline-block;
        font-size: 12pt;
        background-color: black;
        color: white;
    }
    "
}
```

Tags like `url` and `mailto` will become HTML anchors (`<a href="..."/>`).

### Escape codes

Next-mail escape sequences:

|Sequence |Description                      |
|---------|---------------------------------|
|\\[      |to escape left square brackets   |
|\\]      |to escape right square brackets  |
|\\"      |to escape double quotes          |
|\\\\     |to escape backslash              |

## Attachments

Emails and replies can contain file attachments (optional). The attachments' data value is also a Base64 encoded string.

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

## Appointments

Emails can include one appointment (optional).

```
{
  ...
  "appointment": {
    "title": "Sales montly meeting",
    "type": "Event",
    "date": "2020-12-23T17:00:00Z"
    "duration": {
      "minutes": 30
    },
    "all-day": false,
    "timezone": "GMT+5",
    "reminders": [
      {"minutes": -15},
      {"days": -1}
    ],
    "guests": [
      "john-doe@example.com", "sandra-doe@example.com", "sales-team@example.com"
    ],
    "phone-numbers": {
      [
        {"number": "+16172532871", "guest-access-code": "123456"},
        {"number": "+16172532872", "guest-access-code": "123456"}
      ]
    },
    "meeting-url": "meet.google.com/bco-vrja-abc",
    "location": {
      "address-line-1": "77 Massachusetts Ave",
      "address-line-2": null,
      "city": "Cambridge",
      "state": "MA",
      "zip-code": "02139",
      "company-url": "www.example.com",
      "phone-numbers": [
        {"number": "202-555-5645", "type": "OFFICE", "country-code": 1}
      ]
      "meeting-room": "Room 14",
      "gps-coordinates": {
        "latitude": "42.3631788",
        "longitude": "-71.0933875"
      }
    }
  }
}
```

## Tasks

Sometimes you may send an email asking someone to perform a task and want to know the progress.
The current state of the task will be visible to all persons who received that email. The state should be handled and updated by next-mail clients.

Only one task per email allowed (optional).

```
{
  ...
  "task": {
    "checklist": [
      "Create the sales report",
      "Peer review it",
      "Send report to Sandra"
    ],
    "workflow": [
      "1": "TODO",
      "2": "IN-PROGRESS",
      "3": "DONE"
    ],
    "end-states": ["3"],
    "deadline": "2021-02-01T12:15:00Z"
  }
}
```

A description may not be necessary in the task itself, it must be written in the email's body.
Checklist and deadline are optional but `worflow` and `end-states` are mandatory. There could be cases in which more that one state could be considered as a final state. Workflows may be created in the next-mail client and they are copied to the email instead of being written every time per email.

## Tags

Emails may optionally include tags to help sort them or trigger actions setup using third party plugins.

```
{
  ...
  "tags": [
    "sales-department", "reminders", "work"
  ]
}
```

```
{
  ...
  "tags": [
    "echo", "lights", "turn-on", "living room"
  ]
}
```

## Message triggers

The user may be able to setup triggers for emails.

For example, this may be useful for third party apps sending emails that perform some action and must be deleted after they have been used:
"Move email to trash 5 minutes after it has been received"
```
"trigger": {
  "event": {
    "message": "received",
    "from": "john.doe@example.com",
    "includes-tags": ["echo", "google-assistant"]
  },
  "action": {
    "move": "trash"
  },
  "delay": {
    "minutes": 5
  }
}
```

Or to immediately move emails with a task to a folder after they are marked as done.
"Move email to folder called 'Done' after it has been completed"
```
"trigger": {
  "event": {
    "task": ["DONE", "WONT-FIX"]
  },
  "action": {
    "move": "Done"
  },
  "delay": null
}
```

Triggers can only be defined by the recipient and should be setup in the next-mail client application.

[TODO Revise security implications]
