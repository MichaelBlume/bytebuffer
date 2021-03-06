# bytebuffer

Library for packing and unpacking binary data. Simplifies working with
[java.nio.ByteBuffer](http://java.sun.com/j2se/1.5.0/docs/api/java/nio/ByteBuffer.html)
objects.

Handles signed and unsigned values pleasantly. Usually reading or
writing unsigned fields with ByteBuffers is a pain because Java
doesn't have unsigned primitives. The unsigned take-* functions here
actually return a type that is one step larger than the requested
type, ex. take-ushort returns an int, and take-uint returns a
bigint.

See the [API Docs](http://geoffsalmon.github.com/bytebuffer/index.html)

If you're serializing or deserializing data, you should also checkout
[gloss](https://github.com/ztellman/gloss).

## Usage

Use put-* and take-* functions for each data type add to and remove
from a bytebuffer.

    (def buff (byte-buffer 100))
    (put-int buff 84)
    (put-byte buff -44)
    (put-byte buff -44)
    (.flip buff)
    (take-int buff) => 84
    (take-byte buff) => -44
    (take-ubyte buff) => 212 ; -44 is interpreted as 212 when read as unsigned byte

Use with-buffer to bind a default buffer and avoid passing it to every
take-* and put-* function.

    (let [buff (byte-buffer 100)]
      (with-buffer buff
        (put-int 10)
        (put-int 6)
        (.flip buff)
        [(take-int) (take-int)]
      )
    ) => [10 6]

A more compact way to add or remove data is the pack and unpack
functions, which are inspired by Python's struct module. They use
simple format strings, similar to printf's, to define how fields are
arranged in the buffer.

    (pack buff "isbb" 123 43 23 3) ; puts an int, a short and two bytes into buff
    (.flip buff)
    (unpack buff "isbb") => (123 43 24 3)

Use the functions pack-bits and unpack-bits to work with bit fields
within numbers.

    (pack-bits 2 3 2 true 3 false 1 1) => 2r11010001

    (unpack-bits 2r101011001 4 \b 1 1 2) => (10 true 1 0 1)

When parsing binary file or packet structures it's often useful to
pull off a larger piece of data as a separate buffer using slice-off.

    (defn parse-header [buff] ...)
    (defn parse-body [buff] ...)

    (let [hdr (parse-header (slice-off buff 18)) ; slice off 18 byte header
          body (parse-body (slice-off buff (:len hdr))] ; parse body
       ...
    )

See the
[tests](http://github.com/geoffsalmon/bytebuffer/blob/master/test/bytebuffer/buff_test.clj)
for more examples.

## Installation

Either grab the source or use the jar from clojars. Version 0.2.0
works with Clojure 1.3.

### Leiningen

Add `[bytebuffer "0.2.0"]` to :dependencies in project.clj.

### Maven

Add this dependency

    <dependency>
      <groupId>bytebuffer</groupId>
      <artifactId>bytebuffer</artifactId>
      <version>0.2.0</version>
    </dependency>

and ensure you have the following maven repository added.

    <repository>
      <id>clojars.org</id>
      <url>http://clojars.org/repo</url>
    </repository>

## License

Copyright (c) Geoff Salmon

Licensed under [EPL 1.0](http://www.eclipse.org/legal/epl-v10.html)
