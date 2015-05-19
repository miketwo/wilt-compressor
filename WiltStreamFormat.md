# Introduction #

Wilt is a very straightforward implementation of [LZSS](http://en.wikipedia.org/wiki/Lempel-Ziv-Storer-Szymanski),  using an adaptive range coder very similar to the one used by [LZMA](http://www.7-zip.org/sdk.html) for entropy coding. Wilt assumes the file will be compressed into memory (or perhaps into a memory-mapped file) in its entirety, and therefore does not use a dictionary window, but instead uses the entire file as the dictionary.

This text will describe how to decompress a Wilt stream. The method of compression should be easy to infer from this, or from reading the [reference compressor code](http://code.google.com/p/wilt-compressor/source/browse/trunk/Reference/WiltCompress.c).

# Range coder #

The range coder used in Wilt only encodes single bits. Each bit has a probability which is used by the range coder, and then updated by a simple exponential function implemented as shift and subtraction.

## State ##

The range coder maintains two 32-bit unsigned values, the range and the code. The range is initialized to `0xffffffff`, and the code is initialized using the first four bytes of the data stream, in big-endian order.

Every probability is initialized to `0x800` at the start of decoding.

## Normalizing ##

Before reading each bit, the range coder is normalized. A check is done to see if the range is less than `0x1000000`, and if it is, the range is shifted up by 8 bits, and the code is also shifted up by 8 bits and the lowermost 8 bits are filled in with the next byte from the input stream.

## Reading a bit ##

To read a bit, a threshold value is calculated as `(range>>12)*probability`, where `probability` is the probability of this bit being zero, with a range of `1` to `0xfff`. If the value of the code is smaller than this, the bit is 0, and the range is set to the threshold value. If the value of the code is equal or larger, the bit is 1, and both the range and code are decreased by the threshold value.

## Updating the probability ##

After reading a bit, the probability value for this particular type of bit is updated. The rate of adaptation is controlled by a parameter value, whose value depends on what kind of bit is read and is selectable at compression time. Different parts of the algorithm use different parameters.

If the bit read was 1, the probability is updated by subtracting the value of the probability shifted down by the number of bits specified by the parameter. If the bit read was 0, the probability is updated by adding the value of `0x1000` minus the probability, shifted down by the number of bits specified by the parameter.

The parameter can have values between 1 and 12, although the higher values are of little use.

## Differences from LZMA's range coder ##

The difference between Wilt's range coder and LZMA's is that Wilt uses a shift of 12 instead of LZMA's 11 (and thus the starting and maximum probability values are double those in LZMA), and that Wilt does not include the extraneous zero byte at the start of the stream that LZMA has.

# Universal code #

To encode numerical values using the range coder which only handles single bits, Wilt uses a simple universal code very similar to [Elias gamma coding](http://en.wikipedia.org/wiki/Elias_gamma_coding). A number is encoded by first writing one more one bit than the number of highest bit set in the number, followed by a zero bit, and then all the bits in the number except for the highest (which is known to be one), in order from most to least significant. A zero is encoded as a single 0 bit.

Each bit has its own probability. The unary bit string at the start always uses the same probability for the first bit, and another for the second, and so on. However, the actual bits of the number in the second half always use the same probability for the _last_ bit, independent of the total number of bits sent, and another for the _next to last_ bit, and so on.

The probabilities for the first unary part and the second binary part are also updated using two different shift parameters.

An example of how some numbers are encoded follows:

| **Number** | **Encoding** | **Probabilities** |
|:-----------|:-------------|:------------------|
| 0          | 0            | 0                 |
| 1          | 10           | 0 1               |
| 2          | 110 0        | 0 1 2 a           |
| 3          | 110 1        | 0 1 2 a           |
| 4          | 1110 00      | 0 1 2 3 b a       |
| 5          | 1110 01      | 0 1 2 3 b a       |
| 6          | 1110 10      | 0 1 2 3 b a       |
| 7          | 1110 11      | 0 1 2 3 b a       |
| 15         | 11110 111    | 0 1 2 3 4 c b a   |
| 16         | 111110 0000  | 0 1 2 3 4 5 d c b a |
| 17         | 111110 0001  | 0 1 2 3 4 5 d c b a |

# Stream format #

The stream is encoded as a series of either dictionary matches or literal bytes. Each one starts with a single bit that determines which type of record follows. This bit is encoded with its own probability and shift parameter.

There is no end code, so the uncompressed size must be known to find the end of the stream.

## Dictionary matches ##

If the type bit is 1, this indicates a dictionary match. A match is encoded as two numerical values encoded using the universal code - first a length, then an offset. Both values have their own sets of independent probabilities and shift parameters.

The minimum length is 3, so the length is stored as the actual length minus 3. The offset is stored as the offset from the byte about to be written minus 1. Also note that lengths that extend past the end of the data currently read are legal! Thus, before copying each byte from the earlier parts of the file, care should be taken that the previous byte has been written to the buffer.

Some examples follow:

| **Encoded length** | **Encoded offset** | **Buffer before** | **Buffer after** |
|:-------------------|:-------------------|:------------------|:-----------------|
| 0                  | 2                  | ABCD              | ABCDBCD          |
| 1                  | 3                  | ABCD              | ABCDABCD         |
| 5                  | 1                  | ABCD              | ABCDCDCDCDCD     |

## Literals ##

If the type bit is 0, this indicates a single literal byte follows. The byte is encoded as a bit string, from the most to least significant bit.

Probabilities for the bits of the literals depend on the context of the earlier bits received. Thus, the first bit always uses a single probability. The next bit uses one of two probabilities depending on the previous bit. And the next bit after that uses one of four probabilities depending on the two previous bits. The probabilities at each stage are independent from the ones of earlier stages. Thus there are in total 255 different probabilities used to read literal bits.

Literals use one shift parameter for all bits.

## Header ##

To decode the stream, the decoder needs seven parameters: The size of the uncompressed stream, and six shift parameters for the various probabilities in the algorithm. The shift parameters are used for the type bit, the bits in an encoded literal, and for the two different parameters each in the universal codes for lengths and offsets.

At the moment, Wilt does not have a standardized header. The reference compressor uses a simple 7-byte header to transmit the parameters needed by the decompressor, but it is entirely up to the implementor to decide how to transmit the parameters, or whether to hard-code them.