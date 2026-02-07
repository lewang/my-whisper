# Pr-Whisper - Speech-to-Text for Emacs

A simple Emacs package that provides speech-to-text functionality using Whisper.cpp.

Record audio directly from Emacs and have it transcribed and inserted in current buffer at point.



## Demo Video

[![Perfect Speech to Text in Emacs](https://img.youtube.com/vi/bTU9ctFtyBA/0.jpg)](https://youtu.be/bTU9ctFtyBA)

**[Perfect Speech to Text in Emacs](https://youtu.be/bTU9ctFtyBA)** - See the package in action!

## Features

- **Simple toggle interface**: Single key (`C-c .`) to start/stop recording
- **Model selection**: Choose from multiple Whisper models via customization
- **Vocabulary hints**: Provide a custom vocabulary file to improve recognition of proper nouns and specialized
  terms (e.g., Greek names like Socrates, Alcibiades, Diotima)
- **Transcription history**: Browse and re-insert previous transcriptions with `M-x pr-whisper-insert-from-history`
- Automatic transcription using Whisper.cpp
- Text insertion at cursor position
- Async processing - Emacs remains responsive during transcription

## Prerequisites

Before setting up this package, you need to install the following system dependencies:

### 1. Sox (for audio recording)

**Ubuntu/Debian:**
```bash
sudo apt install sox
```

**macOS:**
```bash
brew install sox
```

**Arch Linux:**
```bash
sudo pacman -S sox
```

### 2. Whisper.cpp

Clone and build Whisper.cpp:

```bash
# Clone the repository
git clone https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp

# Build the project
make

# Download models
# For fast mode (required)
bash ./models/download-ggml-model.sh base.en

# For accurate mode (required)
bash ./models/download-ggml-model.sh medium.en
```

## Setup

### With use-package

```elisp
(use-package pr-whisper
  :bind ("C-c ." . pr-whisper-toggle-recording))
```

### With global-set-key

```elisp
(require 'pr-whisper)
(global-set-key (kbd "C-c .") #'pr-whisper-toggle-recording)
```

## Usage

1. Position your cursor where you want the transcribed text
2. Press `C-c .` to start recording
3. Speak into your microphone
4. Press `C-c .` again to stop recording and transcribe
5. The text appears at your cursor position

## Configuration

You can customize pr-whisper through Emacs' built-in customization interface or directly in your `init.el`.

### Using Emacs Customize Interface

Run `M-x customize-group RET pr-whisper RET` to access all customization options:

- **pr-whisper-homedir**: Directory where Whisper.cpp is installed (default: `~/whisper.cpp/`)
- **pr-whisper-model**: Which model to use by default
  - `ggml-base.en.bin` - Fast mode (quick, good accuracy)
  - `ggml-medium.en.bin` - Accurate mode (slower, better accuracy)
  - Custom model filename
- **pr-whisper-vocabulary-file**: Path to vocabulary hints file (default: `~/.emacs.d/whisper-vocabulary.txt`)
- **pr-whisper-backend**: Transcription backend - `cli` (default) or `server`
- **pr-whisper-server-port**: Port for whisper-server (default: 8178)

### Custom Configuration in init.el

```elisp
;; Set custom Whisper.cpp installation directory
(setq pr-whisper-homedir "/usr/local/whisper.cpp/")

;; Choose default model (base.en or medium.en)
(setq pr-whisper-model "ggml-medium.en.bin")

;; Set custom vocabulary file location
(setq pr-whisper-vocabulary-file "~/Documents/vocabulary.txt")

;; Use server backend for faster transcription (~29% speedup)
;; Server starts during recording and warms up while you speak
(setq pr-whisper-backend 'server)
```

### Custom Vocabulary for Proper Nouns

To improve transcription accuracy for proper nouns, technical terms, or specialized vocabulary, create a vocabulary file at `~/.emacs.d/whisper-vocabulary.txt`.

**Example `~/.emacs.d/whisper-vocabulary.txt`:**
```
This transcription discusses classical Greek philosophy, including scholars and figures such as Thrasymachus, Socrates, Plato, Diotima, Alcibiades, and Phaedrus.
```

**Custom vocabulary location:**
```elisp
(setq pr-whisper-vocabulary-file "~/Documents/vocabulary.txt")
```

**For detailed guidance** on vocabulary formats, tips, domain-specific examples, and managing multiple vocabularies,
see [VOCABULARY-GUIDE.md](VOCABULARY-GUIDE.md).

### Recording Indicator (Mode Line)

When recording starts, a flashing red `● REC` appears in the mode line of the buffer that initiated
recording. The indicator is automatically added to `mode-line-format`.

Customize the faces `pr-whisper-recording-bright` and `pr-whisper-recording-dim` to change colors. The
flash speed is controlled by `pr-whisper-flash-interval` (default 0.5 seconds).

## Troubleshooting

### Common Issues

1. **"sox: command not found"**
   - Install sox using your system package manager

2. **"whisper-cli: command not found" or path errors**
   - Ensure Whisper.cpp is built and the path is correct
   - Check that `~/whisper.cpp/build/bin/whisper-cli` exists
   - For server backend, check that `~/whisper.cpp/build/bin/whisper-server` exists
   - If installed elsewhere, customize `pr-whisper-homedir` to match your installation
   - Run `M-x customize-group RET pr-whisper RET` to verify paths

3. **No audio recorded**
   - Check your microphone permissions
   - Test sox manually: `sox -d -r 16000 -c 1 -b 16 test.wav`

4. **Transcription not working**
   - The package now validates paths on startup and will show clear error messages
   - Verify the model files exist in `~/whisper.cpp/models/`
   - Run `M-x pr-whisper-toggle-recording` to see validation errors
   - Test whisper-cli manually with a wav file

### Testing the Setup

Test each component individually:

```bash
# Test sox recording (record 5 seconds)
sox -d -r 16000 -c 1 -b 16 test.wav trim 0 5

# Test whisper transcription (fast mode)
~/whisper.cpp/build/bin/whisper-cli -m ~/whisper.cpp/models/ggml-base.en.bin -f test.wav

# Test whisper transcription (accurate mode)
~/whisper.cpp/build/bin/whisper-cli -m ~/whisper.cpp/models/ggml-medium.en.bin -f test.wav
```

## How It Works

1. **Recording**: Uses `sox` to record audio at 16kHz, mono, 16-bit
2. **Processing**: Calls `whisper-cli` with the recorded audio file
3. **Integration**: Captures the output and inserts it into your Emacs buffer
4. **Cleanup**: Automatically cleans up temporary files and buffers

## License

This project is released under the MIT License.

## Development

### Running Tests

```bash
emacs -Q --batch -L . -l pr-whisper-test.el -f ert-run-tests-batch-and-exit
```

### Byte Compilation

```bash
./build
```

## Contributing

Feel free to submit issues and pull requests to improve this package.
