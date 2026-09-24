# ASH True Stereo BRIR → SPARTA MatrixConv Converter

A small browser-based utility for converting 4-channel **ASH Listening Set True Stereo BRIR** WAV files into the 2-channel time/channel layout expected by **SPARTA MatrixConv**.

The converter is designed specifically to preserve the original audio sample bytes. It does not decode the audio through the Web Audio API, resample it, change sample values, or apply DSP.

## What it does

ASH True Stereo BRIR files use four channels:

| Input channel | Meaning |
|---|---|
| CH1 | LL |
| CH2 | RL |
| CH3 | LR |
| CH4 | RR |

The converter rearranges these channels into a 2-channel WAV layout suitable for MatrixConv:

```
Input: 4 channels × N frames

CH1 = LL
CH2 = RL
CH3 = LR
CH4 = RR


Output: 2 channels × 2N frames

CH1 = [LL for N frames][RL for N frames]
CH2 = [LR for N frames][RR for N frames]
```

In other words, the operation is a **data rearrangement**, not an audio conversion in the usual DSP sense.

## Why "sample-byte lossless"?

"Lossless" here specifically refers to the audio sample data.

The converter copies the raw bytes of every input sample directly from the WAV `data` chunk and only changes their order. It does **not**:

- decode samples through `AudioContext` or `decodeAudioData()`
- resample
- convert between integer and floating-point representations
- change sample values
- apply EQ, filtering, normalization, or any other DSP
- re-encode the audio

The output WAV file itself is **not byte-for-byte identical** to the input file, because its channel count, WAV metadata, data layout, and data-chunk size are intentionally changed. Therefore, "sample-byte lossless" is the precise meaning intended here.

## How to use

1. Open `index.html` in a modern web browser.
2. Select or drag a 4-channel ASH True Stereo WAV file onto the converter.
3. The converted WAV file will be downloaded automatically.
4. In SPARTA MatrixConv, set **Number of Input Channels** to **2**.
5. Load the resulting `*_MatrixConv_Lossless.wav` file into SPARTA MatrixConv.

No server, upload, account, or external dependency is required. The WAV file is processed locally in the browser.

## Supported WAV formats

The current implementation expects:

- RIFF/WAVE files
- exactly 4 input channels
- WAV format tags:
  - PCM (`0x0001`)
  - IEEE floating point (`0x0003`)
  - WAVE_FORMAT_EXTENSIBLE (`0xFFFE`)

The implementation preserves the existing sample encoding rather than converting it.

It is intentionally a small RIFF/WAVE parser rather than a general-purpose audio transcoder. Formats outside the assumptions above may not work.

## Technical implementation

The converter reads the WAV file as an `ArrayBuffer` and parses its RIFF chunks using `DataView`.

For the `fmt ` chunk, it changes the channel count from 4 to 2 and updates the corresponding byte rate and block alignment. For WAVE_FORMAT_EXTENSIBLE files, the channel mask is changed to FL|FR.

For the `data` chunk, the original sample bytes are copied directly into the output in the required order:

```
Output CH1 = input CH1 + input CH2
Output CH2 = input CH3 + input CH4
```

Here "+" means concatenation in time, not numerical addition or mixing.

## Privacy

All processing takes place locally in the browser. The selected WAV file is not uploaded to a server by this application.

## AI generation

The code in this repository was generated entirely by **OpenAI GPT-5.6 Luna**, based on the project requirements and instructions provided by the repository author.

The repository is published primarily as a practical utility and a record of the generated implementation.

## Project status

This is a small, purpose-built utility rather than a general-purpose WAV conversion framework. Contributions and adaptations are welcome.

## Related software

- **SPARTA** — toolbox for spatial audio processing and reproduction.
- **ASH Listening Set** — the BRIR dataset for which this conversion workflow was developed.
