# Namco Museum Remix Reverse Engineering

## About
I really love this goofy almost shovelware-type game, and have been the only one reversing it for the purposes of modding (to my knowledge).
I always wished that there was more information about this game and how it works online.
So I thought it would be a good idea to create a repository of information for anybody like me who wants to mod this game.

### Important Information
To be completely honest, for now at least, I probably won't have any of this stuff organized.

- For decompilation/disassembly, I will be using the US version of the game

# File formats reversing
## ARCV (.arc)
An ARCV archive is **always** in little-endian. Unlike basically every other file in a Wii game,
or even this game, which is in big-endian. The archive data is typically compressed using the LZSS compression algorithm, using the following parameters. The sliding window size is 4096 bytes, the lookahead buffer size is 18 bytes, the minimum match length is 2 bytes, and the initial compressed buffer is set to zero (0x00) bytes. This compressed data is contained alongside some necessary information for compression and decompression, which will be covered directly after this. The archives were very likely packed in Windows XP or something, and they wrote out the magic number in little-endian, and due to the least significant byte being first, it reversed the magic number. And again all of the fields are in little-endian. Internally the game takes the CRC32 of each archive member name and looks up data via the CRC. When repacking an archive, the order does not matter. All that matters is that data is stored along a 64 byte alignment. When you pad the member data, it is faithful to the original format to use the byte 0xA3 to pad to 64 byte alignment.

### Format

There is an archive header at the very beginning of the file:

```
struct ArcvHeader {
    char magic[4]; // VCRA, 0x41524356(ARCV) backwards due to little-endian
    uint32_t n_entries;
    uint32_t size; // size of the entire file
    uint32_t sanity_check; // should be 0
    uint32_t exporter_date; // 0x05020720, or 0x20070205 backwards
    uint8_t padding[44]; // padded to 64 byte boundary
};
```

Then directly following after the archive header, is an array of file entry metadata:

```
struct ArcvEntryMetadata {
    uint32_t offset; // entry offset within archive
    uint32_t size; // entry size
    int8_t filename[0x38]; // padded to 64 byte boundary
};
```

Then directly following is the pool of entry data.

Note: each member data is padded to the boundary of 64 (0x40) bytes, with a placeholder dummy character 0xA3. This is very important for any archive packers, as the data MUST be aligned to 0x40. The dummy character doesn't matter but it is considered good practice to try and pack new archive file as closely as possible to the originals.

## SSZL (.lzs)
A LZSS-compressed ARCV archive is **always** in little-endian. Just an LZSS compressed file with a little sizing metadata.
### Format

```
struct LzsHeader {
    char magic[4]; // SSZL, of course LZSS backwards because little-endian exporting
    uint32_t sanity_check; // MUST be zero. The game will not load the file if this isn't zero. trust
    uint32_t com_size; // compressed size
    uint32_t real_size; // uncompressed size
};
```

Then following the header is just the compressed data of size `com_size`

Note the LZSS compressor/decompressor must have
- The sliding window buffer set to zero, init_chr = 0
- The sliding window size 1 << 12, EI = 12,
- The lookahead buffer size 1 << 4, EJ = 4
