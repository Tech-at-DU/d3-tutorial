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

At this stage clicking the audio button shouldplay the audio. Clicking stop should stop the audio. 

Notice you declared the `audio` variable outside the functions so it can be used in both functions. 

### Analysing the audio

JavaScript allows you to analyse your audio. This process provides a data array that describes the volume of the sound at the current moment in 255 frequency bands. 

