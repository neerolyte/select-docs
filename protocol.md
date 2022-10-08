# Protocol

The serial protocol available on SP Pro inverters allows for querying and writing to arbitrary blocks of memory. This means the client needs to independently know the layout of memory in the inverter.

<!-- TOC -->

- [Protocol](#protocol)
- [Memory space](#memory-space)
- [Messages](#messages)
	- [Query](#query)
	- [Write](#write)
	- [Example Messages](#example-messages)
		- [Hello](#hello)

<!-- /TOC -->

# Memory space

Memory is addressed as 2 byte words i.e it is only possible to address every second byte.

Memory addresses are 32bit and most of the interesting memory appears to be within the 0x0000a000 - 0x0000afff range.

# Messages

There are two request message types:

 * Query - read some memory
 * Write - write some memory

## Query

A query request consists of:

```
TTLLAAAACCCC
```

where:

 * `TT` is the message type - `Q` or `0x51` for Query.
 * `LL` is a byte indicating the number (length) of 2 byte words to return (`0x00` will return 1 word).
 * `AAAA` is a little endian word address (shifting the address by 1 shifts by 2 bytes).
 * `CCCC` is a Cyclic Redundancy Check.

A query response consists of:

 * `TLAAAACC` the exact bytes from the query (including CRC).
 * `MM` the requested memory (the number of 2 byte words specified by L + 1).
 * `CCCC` CRC of the query + memory.

It's possible to request between 2-512 bytes of memory, 

## Write

A write request consists of:

```
TTLLAAAACCDDDDCCCC
```

where:

 * `TT` is the message type - `W` or `0x57` for Write.
 * `LL` is a byte indicating the number (length) of words to be written.
 * `AAAA` is a little endian word address (shifting the address by 1 shifts by 2 bytes).
 * `CCCC` is a Cyclic Redundancy Check (everything to the left).
 * `DDDD` is the data to write, between 1 and 256 words (as specified by `LL` + 1).

## Example Messages

### Hello

Prior to authentication SP LINK sends a request for 0 words from `0xa000` and the SP Pro will respond with the appropriate CRC calculation.

E.g:

```
Query:    0x510000a000009d4b
            TTLLAAAAAAAACCCC
Response: 0x510000a000009d4b0100d819
            TTLLAAAAAAAACCCCMMMMCCCC
```
