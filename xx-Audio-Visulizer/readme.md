# Audio Visualizer

In this example you will make a visualization of audio data. This will not use D3 but I will connect it to D3 in a lter tutorial. The concepts here will just cover some of the ideas that will be needed to get it working with D3. 

This a real time visualization. That is the image will update in real time as data changes in the moment. 

## Audio 

There are some special considerations to understand when working with audio. 

You can't play audio without user interaction. Seriously. read this: 

https://developer.chrome.com/blog/autoplay/

tl;dr Audio is annoying and often abused. Browsers do not allow audio to play unless the user has initiated it. 

### Playing Audio

Create a button that plays audio. 

You can play audio in a couple ways. HTML 5 provides the `<audio>` tag. For this example will you will use the `Audio` object. 

Create a new HTML document. 

Add a Play and Stop button to the body. Give these buttons some id names so you can easily refernce them with JS: 

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

Clicking either button should call the the `playAudio` or `stopAudio` functions. 

### Audio Object 

Here you will setup the `Audio` object and create an `AudioContext`. Working with Audio is a lot like working with Canvas. You will create an Audio object, this is your sound canvas. Then you'll create an Audio context, this contains all of the tools to control the audio object. 

Find an audio file to use or copy the example file from this tutorial's audio folder. 

`bird-whistling-a.wav`

Play the audio with: 

```JS
let audio // declare this outside!

function playAudio() {
  audio = new Audio()
  audio.src = './audio/bird-whistling-a.wav'
  audio.play()
}

function stopAudio() {
  audio.pause()
}
```

At this stage clicking the play button should play the audio file. Clicking stop should stop the audio. 

Notice you declared the `audio` variable outside the functions so it can be used in both functions. 

### Analysing the audio

JavaScript allows you to analyze your audio. This process provides a data array that describes the volume of the sound in a number of frequency bands. Audio data is a series of 8 bit values. 

Analyzing audio needs to work efficiently because of the rate at which data needs to be analyzed. You are listening to audio is real time. To create a visualization that visually displays the data in real time you will be udating the visualization 30 to 60 times per second. This requires looking at a lot of data in a short amount of time. 

To make this more effecient the audio analyzer stores data in bytes. A byte is 8 bits of information that represents a value from 0 to 255. 

This is more effecient because knowing the size of the data in advance allows the computer to store access it more efficiently. 

The analyzer returns an array of data but does this as a `Uint8Array`. This is an array of fixed size that stores only 8 bit unsigned integers. So its a special array that stores integers in the range of 0 to 255. 

Read more about `Uint8Array` here: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array

Add a couple variables to your project: 

```JS
let audio
let frequencyArray
let isPlaying = false
let analyser
```

These will be outside of the functions you have created since they need to be accessed across several of those functions. 

```JS
function playAudio() {
  audio = new Audio()
  const audioContext = new AudioContext() // Add this before assigning source
  audio.src = './audio/bird-whistling-a.wav'
  // Setup the analyser
  analyser = audioContext.createAnalyser()
  analyser.fftSize = 32
  const source = audioContext.createMediaElementSource(audio)
  source.connect(analyser)
  source.connect(audioContext.destination);
  audio.play()
  isPlaying = true
  requestAnimationFrame(renderAudio)
}
```

Here you created an audio context. You'll use this to connect your analyser to the audio object. Create this before setting the audio source. 

Next you creatd the analyser and set the fft size. This will set the number of bands the audio data will be divided into. The value has to be a power of 2 and between 2^5 and 2^15. So 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, and 32768 are the only choices. 




