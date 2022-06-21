# D3 Join API

When working with D3 you may have noticed that elements will be created, updated, and removed from the DOM. D3 does automatically. The Join API gives you control over the elements as they are created, updated and removed from the DOM. 

Why is this important? Understanding the Join API allows you to with D3. Especially for things that apply to interactive charts. 

## Getting started 

Start with a new HTML document. 

Import D3. 

For this example you will need an SVG element and a couple buttons. Add these to the body tag: 

```HTML
<svg id="svg" width="600" height="300"></svg>
<button id="add-button">Add</button>
<button id="remove-button">Remove</button>
```

Put this all in the center of the page with some CSS styles: 

```HTML
<style>
  /* Put the box in the center of the page */
  body,
  html {
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  #svg {
    border: 1px solid;
  }
</style>
```

This should place everything on the page in a center column and add a border around the `#svg` element.

Add a script tag: 

```HTML
<script>
  // Your code here! 
</script>
```

Add a few variables for the width, height, and margins:

```JS
const margin = { top: 20, right: 40, bottom: 20, left: 100 }
const width = 600 - (margin.left + margin.right)
const height = 300 - (margin.top + margin.bottom)
```

## Add some data

The goal of this example will be to display a bar graph and and remove bars in the graph by clicking the buttons. 

The data for this example will be an array of numbers from 0 to 100. 

Define your data: 

```JS
const data = [50]
```

To start your bar chart will have a single bar with a value of 50. 

Add a couple handlers. These will handle clicks on the buttons by adding and removing values from the array: 

```JS
document.querySelector('#add-button')
  .addEventListener('click', e => {
    const n = Math.round(Math.random() * 100)
    data.push(n)
    drawBars()
  })

document.querySelector('#remove-button')
  .addEventListener('click', e => {
    data.pop()
    drawBars()
  })
```

Here I used vanilla JS we couold have used D3 to handle these click events also. 

Notice there is a call to `drawBars()` which is function you'll define in the future!

### Setting up the scales

You need to create a scale for the horizontal and vertical axis. 

For the horizontal scale use a scale band. The domain will be the array of data and the range will calculated from the margin and width of the chart: 

```JS
const xscale = d3.scaleBand()
  .domain(data)
  .range([margin.left, width + margin.left])
  .padding(0.1)
```

Define a vertical scale using scale linear. This scale will have a domain of 0 to 100 and a range based on the height and margins. 

```JS
const yscale = d3.scaleLinear()
  .domain([0, 100]) // range 0 - 100
  .range([height, margin.top])
  .nice()
```

Select the SVG element and create a group to hold the bars. 

```JS
const svg = d3.select('#svg')
const barGroup = svg.append('g')
```

The bars will rectangles, you need to make a selction of these that the chart can work with: 

```JS
const bars = barGroup
  .selectAll('rect')
```

You are storing this reference in a variable so you can work with it as the data changes. 

Now make an axis for the bottom of the chart: 

```JS
const bottomAxis = d3.axisBottom()
  .scale(xscale)
  .ticks(data.length)
```

Now draw the bottom axis: 

```JS
svg
  .append('g')
  .attr('class', 'botton-axis')
  .attr('transform', `translate(${0}, ${height + 5})`)
  .call(bottomAxis)
```

You included a class name here because the bottom axis will need to update when data is added to the array. 

Now draw the left axis: 

```JS
const leftAxis = d3.axisLeft(yscale)
svg
  .append('g')
  .attr('transform', `translate(65, 0)`)
  .call(leftAxis)
```

So far you won't be seeing any bars but you should see the axese. 

### Drawing bars

Earlier when you wrote the handlers for the buttons you included a call to `drawBars()`. You may have noticed that this function had not been defined yet. You will define this function now! 

```JS
function drawBars() {
  // Update x scale 
  
  // Make bars
  
  // Update x axis
  
}

drawBars()
```

The `drawBars()` function will draw the bars in the bar chart and update the horizontal scale. Here the function is stubbed. 

Notice that you are calling the function once at the bottom to initialize the chart. You are also calling this function when the "Add" and "Remove" buttons are clicked. 

Since the x scale will change when new data is added to the data array you need to update the x scale when you redraw the bars. 

```JS
function drawBars() {
  // update x scale 
  xscale
    .domain(data)
    .range([margin.left, width + margin.left])
    .padding(0.1)

  // Make bars
  
  // update x axis
  
}
```

Using scale band the scale is based on a list of values, since you're changing the number of values you need update the domain of the x scale. 

Now make some bars: 

```JS
function drawBars() {
  // update x scale 
  xscale
    .domain(data)
    .range([margin.left, width + margin.left])
    .padding(0.1)

  // Make bars
  barGroup
    .selectAll('rect')
    .data(data)
    .join('rect')
    .attr('x', xscale)
    .attr('y', d => yscale(d))
    .attr('width', xscale.bandwidth())
    .attr('height', d => height - yscale(d))

  // update x axis
  
}
```

Here you selected all rectangles in the `barsGroup`, assigned your data, then set the `x`, `y`, `width`, and `height` attributes to draw the bars. 

The `x` is set by the xscale , `y` is set via yscale. The `width` is set by the `xscale` with bandwidth. 

Note! Since the `rect` draws from the upper left corner you calculated the height as the height of the chart - the data value through the yscale function. 

Last you need to update the axis. 

```JS
function drawBars() {
  // update x scale 
  xscale
    .domain(data)
    .range([margin.left, width + margin.left])
    .padding(0.1)

  // Make bars
  barGroup
    .selectAll('rect')
    .data(data)
    .join('rect')
    .attr('x', xscale)
    .attr('y', d => yscale(d))
    .attr('width', xscale.bandwidth())
    .attr('height', d => height - yscale(d))

  // update x axis
  bottomAxis.scale(xscale)

  svg.selectAll('.botton-axis')
    .transition()
    .duration(400)
    .call(bottomAxis)
  
}
```

Great work! 

So far you should have a single bar and the two axese. Clicking the buttons should add and remove bars from the chart. The data is random so the bars will be different each time you add a new one. 

## D3 Join API

I know it took us a while to get here and all we have is something not too far from some of the previous tutorials. Here is where you will start looking at the join API.

The join API provides options to determine what happens to elements that exist and are: being updated, have been newly added, and are being removed from the DOM. 

When you create a D3 selection D3 makes use of any existing elements that it finds in the DOM. 

If there are no elements or not enough elements D3 creates new elements. 

If there are too many elements D3 removes the extra elements. 

D3 does these things automatically. Sometime you will want to control what happens to these elements. For example you might want to: 

- Style the newly added data element
- Animate animate entering, updating, and exiting elements. 
- You may want to animate elements differently depending on whether they are entering, updating, or exiting. 

Let's try these ideas with the current example. 

Read more about D3 join here: https://observablehq.com/@philippkoytek/d3-part-3-selection-join-explained

### Using the Join API

The `join()` method takes three arguments, each is a callback.

```JS
d3.select('something')
  .join(() => {}, () => {}, () => {})
```

The first callback receives an entering selection, the second an updating selection, and the last gets the exiting selection. 

```JS
d3.select('something')
  .join(enter => {}, update => {}, exits => {})
```

From there you handle the three selections in standard D3 fashion. 

Let's try this out. Earlier you wrote: 

```JS
// Make bars
  barGroup
    .selectAll('rect')
    .data(data)
    .join('rect')
    .attr('x', xscale)
    .attr('y', d => yscale(d))
    .attr('width', xscale.bandwidth())
    .attr('height', d => height - yscale(d))
```

Here you used `.join()` without any arguments. This handles all of the entering, updating, and exiting elements the same. 

What if you wanted to color any new elements red while making the updating elements orange? 

Try this. Refactor this section of your `drawBars` function. 

```JS
// Make bars
barGroup
  .selectAll('rect')
  .data(data)
  // Start your join
  .join(enter => { // Handle entering data
    enter.append('rect') 
      .attr('x', xscale)
      .attr('y', d => yscale(d))
      .attr('width', xscale.bandwidth())
      .attr('height', d => height - yscale(d))
      .attr('fill', 'red')
  }, update => {
    update // handle updated data
      .attr('x', xscale)
      .attr('y', d => yscale(d))
      .attr('width', xscale.bandwidth())
      .attr('height', d => height - yscale(d))
      .attr('fill', 'orange')
  }, exit => {
    console.log('exit - ')
    exit // handle removed data 
      .remove()
  })
```

Here you defined a call back for each of the three datasets: entering, updating, and exiting, and defined what D3 should for each of these. 

For entering data you appended a new element. This is new data so you set the `x`, `y`, `width`, `height`, and `fill`. New elements will have a `fill` of red. 

For updating elements you set the same attributes but set the fill to orange. 

For exiting elements you just removed them. 

Test it out. The initial bar should be red since it was newly added.

Add a new bar and the first bar should become orange since it was updated. The newly added bar should be red. 

Removing a bar should leave you will only orange bars since a bar was removed and the remaining bars were updated! 

### Animating changes

Expand on this by animating the changes. Make new bars fade in:

```JS
enter.append('rect')
  .attr('x', xscale)
  .attr('y', d => yscale(d))
  .attr('width', xscale.bandwidth())
  .attr('height', d => height - yscale(d))
  .attr('fill', 'red')
  .style('opacity', 0) // set the opacity to 0
  .transition() // start a transition
  .duration(400) // set the duration to 400ms
  .style('opacity', 1) // animate the opacity to 1
```

Test this out. New elements should fade in. 

Note! you need to `opacity` with `.style()` instead of `.attr()`.

Note! We need to set the initial opacity to 0 then animate this to 1. 

Animate updated elements. You can animate all of the attributes this time:

```JS
update
  .transition() // start a transition
  .duration(400) // set the duration to 400ms
  .attr('x', xscale)
  .attr('y', d => yscale(d))
  .attr('width', xscale.bandwidth())
  .attr('height', d => height - yscale(d))
  .attr('fill', 'orange')
```

Testing you should updated elements fade from red to orange, and slide to the left or right. 

Animate exiting elements. Edit the exiting selection: 

```JS
exit
  .transition() // start a transition
  .duration(400) // set the duration to 400ms
  .attr('fill', 'gray') // set the fill color to gray
  .style('opacity', 0) // set the opacity to 0
  .remove() 
```

Now you should exiting element change to gray and fade out. Might be hard to see this as the transition is pretty short. Play with the timing to experiment. 

## Challenges

**Challenge:** Modify the transitions here to your taste. Think about all of the attributes you can animate. Any changes applied before `.transition()` will not be animated, while changes applied after `.transition()` will be animated. 

**Challenge:** Applied the join API to one of the previous tutorial examples.

## Conclusion

In this tutorial you explored the D3 join API and used it's enter, exit, and update pattern. This allowed you to handle elements as they enter, exit and are updaed in the DOM. 

