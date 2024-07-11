This is a small utility to help generate test cases for
[github.com/veraison/corim](https://github.com/veraison/corim). It compiles
YAML documents into CBOR. In general YAML structures are converted into
corresponding CBOR structures with two exceptions:

- A map with exactly two keys, "tag" and "value", is converted into a CBOR tag,
  where the value of "tag" is used as the tag number (it must be an int), and
  the value of "value" is the tagged structure.
- A map with exactly one key, "encodedCBOR" is converted into a binary string,
  the contents of which is the CBOR-encoded structure corresponding to that key.

This tool is _almost_ compatible  with the output of `cbor2yaml.rb` from
[cbor-diag](https://github.com/cabo/cbor-diag), with the exception that the
application-specific `!binary` YAML tags generated by the that tool need to be
replaced with the global `!!binary` tags in order to work.

For example, to generate input YAML for this tool from an existing CBOR
document, you can:

    cbor2yaml.rb /path/to/document.cbor | sed 's/!binary/!!binary/g' > document.yaml

(assuming you have `cbor-diag` tools installed).

> [!Note]
> While this can be used as a generic tool for compiling YAML into CBOR, it was
> specifically designed to help generate test cases for `corim` library, and
> its general usefulness outside that was not a consideration.

### Installation

    go install github.com/setrofim/gen-testcase@latest

### Usage

    gen-testcase samples/corim-full.yaml -o /tmp/unsigned-corim.cbor

see output of `gen-testcase -h` for all options.

### COSE_Sign1

In addition to generating a CBOR document, this tool can optionally wrap it in
a [COSE_Sign1](https://datatracker.ietf.org/doc/html/rfc8152#section-4.2)
message. This is done by specifying a signing key with `-s/--signing-key`
option. The key must be in [JWK](https://www.rfc-editor.org/rfc/rfc7517)
format. The content type header is set to `application/rim+cbor`, but an
alternative may be specified with `-c/--content-type` option. Finally,
additional arbitrary data maybe encoded and included in the protected header
under the label `8` with `-m/--meta` option.

    gen-testcase -s samples/ec-p256.jwk -m samples/meta.yaml \
            samples/corim.yaml -o /tmp/signed-corim.cbor