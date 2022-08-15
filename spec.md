---
title: "Embeddable URI references for ELF binaries"
---

*This specification is a work in progress, and has not been
submitted to any standards body for peer review.  Do not use
this specification in any production capacity.*

## Abstract

*This section is non-normative.*

Identifying the provenance of binary software components is
an important topic in the software servicing lifecycle with
numerous use-cases.  [Software Bill of Materials][sbom]
documents provide a reference allowing the identification of
underlying software components present in a binary software
package.  However, linking specific [Executable and Linking Format][elf]
binary objects in a software package to an SBOM document
varies depending on how the software package is built and
distributed.

ELF references provide typed URIs which can be embedded in
ELF binaries as `SHT_NOTE` sections, providing the ability
to have an embedded copy of, or pointer to, any resource
including an SBOM document, allowing for universal
correlation between ELF binaries and their SBOM.

> *TODO*: Improve the above paragraph to explain more about
> why people want this.

## The ELF `.reference` note

The `.reference` note is an `SHT_NOTE` typed section, with
the assigned section name `.reference` featuring an optional
name and required descriptor.

`SHT_NOTE` sections are described on page 2-4 of the [ELF
specification][elf], and the `.reference` note follows
this description.

### The `.reference` name section

The `namesz` and `name` fields are used to describe the
reference type.  The type of the reference is optional
and *may* be omitted.  If the type of the reference is
provided, then it *must* comply with [RFC 6838][rfc6838],
including the [naming requirements in its section 4.2][rfc6838-naming].

The `namesz` field *must* be set to `0` if the `.reference`
note does not have a type, otherwise it *must* be set to
the length of the typed URI in bytes with padding to ensure
4-byte alignment.

Any padding bytes present as part of the `name` field *must*
be set to `NUL` (`0x0`).

### The `.reference` descriptor section

The `descsz` and `desc` fields are used for the reference
URI in the `.reference` note, which *must* be present.

The `descsz` field *must* be set to the length of the
reference URI with padding to ensure 4-byte alignment.

Any padding bytes present as part of the `desc` field *must*
be set to `NUL` (`0x0`).

### Multiple `.reference` sections are allowed

There may be multiple `.reference` note sections in an ELF
object per the [ELF specification][elf], page 1-15:

> The object file format lets one define sections not in the
> list above. An object file may have more than one section
> with the same name.

## Well-known types of `.reference` sections

A minimum set of well-known types are proposed for use as
typed URIs in `.reference` sections:

- Media Type: `text/spdx`

  Purpose: The `.reference` section defines a reference to an
  SPDX document in tagged-value format.

- Media Type: `application/spdx+json`

  Purpose: The `.reference` section defines a reference to an
  SPDX document in JSON format, such as an SPDX 2.2 JSON document.

- Media Type: `application/ld+json; profile="http://spdx.org/rdf/types"`

  Purpose: The `.reference` section defines a reference to an
  SPDX document in JSON-LD format, such as an SPDX 3.0 document.

- Media Type: `application/vnd.cyclonedx+json`

  Purpose: The `.reference` section defines a reference to a
  CycloneDX document in JSON format.

- Media Type: `application/vnd.cyclonedx+xml`

  Purpose: The `.reference` section defines a reference to a
  CycloneDX document in XML format.

- Media Type: `application/x.vnd.purl`

  Purpose: The `.reference` section defines a [package URL][purl].

Other media types *may* be used, provided they meet the [naming
requirements of RFC 6838][rfc6838-naming].

## Examples

*This section is non-normative.*

### Using `data:` URIs to embed text data

*This section is non-normative.*

This example embeds the text `Hello world!` as a `data:` URI
inside a `.reference` section:

```
{
  namesz: 12,
  descsz: 56,
  type: 1,
  name: "text/plain\0\0",
  desc: "data:text/plain;charset=utf-8;base64,SGVsbG8gd29ybGQh\0\0\0"
}
```

### Referencing a local document

*This section is non-normative.*

This example embeds `file:///var/db/sbom/example.spdx` as a
`.reference` section:

```
{
  namesz: 12,
  descsz: 32,
  type: 1,
  name: "text/spdx\0\0\0",
  desc: "file:///var/db/sbom/example.spdx"
}
```

### Referencing an SPDX SBOM document

*This section is non-normative.*

This example embeds `https://example.org/sbom.spdx` as a
`.reference` section:

```
{
  namesz: 12,
  descsz: 32,
  type: 1,
  name: "text/spdx\0\0\0",
  desc: "https://example.org/sbom.spdx\0\0\0"
}
```

### Referencing a CycloneDX SBOM document

*This section is non-normative.*

This example embeds `https://example.org/sbom.cdx.json` as a
`.reference` section:

```
{
  namesz: 32,
  descsz: 36,
  type: 1,
  name: "application/vnd.cyclonedx+json\0\0",
  desc: "https://example.org/sbom.cdx.json\0\0\0"
}
```

### Referencing a Package URL

*This section is non-normative.*

This example embeds `pkg:apk/alpine/nano@6.3-r0?arch=x86_64`
as a `.reference` section:

```
{
  namesz: 24,
  descsz: 40,
  type: 1,
  name: "application/x.vnd.purl\0\0",
  desc: "pkg:apk/alpine/nano@6.3-r0?arch=x86_64\0\0"
}
```

## Security considerations

*This section is non-normative.*

### Authentication of remote references

Remote references may be altered or deleted by the party
which controls the domain the referenced resource is hosted
on.

Referenced resources *should* be authenticated by any
client fetching them using normal authentication methods
such as hash commitments, content-addressable storage,
or signatures, before they are considered trustworthy.

## References

* [Software Bill of Materials][sbom] specification as written
  by CISA.

   [sbom]: https://www.cisa.gov/sbom

* [Executable and Linking Format][elf] specification as written
  by the TIS Committee.

   [elf]: https://refspecs.linuxfoundation.org/elf/elf.pdf

* [RFC 6838][rfc6838] specification as published by the IETF,
  ISSN 2070-1721.

   [rfc6838]: https://www.rfc-editor.org/rfc/rfc6838
   [rfc6838-naming]: https://www.rfc-editor.org/rfc/rfc6838#section-4.2

* [Package URL][purl] specification.

   [purl]: https://github.com/package-url/purl-spec
