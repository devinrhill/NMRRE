# Namco Museum Remix Reverse Engineering

## About
I really love this goofy almost shovelware-type game, and have been the only one reversing it for the purposes of modding (to my knowledge).
I always wished that there was more information about this game and how it works online.
So I thought it would be a good idea to create a repository of information for anybody like me who wants to mod this game.

### Important Information
To be completely honest, for now at least, I probably won't have any of this stuff organized.

- For decompilation/disassembly, I will be using the PAL version of the game

# File formats reversing
## Namco's in-house archival format, ARCV
| Offset |   Description   | Value |             Notes             |
|--------|-----------------|-------|-------------------------------|
|   0x0  | Magic signature |  VCRA |
