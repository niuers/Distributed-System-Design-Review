1. In large application, code change can't happen instanenously, that means old and new versions of the code, and old and new data formats, may potentially all coexist in the system at the same time. 
2. We need maintain compatibility in both directions
   * Backward compatibility: newer code can read data that was written by older code
   * Forward compatibility: older code can read data that was written by newer code

# Formats for Encoding Data
1. Programs usually work with data in (at least) two different representations:
   * In memory: objects, structs, lists, hash tables,t rees etc.
   * When you want to write data to file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (e.g. JSON)
1. We need translation between the two representations.

## Language-Specific Formats
1. Java: java.io.Serializable/Python: pickle
2. Limitations
   * Encoding is tied to a particular language, and read the data in another language is very difficult
   * The decoding process needs to be able to instantiate arbitrary classes. A frequently source of security problems. 
   * Versioning data is not easy
   * Efficiency is not that good.
1. For these reasons it’s generally a bad idea to use your language’s built-in encoding for anything other than very transient purposes.

## JSON, XML, and Binary Variants

### Textual Formats
1. XML: too verbose, unnecessarily complicated
2. JSON: built-in support in web browsers (a subset of javascript) and simplicity relative to XML
3. CSV: less power, still popular

#### Limitations of Textual Formats
1. A lot of ambiguity around the encoding of numbers. 
   * XML/CSV can't distinguish between a number and a string (except by an exteranl schema)
   * JSON doesn't distinguish integers and floating-point numbers, no precision
      * A problem with large numbers, integers larger than 2^53 can't be exactly represented in double-precision floating-point number.
2. JSON/XML support Unicode character strings but don't support binary strings (sequences of bytes without a character encoding) 
1. There's optial schema support for XML/JSON.
3. CSV doesn't have any schema. 

### Binary Encoding
1. Binary encodings for JSON, XML. 
   * since they don’t prescribe a schema, they need to include all the object field names within the encoded data. That is, in a binary encoding of the JSON document, they will need to include the strings userName, favoriteNumber, and interests somewhere.

#### Apache Thrift and Protocol Buffers (protobuf)
1. Both Thrift and Protocol Buffers require a schema for any data that is encoded.
2. there are no field names needed in their encoding. Instead, the encoded data contains field tags, which are numbers (1, 2, and 3)

##### Field Tags and Schema Evolution
1. an encoded record is just the concatenation of its encoded fields. Each field is identified by its tag number (the numbers 1, 2, 3 in the sample schemas) and annotated with a datatype.
2. You can add new fields to the schema, provided that you give each field a new tag number. If old code (which doesn’t know about the new tag numbers you added) tries to read data written by new code, including a new field with a tag number it doesn’t recognize, it can simply ignore that field. 
3. The datatype annotation allows the parser to determine how many bytes it needs to skip. This maintains forward com‐ patibility: old code can read records that were written by new code.
4. As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning. The only detail is that if you add a new field, you cannot make it required. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added.
5. Similarly to removing a field.

##### Datatypes and schema evolution
1. Changing the datatype of field can be tricky: 

#### Apache AVRO
1. A subproject of Hadoop
2. that there are no tag numbers in the schema.
3. there is nothing to identify fields or their datatypes. The encoding simply consists of values concatenated together. A
4. To parse the binary data, you go through the fields in the order that they appear in the schema and use the schema to tell you the datatype of each field. This means that the binary data can only be decoded correctly if the code reading the data is using the exact same schema as the code that wrote the data. Any mismatch in the schema between the reader and the writer would mean incorrectly decoded data.

##### The writer’s schema and the reader’s schema
