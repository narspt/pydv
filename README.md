# pydv

Collection of Python code to experiment with D-STAR and the proposed [vocoder extension](#d-star-vocoder-extension) that allows the use of the open source [Codec 2](http://www.rowetel.com/codec2.html) with D-STAR.

Provides Python interfaces to manage DExtra and DPlus connections (protocols used by reflectors), convert from network data to D-STAR streams (header and frames) and vice versa, as well as encode and decode voice data using [mbelib](https://github.com/szechyjs/mbelib) (decode only) and [codec2](https://svn.code.sf.net/p/freetel/code/codec2/branches/).

Installs the following executables:
* `dv-recorder`, which connects to a reflector and records traffic in .dvtool files
* `dv-player`, which plays back a .dvtool file to a reflector
* `dv-encoder`, which converts a .wav fle to a .dvtool file using the Codec 2 vocoder
* `dv-decoder`, which converts a .dvtool file using any vocoder to .wav
* `dv-transcoder`, which converts a .dvtool file using the AMBE vocoder to a .dvtool file using the Codec 2 vocoder

## D-STAR vocoder extension

I propose the use of the "Flag 3" byte of the header, to mark the vocoder type in the voice frames, as follows (in accordance to section 2.1.1, page 4 of the [D-STAR specification](https://www.jarl.com/d-star/shogen.pdf)):

| Bit | Meaning | Function |
| --- | ------- | -------- |
| `0000000x` | Vocoder | `0`: AMBE (backwards compatible)<br/>`1`: Codec 2 |
| `000000x1` | Codec 2 mode | `0`: 3200 (160 samples/20 ms into 64 bits)<br/>`1`: 2400 (160 samples/20 ms into 48 bits) |
| `00000x?1` | Enable FEC | `0`: No<br/>`1`: Yes |
| `00001000` to `11111111` | Undefined | Use for future expansion |

**FEC is currently not implemented.** _The space available in the frame for voice data is 72 bits, so that leaves us with 8 bits in the case of 3200 mode and 24 in the case of 2400 mode. The latter is enough for 22 bits of FEC, as [FreeDV](https://freedv.org) does in 2400 and 1850 modes._

The vocoder extension is compatible with all current D-STAR hardware (repeaters, hotspots, etc.) and software (repeater controllers, reflectors, etc.), except - of course - transceivers that assume voice data to be in AMBE format and use the corresponding chip for processing.

D-STAR reflectors, like [xlxd](https://github.com/LX3JL/xlxd), can be used to transcode and bridge the two formats. Although implementations of the vocoder extension can use the same reflector connections as AMBE-only clients, it should be avoided, to save the user confusion and establish interoperability.

A solution is to implement another DExtra listener on a different port (30201 instead of 30001). The new port is to be used by reflectors using the "ORF" prefix (Open ReFlector). Any client connected to an ORF reflector will receive streams encoded with Codec 2. All other D-STAR protocol handlers will still send out data encoded with AMBE. Note that the protocol/port only affects data transmitted by the reflector. The stream vocoder should be recognized by all protocol handlers, so a client can still transmit data using any vocoder on any port. The rationale behind this is that DExtra links may be used by repeaters or other reflectors, so it is not really possible to know what their clients support. So, nothing will change when linking a repeater to an XRF reflector, but will do when linking to an ORF one. This work is currently being done in an [xlxd fork](https://github.com/chazapis/xlxd).

The open source vocoder, allows homebrewing transceivers using a [Rasbperry Pi](https://www.raspberrypi.org), an [MMDVM modem](https://github.com/g4klx/MMDVM) (even [one constructed with through-hole components](https://www.florian-wolters.de/blog/2016/02/25/handcrafted-mmdvm-adapter/)), and an old radio. Thus, one could use a D-STAR hotspot as a transceiver, assuming a method to attach a microphone and speaker. 

`dv-encoder`, `dv-decoder` and `dv-transcoder` implement the vocoder extension.

## Building

To build, you must first build and install [mbelib](https://github.com/szechyjs/mbelib) and [codec2](https://svn.code.sf.net/p/freetel/code/codec2/branches/).

On Mac OS X, I used [MacPorts](https://www.macports.org) to install `cmake`, `speexDSP`, and `libsamplerate`. [mbelib](https://github.com/szechyjs/mbelib) compiles and installs to `/usr/local` without problems. To build [codec2](https://svn.code.sf.net/p/freetel/code/codec2/branches/), I had to `export LIBRARY_PATH=$LIBRARY_PATH:/opt/local/lib` before running `make`, and edit the following files to remove unsupported `gcc` flags (from the `build` folder):
* `unittest/CMakeFiles/ofdm_stack.dir/flags.make`, to remove `-fstack-usage`
* `unittest/CMakeFiles/ofdm_stack.dir/link.txt`, to remove `-Wl,-Map=ofdm_stack.map`

---

Based on [ircDDBGateway](https://github.com/g4klx/ircDDBGateway) and [xlxd](https://github.com/LX3JL/xlxd). Tested with an [xlxd fork](https://github.com/chazapis/xlxd).

73 de SV9OAN
