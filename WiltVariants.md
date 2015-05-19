# Introduction #

Since Wilt is very simple, it is easy to tweak for particular uses. This page contains some examples and suggestions for this. Feel free to modify the algorithm as you see fit to suit your purposes!

# Wilt-16 #

Wilt-16 is a variant of Wilt, designed to work on 16-bit words instead of bytes. This makes it slightly more efficient at encoding data that is 16-bit aligned (and worse at data which isn't). It also lets the decoder write only 16-bit values to memory, which is useful on platforms where only 16-bit writes are possible, such as when decompressing directly to VRAM on the Nintendo GBA or DS hand-held consoles.

An implementation of Wilt-16 is available in the source code repository.

## Differences ##

  * Lengths and offsets are stored divided by 2, as they will always be even.
  * The minimum length is 4 instead of 3.
  * The minimum offset is 2 instead of 1.

### Literals ###

As a 65535-probability state for literals would be quite excessive, Wilt-16 uses a shortened context for literal bits. Each bit only relies on the four preceeding bits received (or less when four are not yet available). The sets of 16 different probabilities for each bit are also still independent from each other.

# Possible other variants #

  * Wilt does not use a dictionary window, but instead assumes the whole file is available as a dictionary. To reduce memory usage, a dictionary window could be introduced fairly easily. It would just involve limiting the the range of the length and offset values in the compressor. The universal code might be modified to have a maximum size, too, but the gains from this are likely to be quite small.
  * A Wilt-32 variant could easily be constructed along the same lines as Wilt-16.
  * Pre-loading probabilities with other values than the neutral `0x800` should make it possible to compress very small files with known statistics better.
  * Modifying the range code to output using a more limited alphabet than bytes would make it possible to output streams that are URL-safe without needing a separate base 64 encoding step. This could be useful for passing compressed data in URLs or in other over other non-8-bit channels. Some work is already being done on exploring this possibility.