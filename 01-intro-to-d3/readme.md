# D3 Tutorial

This tutorial will get you started learning D3. D3 is deep! This tutorial covers the fundamental concepts. Later tutorials will delve into more advanced features in D3.

D3 is written with JavaScript and runs in a browser. Most often D3 your D3 projects will display in a browser.

## Getting started

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

Link to the library from the CDN. Add the script tag at the bottom of the body tag:

```HTML
<body>

  <script src="https://d3js.org/d3.v7.min.js"></script>
  <script>
    // Your script here!
  </script>
</body>
```

You can find this link on the [D3 hompage](https://d3js.org)

For more info take a look at GitHub: https://github.com/d3/d3.

### Displaying elements

One of the fundamental steps when working with D3 is selecting elements. You will do this when you want to create new elements or when you want to modify existing elements. 

Here you are selecting the `body`. You will be adding or modifying elements within this selected element:

```JS
d3.select('body') // select the body element
```

Next, select the elements you want to modify or create. In this case, you are selecting divs.

```JS
d3.select('body')
  .selectAll('div') // Select all divs in the body
```

_It doesn't matter that there are no existing divs._ They will exist later D3 will make them!

Next, add some data. Data can be an array containing any type of data. For this example, I've used an array of numbers.

```JS
d3.select('body')
  .selectAll('div')
  .data( [5,6,2,8,4,9,1] ) // Set data 
```

Next call `enter()`. Calling enter tells D3 to "enter" the data. Imagine at this point D3 is looping over your data.

```JS
d3.select('body')
  .selectAll('div')
  .data([5,6,2,8,4,9,1])
  .enter() // enter the data
```

Use `append()` to tell D3 what type of element to append.

```JS
d3.select('body')
  .selectAll('div')
  .data([5,6,2,8,4,9,1])
  .enter()
  .append('div') // append some divs
```

Here D3 creates elements if they don't exist. You need one div for each of the values in your data.

While you haven't drawn anything visible yet. There should be one div created for each number in the array. 

The code you have written so far is boilerplate for D3 and is a starting place for most projects.

### D3 Cheatsheet

Let's review what just happened. Here's the formula for getting started with D3.

```JS
d3.select('body')        // select an element to host
  .selectAll('div')			 // select all the elements you want to work with
  .data([5,6,2,8,4,9,1]) // define your data
  .enter()               // enter your data
  .append('div')         // append elements
```

- `d3.select('any-css-selector')` - Select an element to host the elements you will create. This element will contain all of the elements d3 creates. This element should exist and you will probably want to identify it with an id.
- `.selectAll('any-css-selector')` - Select all elements you will be working with. These elements don't need to exist and will be created if they don't exist.
- `.data([an,array,of,data])` - Add an array of data
- `enter()` - Begins looping over the data.
- `.append('tag-name')` - Appends elements to the DOM for each piece of data.

Time to draw something!

## D3 Working with elements and data

Set the text of each new node. The `text()` method takes a callback function that receives one of the data values and should return the text to display in the node.

The text node is what appears between the opening and closing tags: `<div>Hello World</div>`

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
  .text(d => `$${d.toFixed(2)}`) // Format the numbers
```

Here you took the data element and converted it to a string, added a "$" and formatted to two decimal places.

Stop a moment and imagine the piece of data (`d`) was a passenger object from the Titanic. 🤔 You could print the name or the fare...

### Styling Elements

The `style()` method takes two arguments. First is a CSS style property, and second is either a function or a primitive value.

```JS
d3.select('body')
  ...
  .style('padding', '1em') // padding: 1em
  .style('background-color', 'red') // background-color: red
  .style('margin', '1px') // margin: 1px
```

`.style('padding', '1em')` is equivalent to `padding: 1em`

This is good but in some cases, you'll want the styles to be related to the dated value. In this case, you can replace the second argument with a callback!

With these properties in place, each row should have a padding of 1em, a background color of red, and a 1px margin. Things are starting to look interesting.

Next you should set the width of each element based on the value in your data. 

```JS
d3.select('body')
  ...
  .style('width', (d) => `${d / 10 * 100}%`) // set width based on data
```

Here you've added a callback. The callback takes some data as a parameter and returns the value for the width property.

With this in place, the width of each bar is a percentage of the width of the screen. 

The calculation assumes the max value is 10. If didn't know what the maximum value in the data was you would need to find that first to make this calculation work correctly!

### D3 HTML Review

Inspect the example above in your browser. To see what was generated by D3. It should look something like:

```HTML
<div style="padding: 1em; background-color: red; margin: 1px; width: 50%;">$5.00</div>
<div style="padding: 1em; background-color: red; margin: 1px; width: 60%;">$6.00</div>
<div style="padding: 1em; background-color: red; margin: 1px; width: 20%;">$2.00</div>
<div style="padding: 1em; background-color: red; margin: 1px; width: 80%;">$8.00</div>
...
```

D3 generated a list of standard div tags and assigned some inline styles. You could have written this by hand if you had wanted to and had the time to spare. but why bother when the computer can do it for you! 

## Challenges

Try these challenges to check your understanding! 

**Challenge 1**

Add some more data to the data array. Add as many numbers to the array as you like.

**Stretch Challenge:** Write a function to generate an array of random values. 

NOTE! Since you have written the width style as: `d / 10 * 100` numbers greater than 10 will go off the screen!

NOTE! Adding more values should generate new elements in the browser!

**Challenge 2**

Add some new styles. Try these ideas.

- The height of each bar should be `3rem`. Setting the `border-radius` to `3rem` will round the ends.
- Change the background to a linear gradient. Use property: `background-image` and set the value to: `linear-gradient(to left, #e66465, #9198e5)`
- Set the foreground color to white. Property: `color`, value: `#fff`
- Set font to Helvetica. Property `font-family` value `Helvetica`

**Challenge 3**

Try setting a property using the data value. The values should be in the range of 0 to 10.

- Set the size of the text. Use the property `font-size`. The second argument should be a function that receives `d` and returns `${d * 5}px`.
- Setting the opacity based on the data value. You can use a formula like: `d / 10`
- Try changing the color based on the data value. HSL colors make this easy soemthing like: `hsl(d / 10 * 360, 100%, 50%)`

## Conclusion 

In this tutorial you learned how to create a data driven document with D3. 
