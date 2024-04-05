# Rasn compiler
[![crates.io](https://img.shields.io/crates/d/rasn-compiler.svg)](https://crates.io/crates/rasn-compiler)
[![Help Wanted](https://img.shields.io/github/issues/librasn/compiler/help%20wanted?color=green)](https://github.com/librasn/compiler/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
[![Lines Of Code](https://tokei.rs/b1/github/librasn/compiler?category=code)](https://github.com/XAMPPRocky/tokei)
[![Documentation](https://docs.rs/rasn-compiler/badge.svg)](https://docs.rs/rasn-compiler/)

[Try compiling some ASN.1 online.](https://librasn.github.io/)

The `rasn-compiler` library is a parser combinator that parses ASN.1 specifications and outputs
encoding-rule-agnotic bindings for ASN.1 data elements to be used with the [`rasn`](https://github.com/librasn/rasn) crate.
The compiler heavily relies on the great library [nom](https://docs.rs/nom/latest/nom/) for its basic parsers.
The parser has been designed to generate rasn bindings for ASN.1. It should not be used as a validating tool for ASN.1 modules (yet), since some of the parsers are more lenient than the ASN.1 syntax specification. The compiler aims to become fully compliant with X680 in this regard.

## Example
In order to compile ASN.1 in your build process, invoke the `rasn-compiler` in your [`build.rs` build script](https://doc.rust-lang.org/cargo/reference/build-scripts.html).

```rust
// build.rs build script
use std::path::PathBuf;
use rasn_compiler::prelude::*;

// The compiler's default backend can be replaced
// with a custom backend to generate bindings for
// a different language or framework.
struct CustomBackend;

impl Backend for CustomBackend {
    fn generate_module(
        &self,
        top_level_declarations: Vec<ToplevelDefinition>,
    ) -> Result<GeneratedModule, GeneratorError> {
        // implement a custom backend to generate bindings in a language of your choice
        Ok(GeneratedModule::empty())
    }
}

fn main() {
  // Initialize the compiler
  match Compiler::new()
    // optionally provide a custom backend
    .with_backend(CustomBackend)
    // add a single ASN1 source file
    .add_asn_by_path(PathBuf::from("spec_1.asn"))
    // add several ASN1 source files
    .add_asn_sources_by_path(vec![
        PathBuf::from("spec_2.asn"),
        PathBuf::from("spec_3.asn"),
    ].iter())
    // set an output path for the generated rust code
    .set_output_path(PathBuf::from("./asn/generated.rs"))
    // you may also compile literal ASN1 snippets
    .add_asn_literal(format!(
        "TestModule DEFINITIONS AUTOMATIC TAGS::= BEGIN {} END",
        "My-test-integer ::= INTEGER (1..128)"
    ))
    .compile() {
    Ok(warnings /* Vec<Box<dyn Error>> */) => { /* handle compilation warnings */ }
    Err(error /* Box<dyn Error> */) => { /* handle unrecoverable compilation error */ }
  }
}
```

## CLI
The `rasn-compiler` provides a CLI application that can be activated with the `cli` cargo feature.
Run `./rasn_compiler_cli -h` for usage info.

## ASN1 Support
ASN1 is a complex standard, and not all of its features and encoding rules are supported, yet.

Currently, `rasn` supports the following encoding rules:
* Basic Encoding Rules (BER)
* Canonical Encoding Rules (CER)
* Distinguished Encoding Rules (DER)
* Aligned Packed Encoding Rules (APER)
* Unaligned Packed Encoding Rules (UPER)
* JSON Encoding Rules (JER)

`rasn` and the `rasn-compiler` support the following ASN1 features:

#### Types
* `NULL` type and value
* `BOOLEAN` type and value
* `NumericString` type and value
* `VisibleString` type and value
* `IA5String` type and value
* `GeneralString` type and value
* `UTF8String` type and value
* `BMPString` type and value
* `PrintableString` type and value
* `BIT STRING` type and value (hex- and bitstring declations)
* `OCTET STRING` type and value (hex- and bitstring declations)
* `OBJECT IDENTIFIER` type and value
* `SEQUENCE` type and value
* `SET` type and value
* `SEQUENCE OF` type and value
* `SET OF` type and value
* `ENUMERATED` type and value
* `CHOICE` type and value
* `UTCTime` type and value
* `GeneralizedTime` type and value

#### Constraints
* Single value constraints
* Value range constraints
* Contained subtype constraints
* Size constraints
* Permitted alphabet constraints
* Constraint set operations
* Table constraints

#### Misc
* `DEFAULT` member values
* `COMPONENTS OF` notation
* Choice selection type notation (e.g. `option-1 < Example-choice`)
* extensions and extension groups
* Parameterization (the `rasn-compiler` creates rust representations for invocations of the parameterized data elements in the given spec, i.e. it does not preserve the parameterization itself)
* Information Object Classes (however, they are not represented in the rust bindings)
* Information Objects
* Information Object Sets
