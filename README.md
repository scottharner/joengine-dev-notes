# Developer notes for Jo Engine

## Author note

[Jo Engine](https://github.com/johannes-fetz/joengine) is a game engine for Sega Saturn. I created this document to share my findings with others and give them a chance to contribute. I had a hard time finding documentation and hopefully I will leave something here that makes it easier for the next guy.

## Graphics

- Why does my tile show garbled when using jo engine and the map editor?
  - You cant use compression when saving a graphic as TGA.
- What is the default resolution for jo engine games?
  - 320x240 when JO_NTSC=1
  - 320x224 when JO_NTSC=0
    - I got 320x240 for TV dimensions but could only draw sprite tiles within a 320x224 region.
- What is the difference between jo_sprite_draw3D and jo_sprite_draw3D2?
  - The first one uses center aligned coordinates and the second one uses top left coordinates.
- Why can the Saturn not find my TGA file?
  -  It appears that TGA names might have to be 8 characters or fewer.
  -    

## Builds

- What does JO_COMPILE_USING_SGL do?
  - This makes Jo Engine be a wrapper around SGL.
  - If you disable this, it puts Jo engine into an experimental mode that may be buggy.

## Sound

- How can I get my sound effects to play with Jo Engine?
  - I could never get this to work to satisfaction.
  - With the sample provided sound demo I tried to play my own sound and it was very garbled and unrecognizable.
  - After working with some settings on ffmpeg, I eventually got my sound to be recognizable but it was still mildly glitched.
- How can I use [ponesound](https://github.com/ponut64/SCSP_poneSound) to play sound effects?
  - Disable the Jo Engine audio module in your makefile.
  - Keep in mind that you can only load so many sounds at a time.
    - I tried to load one sound in addition to the ones that come with the demo and it would not load due to lack of memory.
    - You can unload sounds as needed using the pcm_reset function.
    - If you run out of memory there is a silent failure and no sounds will play.
  - You need to convert your sound to PCM with a utility like [ffmpeg](https://ffmpeg.org/).
    - An example command line is: ffmpeg -i "mySound.WAV" -f s8 -ac 1 -ar 15360 "mySound.PCM"
  - You can load your sound in code with a command like: mySoundId = load_8bit_pcm((Sint8 *)"PICKUP.PCM", 15630);
  - You can play your sound in code with a command like: pcm_play(mySoundId, PCM_PROTECTED, 6);
  - To include ponesound you need to:
    - Download the [source](https://github.com/ponut64/SCSP_poneSound).
    - Grab pcmsys.h and pcmsys.c from the jo_demo example and drop them in your project folder.
    - Add an include statement in your code for pcmsys.h.
    - Grab sdrv.bin and drop it in your cd folder.
    - Add the following calls in your code (refer to the jo_demo for examples):
      - load_drv(ADX_MASTER_2304);
      - jo_core_add_vblank_callback(sdrv_vblank_rq);
- Can I put my sounds in subfolders under cd when using ponesound?
  - It is possible but not recommended.
  - Subfolder usage requires calling sbl functions, which is slow.

## Documentation

- Are there any documentation sources other than the official Jo Engine website?
  - [Emerald Nova tutorials](https://emeraldnova.com/shrec/joengine000.php)

## Miscellaneous

- Why are my data structures getting corrupted early in my game?
  - This can happen if you mistakenly call jo_malloc before jo_core_init. jo_core_init must be called first!
- How do I get true random numbers with Jo Engine?
  - Some talk about calling time functions but that was not working for me when trying to seed the random number generator.
    - I wonder if this was because I was using an emulator.
  - I ended up setting up a counter to increment every draw of the title screen and use that counter to seed once the user pressed start.
  - The seed needs to be set by assigning the value to jo_random_seed.
