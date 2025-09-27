# ADIF 4 - A Proposal

### TL;DR

The problem:

* ADIF is "text based", but does not specify the encoding to use in this "text", since this didn't matter for 7-bit ASCII.
* ADIF uses "counts" to determine the length of the data in each field, but does not specify whether the counts should be in bytes or characters, since this didn't matter for US-ASCII.
* Users want to use "extended characters" in their logs.
* [Developers have used both](survey-results/README.md) 8-bit encodings and UTF-8 in their apps, and both are equivalent for 7-bit ASCII.
* But once you add "extended characters", the two encodings are not equivalent, and apps can break not only by showing the wrong characters, but by truncating data, including extraneous data, or even dropping fields or entire records.
* ***So, we have, at the very least, three different ways to interpret the standard, based on which encoding you pick: ASCII, Latin1 and UTF8.*** And these are incompatible with each other if you want to consider including "extended characters" in the standard.

The proposal:

* Produce a new major version of the standard, ADIF 4.
* Make all ambiguous points in the standard explicit.
* Define a new "encoding" header so that apps can explicitly state which encoding they are using.
* Declare "US-ASCII" as the default encoding, and require apps to support importing and exporting it in order to be considered "compliant".
* Declare that apps can reject encodings they don't support, and still be considered "compliant", but provide a page at https://fixes.adif.org/ that apps can send their users to, which provides users with tools to fix encoding issues and convert between encodings.
* Suggest that apps should support "UTF8" imports and exports.
* Suggest that apps should support "Latin1" imports, and might even consider this the default encoding when importing pre-ADIF 4 files.


### Why do we need to talk about a new standard?

The [ADIF Standard](https://www.adif.org/316/ADIF_316.htm) was defined from the very beginning as a "text format". This is never said directly in the documents, but the format is clearly intended to be human-readable and editable. It also mentions line breaks, 7-bit ASCII, and the use of ASCII digits for any numeric values.

The standard does explicitly state that `String` fields should be limited to ["ASCII [...] in the range of 32 through 126"](https://www.adif.org/316/ADIF_316.htm#:~:text=an%20ASCII%20character%20whose%20code%20lies%20in%20the%20range%20of%2032%20through%20126%2C%20inclusive). This is also known officially as "US-ASCII".

However, in the real world, most apps alllowed users to enter "extended characters", such as accented letters like the "ñ" in "Muñoz", the "ü" in "Müller" or the "ß" in "Straße".

And many apps include these "extended characters" in the ADIF files they export. This is clearly not compliant with the standard, but definitely prevalent (see [survey results](survey-results/README.md)).

There is a problem with this: once you get past the basic english alphabet and into "extended characters", there is no such thing as a "text file" anymore. The text has to be characterized by the encoding used to convert the characters to bytes.

> There are two popular encodings: "Windows-1252", which is a superset of the formal standard "ISO-8859-1", both of which encode the "Latin-1" alphabet; and "UTF-8" which encodes all characters in the Unicode standard.
>
> There are other useful encodings, including other 8-bit code page encodings in the ISO-8859 family, covering alphabets like greek, cyrillic and more, or Shift-JIS for Japanese characters.
>
> But for simplicity, we will keep the discussion around the main two, and use the terms "Latin1" and "UTF8" to refer to each one of them.

Both "Latin1" and "UTF8" are supersets of "US-ASCII" (plain english alphabet, numbers, punctuation, etc.) so either is perfectly compatible with the ADIF standard, strictly speaking.

But the two encodings are not compatible once you consider "extended characters". More importantly, **"Latin1" assumes that each character is encoded as a single byte**, while **"UTF8" uses multi-byte sequences** to represent characters outside of "US-ASCII".

Now, the ADIF standard uses "counts" to determine the length of the data in each field. **But the standard never defined whether the counts should be in bytes or characters**. For "US-ASCII", the two are equivalent, but for "extended characters", they are not.

And developers have implemented their apps using logic that assumes either encoding, [with major apps on either side of the fence](survey-results/README.md). Most likely this was the result of just using the "text primitives" available in their programming language or environment of choice. Old Win32 apps would have used the Windows API, which uses "Windows-1252" by default, while new .NET apps would have used the .NET Framework, which uses "UTF-8" by default. Newer languages like Java, Python and Javascript all use Unicode internally too.

So we have a "Schrödinger's Standard", that can be either US-ASCII, Latin1 or UTF8 depending on who you ask, and you cannot really tell which it is until you bring extended characters into the mix.

This has caused extended discussions in the ADIF community, and has led to a lot of confusion and frustration. With different parties arguing that their interpretation of "what text is" is the correct one.

This proposal starts from the premise that all three encodings should be considered valid, but that not everybody has to be required to support all three.


### What is the proposal?

First, we should make this a new major version of the standard, ADIF 4. In order to make it easier to communicate with users, and between developers, that from this clearly marked point on we are talking about a standard that supports multiple encodings and thus requires specificity about which encoding is being used.

Second, we should make all ambiguous points in the standard explicit. There's a section bellow that lists the details. This means declaring that the standard is "text based", with the text using the encoding specified in an "encoding" header, and that counts should be in "characters" as defined by the encoding.

> One point of extensive discussion has been that we should allow developers to use "bytes" as the unit of measurement for counts, and use this for UTF-8 encoded files.
>
> Our counterargument is that if you mix UTF-8 with byte encodings, you end up with something that no longer is "text based" can be considered a binary format, because implementing this requires apps to open files as binary data in order to read bytes and not characters
>
> Another point that can be made is that this is not UTF-8 with byte encodings, but rather Latin-1 text with character counts, where the strings include characters that happen to match UTF-8 sequences. Something known as ["mojibake"](https://en.wikipedia.org/wiki/Mojibake).

Third, to ensure backwards compatibility, we should declare that "US-ASCII" is the default encoding, and require apps to support importing and exporting it in order to be considered "compliant". This is because this is what the current standard defines; and the most prevalent scenario, by far, in the real world is that regardless of which encoding a developer used, the data is still limited to US-ASCII.

Fourth, we should suggest that apps should support "UTF8" imports and exports. This broadens "extended character" support to the entire Unicode standard, and not just a small set like Latin-1 or Greek. New apps, and updated apps, should aim to support UTF-8 as their main encoding for interoperability purposes. For reference, UTF-8 is used by [98.8% of the world's web sites](https://w3techs.com/technologies/cross/character_encoding/ranking).

Fifth, while it is desirable, it is not reasonable to demand all developers to update their apps to support Unicode and UTF-8. Therefore, we should allow apps to reject encodings they don't support, and still be considered "compliant".

At the same time, we don't want to leave users stranded, so the standard should provide a page at https://fixes.adif.org/, that apps can refer their users to, which provides information and tools to help users address encoding issues and incompatible apps.

Sixth, we should suggest that apps should support "Latin1" imports and exports, and might even consider this the default encoding when importing pre-ADIF 4 files. This is because many existing apps use Latin1 as their internal encoding, it's mostly backwards-compatible with apps that can deal with US-ASCII, and it's the most prevalent encoding currently used for "extended characters" in the real world of amateur radio. So while this is not fully compliant with the existing standard, I believe it is a reasonable compromise to minimize impact on end users.

It would not be unreasonable to actually declare "Latin1" as the official default encoding for pre-ADIF 4 as long as we're ok with the implication that such a change would make the new standard not be strictly backwards-compatible. This would increase the likelihood, but not ensure, of a file exported from an existing app being interpreted correctly by an encoding-aware application. If we prefer to keep the standard strictly backwards-compatible, we can just leave it as a suggestion and keep US-ASCII as the default encoding.

### What does this mean for existing apps?

Most apps can remain unchanged and be able to interoperate with apps updated to ADIF 4 in all cases that were valid when the app was written.

These apps can also remain unchanged and still be able to interoperate with _most_ cases that are _prevalent_ in real world usage.

These apps can also remain unchanged and still be able to _mostly_ interoperate, with minimal data loss on non-mandatory fields, when importing from ADIF 4 files with encodings they might not support.

Most apps can be updated to improve their support for ADIF 4 just by adding an "encoding" header to their output files.

Most apps can improve this support by looking at the "encoding" header in the input file, and if it's not supported, they can offer their users a link to https://fixes.adif.org/.

Most apps can achieve full compliance by offering their users different encoding options, including at the very minimum US-ASCII.

Most apps can improve their support for ADIF 4 by using mechanisms to transcode input files to whatever encoding they support internally. We intend to [provide tools to help with this](https://github.com/ham2k/transadif/), and the standard might provide guidance on this respect.

Most apps can improve their support even further by offering their users multiple encoding options when exporting, ideally including UTF-8 and Latin-1 at the very least.

## Detailed changes to the standard

Still to come.
