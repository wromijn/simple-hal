# Halo

## What?

A Java library to produce API output in the [HAL](http://stateless.co/hal_specification.html) format. It lets you create representation objects that serialize with Jackson.

## Why?

#### No magic
An API is a contract and it should be easy to verify and maintain this contact. I wanted a library with as little magic as possible. No automatic behaviour with limited and obscure overrides.

#### No duplicate code
Many libraries make it hard or impossible to re-use parts of the output generation code. I want to be able to define both a ProductSummary and a Product representation and not duplicate the code for the common fields.

## Installation

Use your favourite build automation tool to install the library from the JitPack registry: https://jitpack.io/#wromijn/halo

## Usage

#### Creating a Representation object

```java
Representation representation = new Representation();
```

Use Jackson to serialize the representation.

#### Adding properties
```java
representation.addString("property_name", "String value")
              .addInteger("integer_field", 1)
	      .addNumber("number_field", 1F)
	      .addBoolean("boolean_field", true);
```

Serializes to

```json
{
	"property_name": "String value",
	"integer_field": 1,
	"number_field": 1.0,
	"boolean_field": true
}
```

Like JSON-Schema, Halo makes a clear distinction between numeric fields that can contain floating point values and numeric fields that can not. To emphasize this distinction, "integer" type fields are serialized without a fractional part and "number" type fields are always serialized as floating point values (1 is output as 1.0). This way, an API user can tell even without referring to a schema that a floating point data type is needed to contain the value.

## Adding links

```java
// simple link
representation.addLink("self", "http://api.example.com/item/1")

// link with properties
representation.addLink("self", new Link("http://api.example.com/item/1").setTitle("title"))

// or set the href with a method call
representation.addLink("self", new Link().setHref("http://api.example.com/item/1").setTitle("title"))

// list of links
representation.addLinkList("curies", Arrays.asList(new Link("/link1").setName("ns")))
```

## Embedding objects

Embedding is always done by embedding another Representation object. For example:

```java
private Representation createChildRepresentation(Child c) {
	...
}

// simple embedding
representation.addEmbedded("hal:child", createChildRepresentation(child))

// embedding a stream of objects - you could embed a list instead, but this is quicker
representation.addEmbeddedList("hal:children", children.stream().map(this::createChildRepresentation))
```

## Inlining objects

Inlining is like embedding, but the object is added to the properties instead of the `"_embedded"` section. Inline objects can only have properties: `_links` and `_embedded` sections are removed when they exist.

#### Embedding
```json
{
	"foo": "bar",
	"_embedded": {
		"hal:child": {
			"key": "value"
		}
	}
}
```

#### Inlining
```json
{
	"foo": "bar",
	"child": {
		"key": "value"
	}
}
```

#### Example code:
```java
private Representation createChildRepresentation(Child c) {
	...
}

// simple inlining
representation.addInline("child", createChildRepresentation(child))

// inlining a stream of objects - you could inline a list instead, but this is quicker
representation.addInlineList("children", children.stream().map(this::createChildRepresentation))
```

## Merging Representations
If you want to copy all properties, links and embedded objects of another representation into the current one, you can use `representation.include(another_representation)`. This allows for easy composition of Representations out of several components.

## Adding curies

Simply use the `addCurie(String curie, String href)` method:
```java
representation.addCurie("hal", "http://www.example.com/docs/{rel}");
```

This will produce the following json:
```json
{
    "_links": {
        "curies": [
            {
                "href": "http://www.example.com/docs/{rel}",
                "templated": true,
                "name": "hal"
            }
        ]    
    }
}
```

Duplicate curies (curies in embedded representations that already exist in the parent
representation) will be removed from the output when the object is serialized. This way, you can add the curies that
your application uses to a base representation without sprinkling curie definitions all
over the place.

## Extras

#### Adding whole objects as a property
Provided that the object is serializable with Jackson, you can add a whole object as a property:
```java
addOther(String name, Object o);
```
This is a nice option to have if you want to add an Array or schemaless data as a property.

#### Creating a representation from an object
If you're working with legacy code or just want to add links to an existing model, you can upgrade it to a Representation object.
This will serialize your object to a Jackson JsonNode and create a new Representation from its properties.

```java
Representation.fromObject(Object o);
Representation.fromObject(Object o, ObjectMapper om);
```

[![Build Status](https://travis-ci.org/wromijn/halo.svg?branch=master)](https://travis-ci.org/wromijn/halo)
[![codecov](https://codecov.io/gh/wromijn/halo/branch/master/graph/badge.svg)](https://codecov.io/gh/wromijn/halo)
[![Maintainability](https://api.codeclimate.com/v1/badges/e384ffe146c10612337e/maintainability)](https://codeclimate.com/github/wromijn/halo/maintainability)
[![](https://jitpack.io/v/wromijn/halo.svg)](https://jitpack.io/#wromijn/halo)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
