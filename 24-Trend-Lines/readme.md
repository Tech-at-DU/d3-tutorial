# Trend Lines

Real data is noisy. A trend line cuts through that noise and shows the underlying direction — is this going up, down, or flat? This tutorial covers two ways to compute one, and how to draw it with `d3.line()`.

This isn't a classroom toy problem. Trend lines are one of the most common "can you add ___ to this chart" requests you'll get, on the job or in a class critique — a scatter of noisy points rarely tells a story on its own, but a trend line drawn through it often does.

## Two kinds of trend line

- **Moving average** — smooths the data by averaging each point with its neighbors. Follows the data's shape closely, lags behind sudden changes. Good for "smooth out the noise but keep the wiggles."
- **Linear regression** — fits the single straight line that best matches every point. Ignores local wiggles entirely. Good for "is this overall going up or down, and by how much?"

Neither is "more correct" — pick based on the question you're asking. "What's the underlying pattern hiding under this noisy signal?" → moving average. "Is this generally trending up?" → regression.

## Trend lines in the wild

- **[The Keeling Curve](https://keelingcurve.ucsd.edu/)** — daily atmospheric CO₂ at Mauna Loa since 1958. The raw data saws up and down every year (CO₂ rises and falls with the seasons as plants grow and die back). A smoothed trend line overlaid on it strips out that seasonal wiggle and shows the steady year-over-year rise. Two trend lines, two different stories, same dataset.
- **[NASA: Global Surface Temperature](https://science.nasa.gov/earth/explore/earth-indicators/global-temperature)** — annual global temperature is noisy (El Niño/La Niña years swing it around), so NASA plots a 5-year running average — a moving average — on top so the long-term warming trend isn't lost in the year-to-year noise. This is almost exactly what Part 2 of this tutorial does, with a different dataset.
- **[FRED: Unemployment Rate](https://fred.stlouisfed.org/series/UNRATE)** — the St. Louis Fed's data explorer lets you add a trend line to any economic series it hosts. Worth poking at.

In every case, the raw data alone either buries the story in noise or gets misread from a single unusual data point. The trend line is what turns "here's some numbers" into "here's what's actually happening."

## Part 1: Moving average, over precipitation

This picks up right where [07-Paths](../07-Paths) and [08-axis](../08-axis) left off. If you don't have that code handy, go build it first — this section assumes you already have `baData`, `xscale`, `yscale`, `linegen`, `svg`, and `graph` from the "Complete solution for reference" at the end of 08-axis.

Month-to-month rainfall is spiky — a wet month next to a dry one, over and over. That's exactly the kind of noise a moving average is built to smooth out.

A **simple moving average (SMA)** replaces each point with the average of itself and its `n` nearest neighbors — a sliding "window."

```JS
function movingAverage(data, windowSize, accessor) {
  return data.map((d, i, arr) => {
    const start = Math.max(0, i - windowSize + 1)
    const window = arr.slice(start, i + 1)
    return { date: d.date, precipitation: d3.mean(window, accessor) }
  })
}

const smoothed = movingAverage(baData, 5, d => d.precipitation)
```

Picture sliding a 5-month-wide window left to right across the data. At each stop, average whatever's inside the window — that average becomes the new point. Noisy up-and-down months get flattened out because each one is now blended with its neighbors.

**New here — the extra `.map()` arguments.** Every earlier tutorial used `.map(d => ...)` with one argument. `.map()` actually always passes three: `(element, index, wholeArray)`. Here all three matter — `d` is the current point, `i` is its position (so you know how far back the window can reach), and `arr` is the full array (so you can `.slice()` a window out of it).

**What's `accessor`?** A function you hand in that says "given one data point, give me the number I care about" — here, `d => d.precipitation`. `movingAverage` doesn't know or care what field name you're smoothing; it just calls `accessor(d)` wherever it needs a number. You've written accessors like this before, inline, for scales (`d => d.total`) — this is the same idea, just passed around as its own variable so `movingAverage` stays reusable for any field.

**What's `d3.mean()`?** A D3 helper that averages an array. `d3.mean(window, accessor)` calls `accessor` on every item in `window` and returns the average. Same job as `window.reduce((sum, d) => sum + accessor(d), 0) / window.length`, just shorter. D3 has matching helpers for `d3.sum`, `d3.max`, `d3.min`, and `d3.extent` — all take an array and an accessor, same pattern you used back in tutorial 04.

Notice `movingAverage` returns points shaped exactly like `baData` — `{ date, precipitation }`. That's deliberate: it means `linegen` doesn't need to change at all to draw the smoothed line.

```JS
graph
  .append('path')
  .datum(smoothed) // the smoothed data, not baData
  .attr('d', linegen) // same line generator you already built
  .attr('stroke-width', 2)
  .attr('stroke', 'firebrick')
  .attr('stroke-dasharray', '4 2') // dashed, so it reads as distinct from the raw line
  .attr('fill', 'none')
```

Note this `.append('path')` comes *after* the one drawing the raw `baData` line — later elements paint on top in SVG, so the trend line layers above the noisy original instead of underneath it.

**Challenge:** Try a window of 3, then 12. Compare — a window of 12 is roughly "smooth out a year of seasonal swing," a window of 3 barely smooths anything. Which tells a clearer story about this state's rainfall pattern?

**Challenge:** Add a legend or label distinguishing the raw line from the trend line — a dashed red line with no explanation just looks like more noise.

## Part 2: Linear regression, over 117 years of temperature

This one draws on ideas from [11-Areas](../11-Areas) and [12-Interaction](../12-Interaction) — the `Weather Data in India from 1901 to 2017.csv` dataset and its `convertToArray()` helper — but sets up its own small chart, since the question here ("is this warming, over the long run?") needs the whole 117-year span on one axis, not one year's worth of months like 11-Areas used.

Start a new HTML document with the usual boilerplate — including an `<svg id="svg" width="600" height="300"></svg>` — copy `Weather Data in India from 1901 to 2017.csv` into your project folder, then load it:

```JS
async function handleData() {
  const data = await d3.csv('Weather Data in India from 1901 to 2017.csv')
  // draw stuff here...
}

handleData()
```

Each row is one year, with a column per month. Reuse `convertToArray()` from tutorial 11 to turn one year's row into an array of `{ month, temp }`, then average that array down to a single annual mean temperature:

```JS
function convertToArray(obj) {
  const months = ['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC']
  return months.map(month => {
    const temp = parseFloat(obj[month])
    return { month, temp }
  })
}

function annualMean(yearRow) {
  const months = convertToArray(yearRow)
  return d3.mean(months, d => d.temp)
}
```

Now build a small dataset — one point per year, `{ year, temp }` — from all 117 rows:

```JS
const yearlyData = data.map(d => ({
  year: parseInt(d.YEAR),
  temp: annualMean(d)
}))
```

`yearlyData` is noisy — some years run warmer, some cooler — but the question isn't "what happened in any one year," it's "what's the overall direction across all of them?" That's a job for linear regression, not a moving average.

A **linear regression** finds the slope and intercept of the straight line that minimizes the distance to every point (least squares). No stats library needed for the simple case — it's a handful of sums:

```JS
function linearRegression(data, xAccessor, yAccessor) {
  const xMean = d3.mean(data, xAccessor)
  const yMean = d3.mean(data, yAccessor)

  const slope = d3.sum(data, d => (xAccessor(d) - xMean) * (yAccessor(d) - yMean))
              / d3.sum(data, d => (xAccessor(d) - xMean) ** 2)

  const intercept = yMean - slope * xMean

  return { slope, intercept }
}

const { slope, intercept } = linearRegression(yearlyData, d => d.year, d => d.temp)
```

Picture nudging a straight ruler around on top of the scatter of yearly points until it's as close as possible to every one of them at once — not perfect for any single point, but the best overall fit. `slope` and `intercept` describe that ruler: how steep it is, and where it crosses the y-axis. `y = slope * x + intercept` is the same `y = mx + b` from algebra class.

**Two accessors this time.** Same idea as `accessor` in Part 1 — a function that pulls one number out of a data point — but regression compares x *and* y, so it needs one of each.

**What's `** 2`?** JavaScript's exponent operator. `x ** 2` means "x squared." It shows up here because least-squares math squares each distance before summing, so points above the line and points below it don't cancel each other out.

**What's `const { slope, intercept } = ...`?** Destructuring. `linearRegression` returns one object, `{ slope, intercept }`, and this line unpacks it straight into two separate variables instead of writing `result.slope` and `result.intercept` everywhere after. The same trick works on arrays with `[ ]` — that's what `const [x0, x1] = d3.extent(...)` does below.

Once you have `slope`/`intercept`, you only need **two points** — the line's value at the start and end of your year range — to draw the whole line:

```JS
const [x0, x1] = d3.extent(yearlyData, d => d.year)
const trendPoints = [
  { year: x0, temp: slope * x0 + intercept },
  { year: x1, temp: slope * x1 + intercept }
]
```

Set up scales and draw both the raw yearly points and the trend line:

```JS
const width = 600
const height = 300
const margin = 40

const xscale = d3.scaleLinear()
  .domain(d3.extent(yearlyData, d => d.year))
  .range([margin, width - margin])

const yscale = d3.scaleLinear()
  .domain(d3.extent(yearlyData, d => d.temp))
  .range([height - margin, margin])

const svg = d3.select('#svg')

// The raw yearly points, one circle each
svg
  .selectAll('circle')
  .data(yearlyData)
  .enter()
  .append('circle')
  .attr('cx', d => xscale(d.year))
  .attr('cy', d => yscale(d.temp))
  .attr('r', 2)
  .attr('fill', 'cornflowerblue')

// The regression line, drawn from just the two trendPoints
const linegen = d3.line()
  .x(d => xscale(d.year))
  .y(d => yscale(d.temp))

svg
  .append('path')
  .datum(trendPoints)
  .attr('d', linegen)
  .attr('stroke', 'firebrick')
  .attr('stroke-width', 2)
  .attr('fill', 'none')
```

That's it — a scatter of 117 noisy yearly averages, cut through by a single straight line answering "is this actually warming?"

**Prefer not to hand-roll the math?** [d3-regression](https://github.com/HarryStevens/d3-regression) adds `d3.regressionLinear()`, `regressionPoly()`, `regressionExp()`, and more, as an extra `<script>` tag — same idea, more curve shapes.

**Challenge:** Add axes (tutorial 08) to this chart, so the year and temperature values are actually readable.

**Challenge:** What's the slope telling you? Multiply it by 100 to get "degrees per century." Is that number alarming, reassuring, or somewhere in between?

**Stretch Challenge:** Combine both techniques on one chart — draw the raw yearly points, a moving average over them (smoothing year-to-year swings without flattening the trend entirely), and the linear regression line, all three distinctly styled.

## Check Your Understanding

**Q1.** A moving average and a linear regression are both "trend lines," but they answer different questions. What's the actual difference in what each one is doing to the data?

<details><summary>Answer</summary>

A moving average is *local* — each smoothed point only looks at its nearby neighbors, so the result still follows the data's shape, just with the sharp jitter sanded off. A linear regression is *global* — it considers every point at once and collapses the whole series down to a single straight line, discarding any local wiggle entirely in favor of one overall direction.

</details>

**Q2.** In `movingAverage`, why does `start` get clamped with `Math.max(0, i - windowSize + 1)` instead of just `i - windowSize + 1`?

<details><summary>Answer</summary>

For the first few points in the array, `i - windowSize + 1` would be negative — there aren't `windowSize` neighbors before the start of the data yet. `Array.slice()` with a negative index counts from the *end* of the array instead of clamping to zero, which would silently pull in the wrong data. `Math.max(0, ...)` prevents that by never letting the window start before index 0.

</details>

**Q3.** The regression line is drawn from only two points (`trendPoints`), while the moving average line is drawn from as many points as the original data. Why does regression get away with just two?

<details><summary>Answer</summary>

A moving average line can bend — it needs enough points to trace its actual (locally varying) shape. A regression line is, by definition, perfectly straight, and a straight line is fully determined by any two points on it. Computing more than two would be extra work for a visually identical result.

</details>

## Conclusion

In this tutorial you computed and drew two different kinds of trend line — a moving average that smooths noisy precipitation data while following its shape, and a linear regression that cuts through 117 years of noisy yearly temperatures to answer one question: is it going up? Both reused `d3.line()` and scales you already knew how to build — the new work was entirely in the math that generated the points to feed them, not the drawing.

## Additional Resources

- https://datavizproject.com/data-type/trendline/ — trendline as its own chart type: what it's for, when it's used
- https://www.data-to-viz.com/graph/line.html — line charts, and pitfalls to avoid
- https://observablehq.com/@d3/moving-average — moving average, D3's own example
- https://observablehq.com/@hydrosquall/simple-linear-regression-scatterplot-with-d3 — a regression line over a scatterplot
- https://observablehq.com/@harrystevens/introducing-d3-regression — tour of the d3-regression library (linear, polynomial, exponential, LOESS...)
- https://github.com/HarryStevens/d3-regression
- https://d3indepth.com/shapes/#lines
