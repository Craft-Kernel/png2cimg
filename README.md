# img2cimg.py

Turns a PNG into a `.cimg`, the picture format CraftOS loads. Every picture built into CraftOS
(the wallpaper, the icons, the Solitaire cards) went through this.

It only needs Python 3's standard library, since it decodes the PNG itself.

## Using it

```sh
python tools/assets/img2cimg.py INPUT.png OUTPUT.cimg --size WIDTHxHEIGHT [--fit cover|contain] [--format rle|rgb] [--trim]
```

| Option | What it does |
| --- | --- |
| `--size 32x32` | The size of the `.cimg`. Required. |
| `--fit contain` | Fits the whole picture inside that size, with see-through margins if the shapes don't match. The default. |
| `--fit cover` | Fills the whole size and crops whatever hangs over. Good for wallpapers. |
| `--format rle` | Run-length ARGB, with transparency. Small for icons and flat colours. The default. |
| `--format rgb` | Raw RGB, no transparency. Better for photos and gradients, where neighbouring pixels rarely match. |
| `--trim` | Crops away the see-through margins first (keeping it square), so an icon fills its space. |

Some examples:

```sh
# the wallpaper: fill the 720p screen, as raw rgb
python tools/assets/img2cimg.py "../Craft Assets/CraftOS Wallpaper.png" assets/wallpapers/wallpaper.cimg --size 1280x720 --fit cover --format rgb

# an icon: trim the empty edges, then shrink to 32x32
python tools/assets/img2cimg.py "../Craft Assets/craft-block-1x1.png" assets/icons/craft/craft-32.cimg --size 32x32 --trim
```

It prints the size and how many bytes it wrote.

## What it can read

8-bit RGB or RGBA PNGs that aren't interlaced, which is what most programs save. Anything else
(palette PNGs, 16-bit, greyscale, interlaced) gets a message saying so; save it again as plain
RGBA first. It's pure Python, so big pictures are slow: a 4K PNG takes about a minute.

## After converting

Keep the original PNG in `../Craft Assets`, so it can be converted again later. A brand new
picture also needs a line in `os/fs/assets.S` and one in `os/fs/sysfiles.c` to end up on the
RAM disk (see `assets/README.md`).

## The format

All little-endian:

| Bytes | What |
| --- | --- |
| 4 | `CIMG` |
| 2 | Width |
| 2 | Height |
| 1 | Encoding: 0 = run-length ARGB, 1 = raw RGB |
| 1 | Reserved (0) |
| the rest | The pixels, row by row. Encoding 0: runs of a 1-byte count (1-255) then a 4-byte `0xAARRGGBB`. Encoding 1: 3 bytes per pixel, in the order blue, green, red. |

## AI Use in png2cimg

- README Writing
- Commenting
