# Paths Part 2

In this section we will make a new graph and follow all of the steps we used in the previous graph for review and make a few changes along the way. 

## The Data

The dataset for this example will be the `Weather Data in India from 1901 to 2017.csv`. This dataset contains the mean temperature for each month of the years for the years 1901 to 2017. 

The data came from Kaggle here: https://www.kaggle.com/mahendran1/weather-data-in-india-from-1901-to-2017

Here is a look at the data: 

```CSV
,YEAR,JAN,FEB,MAR,APR,MAY,JUN,JUL,AUG,SEP,OCT,NOV,DEC
0,1901,17.99,19.43,23.49,26.41,28.28,28.6,27.49,26.98,26.26,25.08,21.73,18.95
1,1902,19.0,20.39,24.1,26.54,28.68,28.44,27.29,27.05,25.95,24.37,21.33,18.78
2,1903,18.32,19.79,22.46,26.03,27.93,28.41,28.04,26.63,26.34,24.57,20.96,18.29
...
```

It's the year and each month. The first column is unlabeled and appears to have a row number. 

There are a few ways we can look at this data. 

1. All months and all years. This would be a lot of information but might show a trend over time. 
2. All months for a single year. This would be twelve numbers. It might show the change of temperature over the course of a year. 
3. One month for all years. For example Jan 1901, Jan 1902, Jan 1903 etc. This could show a trend of temperature changes over time. 

## Getting started

- Setup an HTML document 
- Add an SVG element
- Import the D3 library
- Add a script tag for your scripts

Do this from memory if you can. Or refer to the other tutorial projects. 

Add the following script. Here we take the idea from the last examples but this time we'll use `async` and `await`. 

```JS
async function handleData() {
  const data = await d3.csv('Weather Data in India from 1901 to 2017.csv')
  console.log(data)

}

handleData()
```

Note! `handleData()` in this example is `async`. That means we can `await` any promise to resolve inside this function. `d3.csv()` returns a promise. So we use `await` on line 2.

Using console to see the data we get: 

```JS
[
  {: "0", YEAR: "1901", JAN: "17.99", FEB: "19.43", MAR: "23.49", ...}
  {: "1", YEAR: "1902", JAN: "19.0", FEB: "20.39", MAR: "24.1", ...}
  {: "2", YEAR: "1903", JAN: "18.32", FEB: "19.79", MAR: "22.46", ...}
  ...
]
```

Each element of the array is an object. Depending on what we want to show we may have to take one row here and turn it into an array of objects. 

We should have a helper function to do that. 

**Challenge**

Write a function that takes an object with the `MON:TEMP` and turn it into an array of objects with properties `month` and `temp`. While you're at it convert those temperature strings to numbers. For example: 

```JS
// Turn this into 
const obj = {: "0", YEAR: "1901", JAN: "17.99", FEB: "19.43", MAR: "23.49", ...}
function convertToArray(obj) {
  // Challenge...
}

const arr = convertToArray(obj)
// [{ month:'JAN', temp: 17.99 }, { month: 'FEB', temp: 19.43 }, { month: 'MAR', temp: 23.49 }, ...]
```

<details><summary>**Solution**</summary>
<pre><code>
```js
function convertToArray(obj) {
  const months = ['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC']
  return months.map(month => {
    const temp = parseFloat(obj[month])
    return { month: month, temp }
  })			
}
```
</code></pre>
</details>










In the last example we created a graph with straight lines. This used the SVG path. If you recall I showed this example: 

```svg
<path d="M 10,30
  A 20,20 0,0,1 50,30
  A 20,20 0,0,1 90,30
  Q 90,60 50,90
  Q 10,60 10,30 z"/>
```

![heart](images/heart.svg)

Check the course code for this image it's the code from above wrapped in an SVG document! 

I had explained that the SVG path uses groups of numbers to describe the path it draws. In the previous example we used straight lines but the heart shape uses curves. The heart is also filled where out previous experiments used only strokes. 

