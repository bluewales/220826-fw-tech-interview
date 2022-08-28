# Checksums (what are they good for)

> What are checksums and what are they useful for?

A checksum is a number (or an algorithm that produces a number) that represents another, usually larger, piece of data.  The checksum can be used to verify the integrity of the data.  Typically the checksum is transmitted or stored along side the data. When the data is received, the checksum can be recalculated and the compared checksum can be compared to the received checksum.  If they don't match, you can assume that either the data or the checksum have been corrupted and they are usually discarded.

Desirable properties of a checksum are
 - Small changes in the input data result guarantee a change in the output.  For example, an 8 bit checksum might be able to detect any 8 bit flips in the input, but it be possible to find 9 bits flips that produce the same checksum.
 - Unrelated inputs should have a very low chance of producing the same checksum.

Because these properties are surprisingly difficult to produce, it's generally not recommended to create your own checksum algorithm.  Standard CRCs are have been published, have been thoroughly characterized as performing to a certain level, and can usually be implemented very efficiently.