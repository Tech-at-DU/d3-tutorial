# D3 Scales

Scales are used to scale and normalize values. You can use a scale to convert a value from one range to another. Convert a value to color, Convert strings to other values like colors or numbers. 

For this example continue with the code from the previouse example. You should have something like this: 

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
  <svg id="svg" width="500" height="500"></svg>

  <script src="https://d3js.org/d3.v7.min.js"></script>
  <script>
    d3.csv('cities.csv')
      .then(data => {
        console.log(data)
        d3.select('#svg')
          .style('border', '1px solid')
          // select all <circle>s in #svg
          .selectAll('circle')
          .data(data)
          .enter()
          .append('circle')
          .attr('cx', d => parseFloat(d.x) * 2 + 250)
          .attr('cy', d => parseFloat(d.y) * 2 + 250)
          .attr('r', d => parseInt(d.population) * 0.00001)
          .attr('fill', `red`)
          .attr('opacity', 0.25)
          .attr('fill', d => {
            if (d.country === 'USA') {
              return 'cornflowerblue'
            } else if (d.country === 'Pakistan') {
              return 'gold'
            } else if (d.country === 'Italy') {
              return 'green'
            } else if (d.country === 'Brazil') {
              return 'tomato'
            }
          })
      })
  </script>
</body>
</html>
```

## Using D3 Scales

A scale needs a domain and a range. The domain is the input and the range is the output. 

**Domains**

A scale needs to know the domain, this is where the values come from. The domain can be a range like a minimum and maximum value. 

For example, in our cities data, the x value is roughly in the range of -46 to +122. 

The domain for the countries might be an array of all available country names: `['USA', 'Pakistan', 'Italy', 'Brazil']`. 

The domain of population is crazy large numbers nuf said. 

**Ranges**

Ranges can be numbers or other values and can be expressed as min and max values or a list of possible choices. 

For example, if we are posting something on the screen and our SVG canvas is 500 pixels wide the range is 500. 

If we have a list of colors to match our countries against the range might be: `['cornflowerblue', 'gold', 'gold', 'tomato']` a list of colors! 

When it comes to the population we might know that we want the largest circle to be 300px and the smallest to be 10px. 

D3 has several functions that return a scale function that you configure for your use. 

## Scale Linear

A linear scale scales a value proportionally from the domain to the range. 

Create a linear scale. 

```JS
d3.csv('cities.csv')
  .then(data => {
    // Create a linear scale
    const xScale = d3.scaleLinear()
      .domain([-150, 150]) // Set the domain
      .range([0, 500])     // Set the range

    d3.select('#svg')
      .style('border', '1px solid')
      // select all <circle>s in #svg
      ...
```

What happened here: 

```JS
const xScale = d3.scaleLinear()
  .domain([-150, 150]) // Set the domain
  .range([0, 500])     // Set the range
```
Here you are creating a function `xScale` that is a linear scale function. 

You set the domain to `[-150, 150]`. This says this function will translate numbers from -150 to +150. 

Last you set the range to `[0, 500]`. This says your scale function will return values from 0 to 500. 

All together the `xScale` function expects to take in numbers from -150 to 150 and return valuse from 0 to 500. You should expect:

- `xScale(-150)` to return `0` 
- `xScale(0)` to return `250` 
- `xScale(150)` to return `500`

Use `xScale` to set the `cx` attribute: 

```JS
...
.attr('cx', d => xScale(d.x))
...
```

This is a big improvement over the last method! 

Changing the size of your SVG. makes it 800 wide. 

```HTML
...
<svg id="svg" width="800" height="500"></svg>
...
```

Fix the scale to match the new width. To do this you'll change the range of the scale. The domain has not changed. 

```JS
const xScale = d3.scaleLinear()
  .domain([-150, 150])
  .range([0, 800]) // Change the range!
```

**Challenge**

Create a new linear scale for the y value. 

### Scale Ordinal

An ordinal scale maps arbitrary values like strings and dates to other arbitrary values like strings and colors. 

The country names would use an ordinal scale. 

Define a new oridinal scale: 

```JS
const countryScale = d3.scaleOrdinal()
  .domain(['USA', 'Pakistan', 'Italy', 'Brazil'])
  .range(['cornflowerblue', 'gold', 'gold', 'tomato'])
```

Then use this new scale to set the color: 

```JS
...
.attr('fill', d => countryScale(d.country))
...
```

This was a big improvement! 

What about population? This was a very ugly implementation with some arbitrary values. Why multiply by `0.00001`? We can do better with a scale! 

Make a new scale: 

```JS
const popScale = d3.scaleLinear()
  .domain([525010, 14910352])
  .range([10, 100])
```

Here I found the largest and smallest population values and used these for the domain. 

Then I decided on the largest and smallest circles diameter I wanted. I chose: 10 and 100. 

Now use the scale: 

```JS
...
.attr('r', d => popScale(d.population))
...
```

This is a lot better than the previous solution. 

**Challenge**

Edit the domain until the largest and smallest circles are sizes that you like: 

Wait! These circles do not display the populations as accurately as they could be. Currently, we're showing the diameter as the population but the circle expresses itself as an area. 

You can think of this as drawing a line where each pixel represents a person. If you had a 10-pixel line and a 20-pixel line the second would represent twice as many people. 

If you drew a 10px radius circle it would contain 78 pixels. A 20-pixel diameter circle would contain 314 pixels. The difference there is about 4 to 1. Remember were trying to express a 2 to 1 relation. 

D3 can help us with a scale! 

### scaleSqrt

`scaleSqrt` scales by area rather than linear. Try it out here. 

Edit the `popScale`:

```JS
const popScale = d3.scaleSqrt()
  .domain([525010, 14910352])
  .range([10, 200])
```

Notice the change. It may appear to be subtle. The sizes should be more accurately related now. 

## Finding min and max 

This is getting better all the time. There are still a couple of places where our code is awkward. 

Notice the population scale. To get the range we needed to look at the cities.csv and find the largest and smallest values. This would not work with more than 10 items. It's not going to work if something changes. 

The same is true for the x and y coordinates. 

You could use vanilla JS to calculate these but D3 provides some helper functions. 

In your code before you create your scales add: 

```JS
const minX = d3.min(data, d => parseFloat(d.x))
const maxX = d3.max(data, d => parseFloat(d.x))
```

You can use these values in place of the hard coded numbers:

```JS
const xScale = d3.scaleLinear()
  .domain([minX, maxX]) // Use the min and max here
  .range([700, 100])
```

**Challenge**

Find the min and max y values.

Use these values in the domain for the y scale function. 

**Challenge**

Find the min and max population values. 

Use these values in the domain of the population scale function. 

### Making a set

The last area where things are not working well is in country names. We had to add all of these names manually. If there had been more than four this would not have been easy, and as it is it's not scalable. 

There is probably a strictly D3 method for this. I had a hard time figuring this since the methods seem to change over different versions of d3. We are using d3 v7 which didn't have an obvious answer.

I used vanilla JS for this. Try this: 

```JS
const countries = Array.from(new Set(data.map(d => d.country)))
```

Here `countries` should be a list of unique country names. 

Use this to define the domain for your `countryScale`.

```JS
const countryScale = d3.scaleOrdinal()
  .domain(countries) // Use the list counties here
  .range(['cornflowerblue', 'gold', 'gold', 'tomato'])
```

While these changes don't make a large change in the visualization behind the scenes they make big differences in how our code operates! 

### Extents

Earlier you found the minimum and maximum values from a set of values. These are the extents. Since this is such a common operation D3 provides a helper method that does just this! 

Like everything else, the extents method takes a callback! The extents method returns an array containing the minimum and maximum values. 

Try it out:

```JS
// Replace this: 
const minX = d3.min(data, d => parseFloat(d.x))
const maxX = d3.max(data, d => parseFloat(d.x))
// With: 
const x_extent = d3.extent(data, d => parseFloat(d.x))
```

Now define your `xScale` using the `x_extent` for the domain. 

```JS
const xScale = d3.scaleLinear()
  .domain(x_extent) // Use x_extent here! 
  .range([700, 100])
```

**Challenge**

Use `d3.extent()` to find the min and max values for y. 

Then use this to define the `yScale`.

**Challenge**

Use extent to find the min and map population. 

Then use the population extent to set the population domain. 

![example](images/example-1.png)

