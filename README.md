# next-mail

Next-Mail specifications:

Mail format: JSON

Structure:
```
{
  "from": "john-doe@example.com",
  "from-fingerprint": "$2y$12$sgofxSYqPzFN78yFneHU6OMY3ydUMtXjG6fvSRkfCNJNudlkzOWQq",
  "to": "sandra-doe@example.com",
  "cc": null,
  "bcc": null,
  "message-id": "9e279631-0055-44be-b297-00d0ed6aae46",
  "message-id-version": "uuid-v4",
  "sent-time": "2020-12-22T00:04:12Z",
  "subject": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
  "body":"RHVpcyBwdXJ1cyBsYWN1cywgc2NlbGVyaXNxdWUgaWQgdGluY2lkdW50IHNlZCwgZGlnbmlzc2ltIGlkIGFyY3UuIERvbmVjIHV0IGxvYm9ydGlzIGR1aSwgdml0YWUgdWxsYW1jb3JwZXIgbGlndWxhLiBEdWlzIHBlbGxlbnRlc3F1ZSwgbnVuYyBpbiBjb21tb2RvIGltcGVyZGlldCwgbWkgbGliZXJvIG1hdHRpcyB0dXJwaXMsIHNpdCBhbWV0IHZlbmVuYXRpcyBkb2xvciBudW5jIGN1cnN1cyB0dXJwaXMuIE51bGxhbSBub24gbnVsbGEgdGVtcHVzLCBwbGFjZXJhdCB1cm5hIHF1aXMsIHZlaGljdWxhIG5pc2kuIE1hZWNlbmFzIGEgZGlnbmlzc2ltIGlwc3VtLCBuZWMgdHJpc3RpcXVlIG5pc2kuIER1aXMgYXVjdG9yIGdyYXZpZGEgbmlzbCwgdml0YWUgY3Vyc3VzIHVybmEgY29udmFsbGlzIG5vbi4gUGhhc2VsbHVzIG5pc2wgbGFjdXMsIG1vbGxpcyBtYWxlc3VhZGEgbWFzc2EgaWQsIHNvZGFsZXMgbGFjaW5pYSBvZGlvLiBBZW5lYW4gaWQgYXVndWUgYXQgbWFzc2EgZmV1Z2lhdCBydXRydW0gdmVsIGFjIGR1aS4gTnVsbGFtIGxhb3JlZXQgZGlnbmlzc2ltIHRpbmNpZHVudC4gRG9uZWMgZXQgZ3JhdmlkYSB0ZWxsdXMsIHV0IGNvbW1vZG8gc2FwaWVuLiBQcmFlc2VudCB2aXRhZSBsaWJlcm8gb3JjaS4gSW50ZWdlciB1dCBsYWN1cyB1dCBsZWN0dXMgZnJpbmdpbGxhIHZlbmVuYXRpcy4gSW4gdmFyaXVzIGxhY3VzIHV0IGxlY3R1cyBhbGlxdWFtLCBmZXVnaWF0IGZyaW5naWxsYSBuaXNsIHVsbGFtY29ycGVyLiBTZWQgbW9sZXN0aWUgcHVydXMgZW5pbSwgdml2ZXJyYSB0aW5jaWR1bnQgbGliZXJvIGVmZmljaXR1ciBxdWlzLiBOdW5jIHZhcml1cyBsZWN0dXMgZXUgcHVydXMgZGlnbmlzc2ltLCBpbnRlcmR1bSBjb252YWxsaXMgYXVndWUgcHJldGl1bS4NCg0KUGVsbGVudGVzcXVlIGZhY2lsaXNpcyB2ZWwgbWV0dXMgaWQgbGFjaW5pYS4gTWF1cmlzIGVnZXQgZmF1Y2lidXMgdG9ydG9yLiBOdWxsYSBwaGFyZXRyYSBlc3QgZWdldCBzZW1wZXIgYWNjdW1zYW4uIFZlc3RpYnVsdW0gc2l0IGFtZXQgYXVndWUgbmlzaS4gQWVuZWFuIG5pc2kgdGVsbHVzLCBmYWNpbGlzaXMgdXQgc2FnaXR0aXMgdmVsLCBmZXJtZW50dW0gYWMgYXJjdS4gTnVsbGEgYmxhbmRpdCBxdWFtIHV0IG9kaW8gdmVzdGlidWx1bSB0cmlzdGlxdWUuIERvbmVjIGVnZXQgbWkgbWF1cmlzLiBJbiBzZWQgdm9sdXRwYXQgZW5pbS4gQ3JhcyB0ZW1wb3IgbWF0dGlzIG9yY2ksIGlkIHRlbXBvciBleCBsdWN0dXMgYS4gRnVzY2UgYXQgbWFzc2EgbmVjIGFyY3UgbGFvcmVldCBzb2RhbGVzIGV0IGV0IGVuaW0uIFN1c3BlbmRpc3NlIGxlY3R1cyBlbmltLCBsYWNpbmlhIGEgbGlndWxhIGF0LCBibGFuZGl0IGNvbnZhbGxpcyBhbnRlLiBNYWVjZW5hcyBydXRydW0sIGVzdCB1dCB0ZW1wb3IgdmVzdGlidWx1bSwgYXVndWUgZW5pbSBncmF2aWRhIG1hZ25hLCBldCBtYWxlc3VhZGEgbWV0dXMgbmlzbCBncmF2aWRhIGRpYW0uIFByYWVzZW50IGp1c3RvIG5pYmgsIHZlaGljdWxhIGEgZmVybWVudHVtIGVnZXQsIHBvcnRhIGV0IG1pLiBBbGlxdWFtIHNhcGllbiBuaXNpLCBwb3N1ZXJlIGF0IGVnZXN0YXMgZXQsIGZhY2lsaXNpcyBoZW5kcmVyaXQgbWV0dXMuDQoNCg==",
  "attachments": [
    {
      "file-name": "Monthly report January 2021.pdf",
      "mime-type": "application/pdf",
      "data": "iVBORw0KGgoAAAANSUhEUgAAA4QAAAMdCAYAAADDJ7iwAAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAB3RJTUUH5AoFFic1qLKyIwAAIABJREFUeNrsvduTJdd55ffbec8896rq6uoLGt0NgKSGkiySEw6GzAe9jEejGD067Aj/M3qcZ9sxM2GPHzQOeSwrwpJt2RpQ1IWSOOSQFCASFEEBTaKbfauu27nkyXvu3H7Is3dnNUFKIjAyCe4f4kRVnzp1TmZWNbpWre9bSyilFBaLxWL5mUMphRDiB+4bMvz4i48f/lkpZf784mO6rkMIgeM45nGO4wDQdR1KKVzXBUBKSVmW1HWN67o4jsNmsyEIAnzfp65rPM+jrmuUUiRJguM4ZFlG0zREUUQURQghzOPKskRKie/7BEFA13XUdY3v++b1hRC4rmte03EcPM8z9w+PVZ/H8Bq933V88T6LxW"
    }
  ]
}
```

## Sender identification

TODO

## 

## Dates format

ISO 8601 (UTC only)

## Body text format

Body is always sent in Base 64 enconded format. No HTML or JavaScript allowed.

### Rich text markup

#### Characters

All email body characters are represented in 3 bytes as follows:

[11110000100111111001100010000001]   [00000000]
0                                15  16      23

bits[0:15]: UTF-8 character (in this example: 😁)
bits[16]: not used
bits[17]: not used
bits[18]: sub-index
bits[19]: super-index
bits[20]: striked text
bits[21]: underlined
bits[22]: italics
bits[23]: bold

To extract the text without format we can drop the third byte or apply a binary AND operation with mask 11111111 11111111 00000000.

#### Font size

[<font-size>]<text>[/<font-size>]

Example:
[14]4c6f72656d20697073756d20646f6c6f722073697420616d6574[/14]

The text above would be shown in 14 points size font.

#### Font color

[rgb:<hex-number>]<text>[/rgb]

Example:
[rgb:FFFFFF]4c6f72656d20697073756d20646f6c6f722073697420616d6574[/rgb]

#### Images

External images:

[img]https://avatars0.githubusercontent.com/u/1157916?s=96&v=4[/img]

Inline images:

[img]image/png;iVBORw0KGgoAAAANSUhEUgAAAA4AAAAQCAYAAAAmlE46AAAAAXNSR0IArs4c6QAAAIRlWElmTU0AKgAAAAgABQESAAMAAAABAAEAAAEaAAUAAAABAAAASgEbAAUAAAABAAAAUgEoAAMAAAABAAIAAIdpAAQAAAABAAAAWgAAAAAAAABgAAAAAQAAAGAAAAABAAOgAQADAAAAAQABAACgAgAEAAAAAQAAAA6gAwAEAAAAAQAAABAAAAAAfkeR3QAAAAlwSFlzAAAOxAAADsQBlSsOGwAAAVlpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IlhNUCBDb3JlIDUuNC4wIj4KICAgPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4KICAgICAgPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIKICAgICAgICAgICAgeG1sbnM6dGlmZj0iaHR0cDovL25zLmFkb2JlLmNvbS90aWZmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICA8L3JkZjpEZXNjcmlwdGlvbj4KICAgPC9yZGY6UkRGPgo8L3g6eG1wbWV0YT4KTMInWQAAAB5JREFUKBVj/A8EDGQAJjL0gLWMasQTcqOBM8IDBwBlLAQctY7BrAAAAABJRU5ErkJggg==[/img]

#### Tables

TODO

## Attachments fomat

The attachments data value is also a Base64 encoded string.
