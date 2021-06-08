# zig-wav

A simple one-file Zig library for reading and writing WAV files (uncompressed
PCM format only). Does not provide any transcoding.

Requires Zig 0.8.0.

## API

Reading wav files:

```zig
var reader = // any std.io.Reader

// preload: load the format information (sample rate, etc) and leave the
// reader in position to start reading actual data.
const preloaded = try wav.preload(reader);

// you can inspect the preloaded info. supported formats are .unsigned8,
// .signed16_lsb, .signed24_lsb, and .signed32_lsb.
_ = preloaded.num_channels;
_ = preloaded.sample_rate;
_ = preloaded.format;
_ = preloaded.num_samples;

// now, you can read the audio data.
const num_bytes = preloaded.getNumBytes();

var buffer = try allocator.alloc(u8, num_bytes);
defer allocator.free(buffer);

if (do_it_yourself) {
    try reader.readNoEof(buffer);
} else {
    // this is just a wrapper around readNoEof
    try wav.load(reader, preloaded, buffer);
}
```

Writing wav files:

```zig
var writer = // any std.io.Writer

if (all_at_once) {
    // write the whole file at once
    try wav.save(writer, data, .{
        .num_channels = 1,
        .sample_rate = 44100,
        .format = .signed16_lsb,
    });
} else {
    // or if you want to stream out:
    try wav.writeHeader(writer, .{
        .num_channels = 1,
        .sample_rate = 44100,
        .format = .signed16_lsb,
    });
    // now write bytes as they become available.

    // if you have a SeekableStream with your Writer, you can now go back and
    // update the header:
    try patchHeader(writer, seekable_stream, num_bytes_written);
}
```
