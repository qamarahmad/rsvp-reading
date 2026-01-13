# RSVP Reader

A Svelte-based Rapid Serial Visual Presentation (RSVP) reader for speed reading.

## What is RSVP?

Rapid Serial Visual Presentation (RSVP) is a technique where text is displayed one word at a time at a fixed focal point. This eliminates the need for eye movements (saccades) during reading, potentially allowing for significantly faster reading speeds.

The app uses **Optimal Recognition Point (ORP)** highlighting - the red letter in each word indicates the point where your eye naturally focuses for fastest word recognition. This is calculated based on word length:

- 1-3 letter words: 1st letter
- 4-5 letter words: 2nd letter
- 6-9 letter words: 3rd letter
- 10+ letter words: 4th letter

## Features

- **Adjustable reading speed**: 50-1000 words per minute (WPM)
- **ORP highlighting**: Red-highlighted focal letter for faster recognition
- **Fade effect**: Optional smooth transitions between words
- **Punctuation pauses**: Configurable extra pause on sentence-ending punctuation
- **Periodic pauses**: Optional pause every N words for comprehension
- **Progress tracking**: Visual progress bar and time remaining
- **Keyboard shortcuts**: Full keyboard control for hands-free reading
- **Dark theme**: Easy on the eyes with black background

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/rsvp.git
cd rsvp

# Install dependencies
npm install

# Start development server
npm run dev
```

## Usage

### Running the App

```bash
# Development mode with hot reload
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Controls

**Buttons:**
- **Play**: Start reading from the beginning or current position
- **Pause**: Pause reading
- **Resume**: Continue from where you paused
- **Stop**: Stop and reset to beginning
- **Restart**: Stop and immediately start from beginning

**Keyboard Shortcuts:**
| Key | Action |
|-----|--------|
| `Space` | Play/Pause/Resume |
| `Escape` | Stop |
| `Arrow Up` | Increase speed (+25 WPM) |
| `Arrow Down` | Decrease speed (-25 WPM) |
| `Arrow Left` | Go back one word |
| `Arrow Right` | Skip forward one word |

### Settings

Click the gear icon to access settings:

- **Words Per Minute**: Reading speed (50-1000 WPM)
- **Enable Fade Effect**: Smooth fade transition between words
- **Fade Duration**: Duration of fade effect (50-300ms)
- **Pause on Punctuation**: Extra pause at sentence endings
- **Punctuation Pause Multiplier**: How much longer to pause (1-4x)
- **Pause Every N Words**: Take a break every N words (0 = disabled)
- **Pause Duration**: Length of periodic pauses (100-2000ms)

### Setting Custom Text

1. Click the pencil icon in the header
2. Paste or type your text in the textarea
3. Click "Apply" to load the text
4. Click "Play" to start reading

## Project Structure

```
rsvp/
├── src/
│   ├── App.svelte          # Main application component
│   ├── app.css             # Global styles
│   ├── main.js             # Application entry point
│   ├── lib/
│   │   └── rsvp-utils.js   # Core RSVP utility functions
│   └── tests/
│       ├── setup.js        # Test setup
│       └── rsvp-utils.test.js  # Unit tests
├── index.html
├── package.json
├── vite.config.js
└── README.md
```

## Testing

```bash
# Run tests in watch mode
npm test

# Run tests once
npm run test:run

# Run tests with coverage
npm run test:coverage
```

## API Reference

### Utility Functions (`src/lib/rsvp-utils.js`)

#### `parseText(text)`
Parses input text into an array of words.

```javascript
parseText('Hello world') // ['Hello', 'world']
```

#### `getORPIndex(word)`
Calculates the Optimal Recognition Point index for a word.

```javascript
getORPIndex('hello') // 1 (second letter 'e')
```

#### `getActualORPIndex(word)`
Gets the actual character index for ORP, accounting for leading punctuation.

```javascript
getActualORPIndex('"hello') // 2 (skips the quote)
```

#### `getWordDelay(word, wpm, pauseOnPunctuation, multiplier)`
Calculates the display delay for a word based on WPM and punctuation.

```javascript
getWordDelay('hello', 300) // 200 (ms)
getWordDelay('end.', 300, true, 2) // 400 (ms)
```

#### `formatTimeRemaining(remainingWords, wpm)`
Formats remaining time as MM:SS.

```javascript
formatTimeRemaining(300, 300) // '1:00'
```

#### `splitWordForDisplay(word)`
Splits a word into parts for ORP display.

```javascript
splitWordForDisplay('hello')
// { before: 'h', orp: 'e', after: 'llo' }
```

#### `shouldPauseAtWord(wordIndex, pauseAfterWords)`
Checks if reading should pause at the current word.

```javascript
shouldPauseAtWord(10, 10) // true
shouldPauseAtWord(5, 10)  // false
```

## Browser Support

Works in all modern browsers:
- Chrome (recommended)
- Firefox
- Safari
- Edge

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is open source and available under the [MIT License](LICENSE).

## Acknowledgments

- Based on RSVP research in cognitive psychology
- Inspired by various speed reading applications
- Built with [Svelte](https://svelte.dev/) and [Vite](https://vitejs.dev/)
