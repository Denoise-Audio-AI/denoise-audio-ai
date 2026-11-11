# denoise-audio-ai

Almost every recording problem you hear in a podcast, interview, lecture capture or voice-over is the same problem: the signal you want is buried in a signal you do not want. Denoising is the process of separating the two. Done well, the listener never notices the processing, only that the voice sounds close, dry and easy to follow. Done badly, it leaves a thin, watery, metallic voice that is more tiring to hear than the original noise.

This guide covers what noise reduction actually does, which noise types it handles well, how to choose between a plugin, a command-line tool and a hosted service, and how to evaluate the output before you ship it.

## What noise reduction actually does

Noise reduction estimates the noise component of a recording and subtracts it from the mix. Classical tools do this with a spectral model: they learn the shape of the noise floor, then attenuate energy that matches that shape. Modern AI models take a different route. They are trained on paired noisy and clean speech, so they learn to reconstruct the voice itself rather than only to suppress frequencies.

That distinction explains the practical difference you hear. A spectral tool removes hiss cleanly because hiss is stationary, but it cannot tell a fan from a sustained vowel, so it leaves artefacts behind. A trained speech model can follow a voice through non-stationary noise, but it can also over-reach, inventing detail or flattening a voice into something that sounds plausible but no longer quite like the speaker.

Both approaches are legitimate. Neither is magic. The rest of this guide is about picking the right one for your material. If you want the background, the [Wikipedia entry on noise reduction](https://en.wikipedia.org/wiki/Noise_reduction) is a reasonable starting point, and the [signal-to-noise ratio](https://en.wikipedia.org/wiki/Signal-to-noise_ratio) article explains the measurement that underlies almost every decision below.

## What denoising can fix, and what it cannot

Noise reduction works best when three conditions hold: the noise is reasonably steady, the voice is clearly louder than the noise, and the noise does not overlap the voice in frequency.

Good candidates:

- Room tone, air-conditioning hum, computer fans and laptop whine
- Electrical hum at 50 or 60 Hz and its harmonics
- Traffic and street noise behind an outdoor interview, when the speaker is close-miked
- Tape hiss and the broadband noise floor of older recordings
- Crowd murmur behind a lavalier microphone, up to a point

Bad candidates:

- Reverb and echo, which are not additive noise but delayed copies of the voice
- Clipping and distortion, which destroy information rather than adding to it
- Clicks, pops, keyboard taps and door slams, which are short and unpredictable
- A voice recorded from across a room with the built-in microphone, where the voice and the noise are at the same level

The last case is the one that generates most disappointment. If a recording arrives with a signal-to-noise ratio of roughly 0 dB, no tool will recover a clean voice, because the information simply is not there. Audacity's own documentation is candid about this: its noise reduction "is not suitable for individual clicks and pops, or irregular background noise such as from traffic or an audience," and results degrade quickly once the noise approaches the level of the speech. You can read the full description, including a good explanation of why over-processing produces "musical noise" artefacts, in the [Audacity Noise Reduction manual](https://manual.audacityteam.org/man/noise_reduction.html).

## Fix the recording before you fix the file

An hour spent on the recording setup saves several hours of repair, and it always sounds better.

1. **Get the microphone closer.** Halving the distance between mouth and capsule raises the direct sound by roughly 6 dB while leaving the room noise unchanged. This is the single highest-leverage change available.
2. **Record a few seconds of room tone.** Every spectral tool needs a noise-only sample to build a profile. Ten seconds of silence at the start of a session costs nothing and makes later processing markedly better.
3. **Kill the obvious sources.** Switch off the air-conditioner, close the window, move the laptop off the desk, and put a rug or a blanket behind the speaker to reduce reflections.
4. **Set levels conservatively.** Peaks should sit around -12 to -6 dBFS. Recorded clipping cannot be undone, and denoising makes clipping sound worse by removing the noise that was masking it.
5. **Keep the original file.** Always process a copy. You will want to re-run the enhancement with different settings, or to prove later what the source sounded like.

## Three ways to denoise, and when each makes sense

### 1. A desktop editor or DAW plugin

This is the right choice when you are already editing in a DAW and want full control over parameters. The classic workflow is a noise profile plus a reduction curve, run before compression and after any click removal, exactly as the Audacity documentation describes. The trade-off is time: every file needs a profile, a preview pass and a judgement call, and the artefacts that appear at aggressive settings require a trained ear to catch.

Reaper, Adobe Audition, Logic and Audacity all include competent spectral tools. The results are predictable and auditable, which matters if you process the same show every week.

### 2. The command line

For batch work, [FFmpeg](https://ffmpeg.org/) is hard to beat. Two filters are worth knowing:

- [`afftdn`](https://ffmpeg.org/ffmpeg-filters.html#afftdn) is an FFT-based denoiser with a tracked noise floor. It handles hum and hiss well and exposes parameters for noise floor, track noise and band noise, so it suits steady room tone.
- [`arnndn`](https://ffmpeg.org/ffmpeg-filters.html#arnndn) runs a recurrent neural network model, the same family of approach as [RNNoise](https://github.com/xiph/rnnoise). It is far better on non-stationary noise such as traffic and chatter, at the cost of a model file and more CPU.

A minimal invocation looks like this:

```bash
ffmpeg -i input.wav -af "afftdn=nf=-25" -c:a pcm_s16le output.wav
```

Command-line processing is reproducible, scriptable and free. What it is not is judgemental: it will happily mangle a recording if the settings are wrong, and nobody will warn you.

### 3. A hosted web service

If you record occasionally rather than daily, installing and tuning a chain is poor value. A hosted service is the pragmatic option: upload the file, compare the result, download it.

[Denoise Audio](https://denoiseaudio.com/) is built for exactly this case. You upload an MP3, WAV or M4A file up to 100 MiB and five minutes long, listen to a free preview of up to 30 seconds, and pay once — currently US$2.99 per recording — only if the preview convinces you. There is no account and no subscription, and the output is available as MP3 or WAV. The [pricing page](https://denoiseaudio.com/pricing) lists the current terms. If you process audio professionally, a local chain will be cheaper per hour; if you need one interview cleaned tonight, the hosted route is faster.

## How to judge the result before you publish

Two habits separate people who ship clean audio from people who ship damaged audio.

**Always listen to a genuine A/B at the same playback position.** Not the intro versus the outro — the same three seconds, switched back and forth. The right amount of reduction is usually less than you expect. If the enhanced version sounds thinner, sit further back, or has a slight flutter on sustained vowels, you have gone too far.

**Listen for the specific artefacts.** Musical noise sounds like random short tones or tiny bells, usually audible in the gaps between words. Over-suppression sounds like a voice that ducks and recovers around each syllable. Model artefacts sound like smoothed, slightly robotic consonants, especially sibilants. None of these are fixable after the fact, so check before delivery.

If you want an objective number as well as an opinion, the [ITU-T P.835](https://www.itu.int/rec/T-REC-P.835/en) framework is the reference for rating speech and background noise separately, and models such as [DNSMOS P.835](https://arxiv.org/abs/2110.01763) predict those three scores without needing a clean reference. That is useful for comparing settings across a hundred files; it is not a replacement for listening to the one file you are about to publish.

## A repeatable workflow

For spoken-word material, this sequence is reliable:

1. Archive the original recording untouched.
2. Apply click removal and notch filtering for hum, if needed.
3. Denoise, using the gentlest setting that makes the noise inaudible in the gaps.
4. High-pass filter around 80 Hz to remove rumble below the fundamental of the voice.
5. Compress lightly, then normalise to your delivery target.
6. Listen once on speakers and once on headphones, at a realistic volume.

The order matters: noise reduction before compression, because compression raises the noise floor along with the voice, and high-pass filtering after denoising, because low-frequency rumble confuses the noise estimator.

## Privacy and file handling

Cloud processing means someone else's server holds your recording, so read the retention terms before uploading anything confidential. A clear policy states when files expire. Denoise Audio, for example, deletes unpurchased uploads 24 hours after upload and paid results 24 hours after they become ready, and it does not require an account to download. If your material is under NDA, a local FFmpeg or DAW chain keeps it on your own disk.

## The short version

Denoise early, denoise gently, and judge the result by listening rather than by looking at a waveform. Fix the recording setup first, because no algorithm recovers detail that was never captured. Then pick the tool that matches your volume of work: a DAW for control, FFmpeg for batches, and a hosted service such as [Denoise Audio](https://denoiseaudio.com/) when you need one file cleaned properly without building a pipeline for it. Preview before you pay, compare at the same position, and keep the original.
