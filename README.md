# Schemars

[![Travis (.org)](https://img.shields.io/travis/GREsau/schemars?logo=travis)](https://travis-ci.org/GREsau/schemars)
[![Crates.io](https://img.shields.io/crates/v/schemars)](https://crates.io/crates/schemars)
[![Docs](https://docs.rs/schemars/badge.svg)](https://docs.rs/schemars)

Generate JSON Schema documents from Rust code

Work in progress!

## Basic Usage

If you don't really care about the specifics, the easiest way to generate a JSON schema for your types is to `#[derive(JsonSchema)]` and use the `schema_for!` macro. All fields of the type must also implement `JsonSchema` - Schemars implements this for many standard library types.

```rust
use schemars::{schema_for, JsonSchema};

#[derive(JsonSchema)]
pub struct MyStruct {
    pub my_int: i32,
    pub my_bool: bool,
    pub my_nullable_enum: Option<MyEnum>,
}

#[derive(JsonSchema)]
pub enum MyEnum {
    Unit,
    StringNewType(String)
}

fn main() {
    let schema = schema_for!(MyStruct);
    println!("{}", serde_json::to_string_pretty(&schema).unwrap());
}
```

<details>
<summary>Click to see the output JSON schema...</summary>

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MyStruct",
  "type": "object",
  "required": [
    "my_bool",
    "my_int",
    "my_nullable_enum"
  ],
  "properties": {
    "my_bool": {
      "type": "boolean"
    },
    "my_int": {
      "type": "integer",
      "format": "int32"
    },
    "my_nullable_enum": {
      "anyOf": [
        {
          "$ref": "#/definitions/MyEnum"
        },
        {
          "type": "null"
        }
      ]
    }
  },
  "definitions": {
    "MyEnum": {
      "anyOf": [
        {
          "enum": [
            "Unit"
          ]
        },
        {
          "type": "object",
          "required": [
            "StringNewType"
          ],
          "properties": {
            "StringNewType": {
              "type": "string"
            }
          }
        }
      ]
    }
  }
}
```
</details>

### Serde Compatibility

One of the main aims of this library is compatibility with [Serde](https://github.com/serde-rs/serde). Any generated schema *should* match how [serde_json](https://github.com/serde-rs/json) would serialize/deserialize to/from JSON. To support this, Schemars will check for any `#[serde(...)]` attributes on types that derive `JsonSchema`, and adjust the generated schema accordingly.

```rust
use schemars::{schema_for, JsonSchema};
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize, JsonSchema)]
#[serde(rename_all = "camelCase")]
pub struct MyStruct {
    #[serde(rename = "myNumber")]
    pub my_int: i32,
    pub my_bool: bool,
    #[serde(default)]
    pub my_nullable_enum: Option<MyEnum>,
}

#[derive(Deserialize, Serialize, JsonSchema)]
#[serde(untagged)]
pub enum MyEnum {
    Unit,
    StringNewType(String)
}

fn main() {
    let schema = schema_for!(MyStruct);
    println!("{}", serde_json::to_string_pretty(&schema).unwrap());
}

```

<details>
<summary>Click to see the output JSON schema...</summary>

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MyStruct",
  "type": "object",
  "required": [
    "myBool",
    "myNumber"
  ],
  "properties": {
    "myBool": {
      "type": "boolean"
    },
    "myNullableEnum": {
      "anyOf": [
        {
          "$ref": "#/definitions/MyEnum"
        },
        {
          "type": "null"
        }
      ]
    },
    "myNumber": {
      "type": "integer",
      "format": "int32"
    }
  },
  "definitions": {
    "MyEnum": {
      "anyOf": [
        {
          "type": "null"
        },
        {
          "type": "string"
        }
      ]
    }
  }
}
```
</details>

`#[serde(...)]` attributes can be overriden using `#[schemars(...)]` attributes, which behave identically (e.g. `#[schemars(rename_all = "camelCase")]`). You may find this useful if you want to change the generated schema without affecting Serde's behaviour, or if you're just not using Serde.

## Feature Flags
- `chrono` - implements `JsonSchema` for all [Chrono](https://github.com/chronotope/chrono) types which are serializable by Serde.
- `derive_json_schema` - implements `JsonSchema` for Schemars types themselves

