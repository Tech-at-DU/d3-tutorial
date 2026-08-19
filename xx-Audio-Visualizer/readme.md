# Audio Visualizer

**This one's optional.** It's not part of the core path or the chart-type branches — it's here as a bonus project for anyone who wants a different angle on "data visualization." Every other tutorial in this repo visualizes data that already exists in a file. This one visualizes data as it's *generated*, live, dozens of times a second, from sound playing through your speakers. Same underlying skills (bind data to shapes, scale it, draw it), very different feel.

Most of this tutorial won't touch D3 at all — the Web Audio API has its own vocabulary to learn first. Once you have real audio data flowing as plain JavaScript arrays, you'll connect it to D3 at the end using the same scales, bars, and join pattern from tutorials 09 and 19.

This is a real time visualization. The image updates continuously as audio plays, not once after data loads.

## Audio

There are some special considerations to understand when working with audio.

You can't play audio without user interaction. Seriously, read this: 

https://developer.chrome.com/blog/autoplay/

tl;dr Audio is annoying and often abused. Browsers do not allow audio to play unless the user has initiated it.

### Playing Audio

Create a button that plays audio.

You can play audio in a couple ways. HTML5 provides the `<audio>` tag. For this example you will use the `Audio` object instead.

Create a new HTML document.

Add a Play and Stop button to the body. Give these buttons some id names so you can easily reference them with JS:

```HTML
<button id="button-play">Play</button>
<button id="button-stop">Stop</button>
```

Add a script tag to your document and create some variables to reference the button elements you created:

```HTML
<script>
  const playButton = document.getElementById('button-play')
  const stopButton = document.getElementById('button-stop')
</script>
```

Add a couple event listeners that will handle click events on these buttons:

```JS
playButton.addEventListener('click', (e) => {
  playAudio()
})

stopButton.addEventListener('click', (e) => {
  stopAudio()
})

function playAudio() {

}

function stopAudio() {

}
```

Clicking either button should call the `playAudio` or `stopAudio` functions.

### Audio Object

Here you will set up the `Audio` object and create an `AudioContext`. Working with Audio is a lot like working with Canvas. You create an Audio object — think of it as your sound canvas. Then you create an Audio context, which contains all of the tools to control the audio object.

Find the `audio` folder in this repo (same place as the `data` folder used in earlier tutorials) and copy `bird-whistling-a.wav` into the directory where you are working.

**Watch out!** If you'd rather use your own audio file, keep it *local* to your project instead of linking to some other website's audio. The next section is going to analyze this audio's waveform, and browsers block that analysis on cross-origin audio unless the remote server explicitly opts in with CORS headers. A file sitting in your own project folder sidesteps the whole problem.

Play the audio with:

```JS
let audio // declare this outside!

function playAudio() {
  audio = new Audio()
  audio.src = './bird-whistling-a.wav'
  audio.play()
}

function stopAudio() {
  audio.pause()
}
```

At this stage clicking the play button should play the audio file. Clicking stop should stop the audio.

Notice you declared the `audio` variable outside the functions so it can be used in both functions.

### Analyzing the audio

JavaScript allows you to analyze your audio. This process provides a data array that describes the volume of the sound in a number of frequency bands. Audio data is a series of 8-bit values.

Analyzing audio needs to work efficiently because of the rate at which data needs to be analyzed. You're listening to audio in real time. To create a visualization that visually displays the data in real time you'll be updating the visualization 30 to 60 times per second. This requires looking at a lot of data in a short amount of time.

To make this more efficient the audio analyzer stores data in bytes. A byte is 8 bits of information that represents a value from 0 to 255.

This is more efficient because knowing the size of the data in advance allows the computer to store and access it more efficiently.

The analyzer returns an array of data, but does this as a `Uint8Array`. This is an array of fixed size that stores only 8-bit unsigned integers — a special array that stores integers in the range of 0 to 255.

Read more about `Uint8Array` here: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array

Add a couple variables to your project:

```JS
let audio
let frequencyArray
let isPlaying = false
let analyser

const FFT_SIZE = 32 // must be a power of 2 between 32 and 32768
```

These will be outside of the functions you have created since they need to be accessed across several of those functions.

```JS
function playAudio() {
  audio = new Audio()
  const audioContext = new AudioContext() // Add this before assigning source
  audio.src = './bird-whistling-a.wav'
  // Setup the analyser
  analyser = audioContext.createAnalyser()
  analyser.fftSize = FFT_SIZE
  const source = audioContext.createMediaElementSource(audio)
  source.connect(analyser)
  source.connect(audioContext.destination)
  audio.play()
  isPlaying = true
  requestAnimationFrame(renderAudio)
}
```

Here you created an audio context. You'll use this to connect your analyser to the audio object. Create this before setting the audio source.

Next you created the analyser and set the fft size. This sets the number of bands the audio data will be divided into. The value has to be a power of 2 between 2^5 and 2^15. So 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, and 32768 are the only choices.

`source.connect(analyser)` sends the audio's data to the analyser so it can be inspected. `source.connect(audioContext.destination)` sends the audio straight to your speakers. Notice these are two separate connections — the analyser doesn't need to also connect to the destination, because the source is already reaching your speakers through the second line. The analyser is just eavesdropping on a copy of the signal, not sitting in the middle of it.

### Reading the frequency data

`analyser.fftSize` controls how many numbers come out the other end, but not directly — the number of usable values is always **half** the fft size. This is called `frequencyBinCount`, and D3-side code will need it, so it's worth understanding: with `fftSize = 32`, `analyser.frequencyBinCount` is `16`. Each of those 16 numbers represents the volume of one frequency band, as a byte from 0 (silent) to 255 (loudest).

Create an array to hold that data. Add this inside `playAudio()`, after the analyser is created:

```JS
frequencyArray = new Uint8Array(analyser.frequencyBinCount)
```

Now define `renderAudio()` — the function `requestAnimationFrame` is already calling, from the line at the bottom of `playAudio()`.

```JS
function renderAudio() {
  analyser.getByteFrequencyData(frequencyArray) // fills frequencyArray with current data
  console.log(frequencyArray)

  if (isPlaying) {
    requestAnimationFrame(renderAudio) // do it again next frame
  }
}
```

`getByteFrequencyData()` doesn't return anything — it fills the array you give it with the current volume of each frequency band, right now, at this exact instant. That's why `frequencyArray` was created once, ahead of time, and passed in by reference: every call to `renderAudio()` overwrites the same array with fresh values.

`requestAnimationFrame(renderAudio)` schedules `renderAudio` to run again just before the browser's next repaint — usually 60 times a second. Calling it again at the *end* of `renderAudio()`, only `if (isPlaying)`, is what creates a self-sustaining loop that keeps running every frame until you stop it.

Update `stopAudio()` to actually break that loop:

```JS
function stopAudio() {
  audio.pause()
  isPlaying = false
}
```

Without this, the animation loop would keep calling `getByteFrequencyData()` forever, even after the audio stopped.

Test it. Click play, watch the console. You should see a stream of `Uint8Array(16)` values, changing continuously while audio plays, all zeros once it stops.

**Challenge:** Try changing `FFT_SIZE` to `128` and reload. Compare the size of the array in the console — more bins means more detail in the visualization, at the cost of more values to process every frame.

## Visualizing the data with D3

Now the payoff. You have 16 numbers, updating 60 times a second, each between 0 and 255. That's exactly the shape of data tutorial 09 turned into a bar chart — except this bar chart needs to redraw every single frame instead of once.

Import D3, same as every other tutorial:

```HTML
<script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
```

Add an SVG element to your page:

```HTML
<svg id="svg" width="600" height="300"></svg>
```

Set up scales. Since `FFT_SIZE` is a fixed constant, you know the number of bars ahead of time — no need to wait for the analyser to exist first:

```JS
const width = 600
const height = 300

const xscale = d3.scaleBand()
  .domain(d3.range(FFT_SIZE / 2)) // one band per frequency bin
  .range([0, width])
  .padding(0.1)

const yscale = d3.scaleLinear()
  .domain([0, 255]) // byte values are always 0-255
  .range([0, height])
```

`d3.range(FFT_SIZE / 2)` generates `[0, 1, 2, ..., 15]` — a plain index for each bin, since the bins themselves don't have any more meaningful label than "which band is this."

Now draw (and redraw) the bars. This uses `.join()` from tutorial 19, since this function will run every single frame:

```JS
function drawBars() {
  d3.select('#svg')
    .selectAll('rect')
    .data(Array.from(frequencyArray))
    .join('rect')
    .attr('x', (d, i) => xscale(i))
    .attr('y', d => height - yscale(d))
    .attr('width', xscale.bandwidth())
    .attr('height', d => yscale(d))
    .attr('fill', 'cornflowerblue')
}
```

`Array.from(frequencyArray)` converts the `Uint8Array` into a regular array — `.data()` expects that. Everything else here is the exact bar-drawing code from tutorial 09: `xscale`/`bandwidth()` for horizontal position and width, `yscale` for height, flipped so bars grow up from the bottom.

Call `drawBars()` from inside `renderAudio()`, right after reading the fresh data:

```JS
function renderAudio() {
  analyser.getByteFrequencyData(frequencyArray)
  drawBars()

  if (isPlaying) {
    requestAnimationFrame(renderAudio)
  }
}
```

Click play. You should have 16 bars jumping up and down in time with the audio.

## Challenges

**Challenge:** Give each bar a color based on its own volume, using an idea from tutorial 10 — try `d3.scaleSequential(d3.interpolateInferno).domain([0, 255])` and set fill with `d => colorScale(d)`.

**Challenge:** Try `analyser.getByteTimeDomainData()` instead of `getByteFrequencyData()`. It fills the same kind of array, but with waveform data (amplitude over time) instead of frequency data. The values center around 128 instead of starting at 0 — you'll need to adjust your `yscale` domain accordingly. Read more: https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode/getByteTimeDomainData

**Stretch Challenge:** Bars are the simplest shape here — try circles arranged in a radial layout instead, using the trig ideas from tutorial 05.

## Check Your Understanding

**Q1.** Why does `frequencyArray` get created once, before the animation loop starts, instead of a fresh array being created inside `renderAudio()` on every frame?

<details><summary>Answer</summary>

`getByteFrequencyData()` doesn't return a new array — it writes into the array you hand it. Reusing the same `Uint8Array` avoids allocating a brand-new array 60 times a second, which matters here specifically because this code runs constantly, unlike the one-time data loads in every other tutorial.

</details>

**Q2.** Why does `requestAnimationFrame(renderAudio)` get called from *inside* `renderAudio()` itself, rather than with `setInterval()`?

<details><summary>Answer</summary>

`requestAnimationFrame` syncs its callback to the browser's actual repaint cycle, so the animation stays smooth and pauses automatically when the tab isn't visible (saving battery/CPU) — `setInterval()` has no awareness of rendering and will happily keep firing in a background tab. Calling it again from inside its own callback, guarded by `if (isPlaying)`, is the standard way to build a loop that can be cleanly stopped.

</details>

**Q3.** This tutorial calls `drawBars()` using `.join('rect')` rather than `.enter().append('rect')`. Why does that distinction matter so much more here than it did in, say, tutorial 09?

<details><summary>Answer</summary>

In tutorial 09, the bars were drawn exactly once — `.enter().append()` was all that was ever needed. Here, `drawBars()` runs 60 times a second against the *same 16 rects* over and over. `.enter().append()` alone would only handle the very first frame correctly; every frame after that, those 16 rects already exist, so `.join()`'s update behavior (reusing and re-attributing existing elements instead of piling up new ones) is what makes the loop work at all.

</details>

## Conclusion

In this tutorial you worked with the Web Audio API to analyze sound in real time, then fed that stream of live data into the same scale-and-bar drawing pattern from tutorial 09, updated every frame using the join pattern from tutorial 19. The data source was completely different from anything else in this repo — generated continuously instead of loaded once from a file — but the D3 skills needed to visualize it were exactly the ones you already had.
