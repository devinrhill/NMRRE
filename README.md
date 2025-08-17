# Namco Museum Remix Reverse Engineering

## About
I really love this goofy almost shovelware-type game, and have been the only one reversing it for the purposes of modding (to my knowledge).
I always wished that there was more information about this game and how it works online.
So I thought it would be a good idea to create a repository of information for anybody like me who wants to mod this game.

### Important Information
To be completely honest, for now at least, I probably won't have any of this stuff organized.

- For decompilation/disassembly, I will be using the PAL version of the game

# File formats reversing
## ARCV (.arc)
An ARCV archive is **always** in little-endian. Unlike basically every other file in a Wii game,
or even this game, which is in big-endian. The archive data is typically compressed using the LZSS compression algorithm, using the following parameters. The sliding window size is 4096 bytes, the lookahead buffer size is 18 bytes, the minimum match length is 2 bytes, and the initial compressed buffer is set to zero (0x00) bytes. This compressed data is contained alongside some necessary information for compression and decompression, which will be covered directly after this.

### Format

There is an archive header at the very beginning of the file:
|Offset|Description|Value|Notes|
|:----:|:---------:|:---:|:---:|
|0x0|Magic signature|VCRA||
|0x4|Member count|u32 = 0x00000000||
|0x8|File size|u32||
|0xC|Padding|u32 = 0||
|0x10|Unknown value|u32 = 0x20070205|Possibly storing the date of which the archive was packed?|
|0x14-0x40|Padding|44 bytes of zeroes||

Then directly following after the archive header, is an array of member metadatum:
Archive member metadata chunk
|Offset|Description|Value|Notes|
|:----:|:---------:|:---:|:---:|
|0x0|Absolute offset to member data|u32||
|0x4|Size of member data|u32||
|0x8-0x40|Null-terminated member filename||

Then directly following is the pool of member datum.

Note: each member data is padded to the boundary of 64 (0x40) bytes, with a placeholder dummy character 0xA3. This is very important for any archive packers, as the data MUST be aligned to 0x40. The dummy character doesn't matter but it is considered good practice to try and pack new archive file as closely as possible to the originals.

## SSZL (.lzs)
A LZSS-compressed ARCV archive is **always** in little-endian.
### Format
|Offset|Description|Value|Notes|
|:----:|:---------:|:---:|:---:|
|0x0|Magic signature|SSZL||
|0x4|Padding|u32 = 0x00000000||
|0x8|Compressed size|u32||
|0xC|Uncompressed size|u32|Needed for any decompressor's decompressed data buffer, and for error-checking|
|0x10|Compressed Data|unsigned char\[Compressed Size\]|Spans until the end of the file|
