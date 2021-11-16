## Drawing an Axis

This next section will continue from the percipitation graph in [07-Paths](../07-Paths).

The current graph is looking good but it would be hard for people to intuit what we were expressing with this image. Which asks the question: What is this image expressing? 

Time is is running left to right on the horizontal axis. What is the time? The graph shows 1998 to 2017. 

A the vertical axis the graph is showing how much rain fell for each month. 

There should be a month for each peak but it's hard to tell 2005 is. The higher peaks show more rain fell but I can't tell how many inches of rain fall that represents. 

We need an axis! An axis is a series of labels that run along the horizontal and vertical axis of the graph. If you're thinking D3 has a tool to generate an axis you would be correct! 

We need to express a list of dates along the horizontal axis. We may will also want to format the dates. Currently the dates from the data look like this: 

```
30/11/2016
```

This is compact but I want some flexibity here. D3 has us covered! D3 has a time parsing function. Rather than date strings we would prefer to have a date object representing each date. 

Add the following with your scales:

```JS
// Parse the dates in the data
// Dates are formatted: 30/11/2017
const parseTime = d3.timeParse('%d/%m/%Y')
// parse the dates for d3
baData.forEach(d => d.date = parseTime(d.date))
```

`d3.parseTime()` takes a string that expresses the format of our dates. Then we call our `parseTime()` function on each item in our data and pass in the existing date string. This function returns a date object. 

Modify your current code to get a refernce to the SVG element. 

```JS
// Draw something on our svg
const svg = d3.select('#svg') // change this line!
	.append('path')
```







