# D3 Tutorial

This tutorial will get you started learning D3. D3 is deep! This tutorial covers the fundamental concepts. Later tutorials will delve into features and systems in D3. 

## Getting started

D3 is written with JavaScript and runs in your browser. Most often D3 projects will display in a web browser. 

Start by making a new web page. create: `index.html`. Add the boilerplate HTML code to your page:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
	
</body>
</html>
```

To use D3.js you need to link to the library. You can download it from the website or link to it from a CDN. We'll choose the second method. 

Link to the library from the CDN. Add the script tag to the bottom of the body tag: 

```HTML
<body>

  <script src="https://d3js.org/d3.v7.min.js"></script>
  <script>
    // Your script here! 
  </script>
</body>
```

You can find this link on the [D3 hompage](https://d3js.org)

It might be easier to find some info on their GitHub: https://github.com/d3/d3.

### Displaying elements

Select an element to work with. You will be adding or modifying elements within the element we select here. Here we are selecting the `body`:

```JS
d3.select('body')
```

Next, select the elements you want to modify or create. In this case, we are selecting divs.

```JS
d3.select('body')
  .selectAll('div')
```

*It doesn't matter that there are no existing divs.* They will exist later D3 will make them!

Next, add some data. Data can be an array containing any type of data. For this example, I've used an array of numbers. 

```JS
d3.select('body')
  .selectAll('div')
  .data([5,6,2,8,4,9,1])
```

Next call `enter()`. Calling enter tells D3 to "enter" the data. Imagine at this point D3 is looping over your data. 

```JS
d3.select('body')
  .selectAll('div')
  .data([5,6,2,8,4,9,1])
  .enter()
```

Use `append()` to tell D3 what type of element to append. 

```JS
d3.select('body')
  .selectAll('div')
  .data([5,6,2,8,4,9,1])
  .enter()
  .append('div')
```

Here D3 creates elements if they don't exist. We need one div for each of the values in our data. 

While we haven't drawn anything visible yet. There should be one div created for each number in the array. 

The code you have written so far is sort of boilerplate for D3 and is a starting place for most projects. 

### D3 Cheatsheet

Let's review what just happened. Here's the formula for getting started with D3. 

```JS
d3.select('body')        // select() an element to host 
  .selectAll('div')			 // selectAll() the elements you want to work with
  .data([5,6,2,8,4,9,1]) // define your data()
  .enter()               // enter() your data 
  .append('div')         // append() elements
```

- `d3.select('#someId')` - Select an element to host the elements you will create. This element will contain all of the elements d3 creates. This element should exist and you will probably want to identify it with an id. 
- `.selectAll('someElement')` - Select all elements you will be working with. These elements don't need to exist and will be created if they don't exist. 
- `.data(someArray)` - Add an array of data
- `enter()` - Begins looping over the data. 
- `.append('someEl')` - Appends elements to the DOM for each piece of data.

Time to draw something!

## D3 Working with elements and data

Set the text of each new node. This method takes a function that receives one of the data values and should return the text to display in the node.

```JS
d3.select('body')
  .selectAll('div')
  .data([5,6,2,8,4,9,1])
  .enter()
  .append('div')
  .text((d) => d)
```

What happened here? You started D3. Pointed D3 to the body tag to work with as a container for dynamically generated elements. 

You selected all of the div tags (there are none at the moment.) You gave D3 an array of numbers to work with. 

Next, you called `.enter()` this method identifies DOM elements that need to be added to a selection. Currently, there are no divs but you have 7 numbers in your array, so we need to `.append()` some new divs. 

Last, for each div, you set the text of that div. Here text takes a callback that receives a piece of data from the array and is expected to return the value that you'd like to see. 

The result is a list of divs each displaying one of the elements from the array. 

Try this, make the change below to format the numbers: 

```JS
d3.select('body')
  ...
  .text(d => `$${d.toFixed(2)}`)
```

Here you took the data element and converted it to a string, added a "$" and a decimal and some 0s. 

Stop a moment and imagine the piece of data (`d`) was a passenger object from the Titanic. 🤔 You could print the name or the fare...

### Styling Elements

The `style()` method takes two arguments. First is the style attribute to set, and second is either a function or a primitive value.

```JS
d3.select('body')
  ...
  .style('padding', '1em')
  .style('background-color', 'red')
  .style('margin', '1px')
```

Use the `.style()` method to apply styles to each element. Style takes two parameters. The first is the style property name. The second is the value of that property. 

`.style('padding', '1em')` is equivalent to `padding: 1em`

This is good but in some cases, you'll want the styles to be related to the dated value. In this case, you can replace the second argument with a callback! 

With these properties in place, each row should have a padding of 1em, a background color of red, and a 1px margin. Things are starting to look interesting. 

```JS
d3.select('body')
  ...
  .style('width', (d) => `${d / 10 * 100}%`)
```

Here you've added a callback. The callback takes the data as a parameter and returns the value for the width property. 

With this in place, the width of each bar is a percentage of the width of the screen.

### D3 HTML Review 

Inspect the example above in your browser. To see what was generated by D3. It should look something like: 

```HTML 
<div style="padding: 1em; background-color: red; margin: 1px; width: 50%;">$5.00</div>
<div style="padding: 1em; background-color: red; margin: 1px; width: 60%;">$6.00</div>
<div style="padding: 1em; background-color: red; margin: 1px; width: 20%;">$2.00</div>
<div style="padding: 1em; background-color: red; margin: 1px; width: 80%;">$8.00</div>
...
```

D3 generated a list of standard div tags and assigned some inline styles. You could have written this by hand if you had wanted to and had the time to spare. 

## Challenges 

**Challenge 1**

Add some more data to the data array. Add as many numbers to the array as you like. 

NOTE! Since you have written the width style as: `d / 10 * 100` numbers greater than 10 will go off the screen!

NOTE! Adding new numbers should generate new elements in the browser!

**Challenge 2**

Add some new styles. Try these ideas. 

- The height of each bar should be `3rem`. Setting the `border-radius` to `3rem` will round the ends. 
- Change the background to a linear gradient. Use property: `background`  and set the value to: `linear-gradient(to left, #e66465, #9198e5)`
- Set the foreground color to white. Property: `color`, value: `#fff`
- Set font to Helvetica. Property `font-family` value `Helvetica`

**Challenge 3**

Try setting a property using the data value. The values should be in the range of 0 to 10.

- Set the size of the text. Use the property `font-size`. The second argument should be a function that receives `d` and returns ``${d * 5}px``.
- Setting the opacity based on the data value. You can use a formula like: `d / 10`
- Try changing the color of the gradient-based on the data value.
