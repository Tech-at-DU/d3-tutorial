# Make a Website with D3

Every tutorial so far has used D3 to draw charts — mostly SVG. But go back to [01 Intro to D3](../01-intro-to-d3): the very first example never touched SVG at all. It bound data to plain `<div>` elements. That's the whole idea behind the name D3 — **Data-Driven Documents**. D3 doesn't care whether the element you `.append()` is a `<circle>`, a `<div>`, an `<a>`, or an `<li>`. Selection, data, enter, append — it all works the same.

This tutorial pushes that idea somewhere unusual: can you build a small webpage — nav bar, content, footer — using D3 instead of writing every element by hand?

**Be honest with yourself about this one.** Real production websites are built with tools like React, Vue, or plain hand-written HTML, because those tools solve problems D3 was never designed for — routing between pages, managing UI state, forms, accessibility, reactivity when data changes. D3 only knows how to do one thing: bind data to elements and keep the DOM in sync with that data. That's a genuinely useful skill (you already used it to make every chart in this repo), but it isn't a website framework. Treat this tutorial as an exploration of D3's boundaries, not a recommendation for how to build your next site.

## Getting started

Create a new HTML document with the usual boilerplate, but this time build out some structural tags instead of just an `<svg>`:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My D3 Website</title>
  <style>
    body { font-family: Helvetica, sans-serif; margin: 0; }
    nav { display: flex; gap: 1rem; padding: 1rem; background: #222; }
    nav a { color: white; text-decoration: none; }
    nav a:hover { text-decoration: underline; }
    #content {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 1rem;
      padding: 1rem;
    }
    .card { border: 1px solid #ccc; border-radius: 8px; padding: 1rem; }
    .card h3 { margin-top: 0; }
    footer { padding: 1rem; text-align: center; color: #666; }
  </style>
</head>
<body>
  <header>
    <nav id="nav"></nav>
  </header>
  <main id="content"></main>
  <footer id="footer"></footer>

  <script type="module">
    import * as d3 from "https://cdn.jsdelivr.net/npm/d3@7/+esm";
    // Your code here
  </script>
</body>
</html>
```

Notice `#nav`, `#content`, and `#footer` are all empty. You're going to fill them in with D3, the same way you filled in an empty `<svg>` in every other tutorial.

## Building a nav bar from data

A nav bar is just a list of links. Represent that as data:

```JS
const links = [
  { label: 'Home', href: '#home' },
  { label: 'About', href: '#about' },
  { label: 'Cities', href: '#cities' },
]
```

Now bind it to `<a>` elements inside `#nav`. This is the exact same pattern you used for circles in tutorial 01:

```JS
d3.select('#nav')
  .selectAll('a')
  .data(links)
  .enter()
  .append('a')
  .attr('href', d => d.href)
  .text(d => d.label)
```

Reload the page. You should have a row of working links across the top, generated entirely from the `links` array. Add a fourth link to the array and reload — a fourth `<a>` shows up with no other code changes. That's the "data driven" part.

## Building content from a dataset

Let's reuse `cities.csv` — the same dataset from tutorials 03 through 15 — and turn it into a grid of content cards instead of a chart.

Find the `data` folder in this repo and copy `cities.csv` into the directory where you're working, then load it:

```JS
async function loadCities() {
  const data = await d3.csv('cities.csv')
  console.log(data)
}

loadCities()
```

Each card needs more than one piece of text inside it — a heading and a couple of paragraphs. So far every tutorial has built one kind of child element per `.selectAll().data().enter().append()` chain (all circles, or all text). To build a *compound* element — a `<div class="card">` containing its own `<h3>` and `<p>` tags — you can use `.each()` to run custom code once per entered element:

```JS
async function loadCities() {
  const data = await d3.csv('cities.csv')

  d3.select('#content')
    .selectAll('.card')
    .data(data)
    .enter()
    .append('div')
    .attr('class', 'card')
    .each(function (d) {
      const card = d3.select(this) // the card div for this one data point
      card.append('h3').text(d.label)
      card.append('p').text(d.country)
      card.append('p').text(`Population: ${parseInt(d.population).toLocaleString()}`)
    })
}

loadCities()
```

`.each()` runs its callback once for every element in the selection, and inside it `this` refers to that one DOM element — the same `this` trick you used for the tooltip in tutorial 18. Wrapping it with `d3.select(this)` lets you keep using normal D3 methods (`.append()`, `.text()`) scoped to just that one card.

**Alternative approach:** tutorial 13's legend built a circle and a label as two *separate* `.selectAll().data().enter().append()` chains against the same parent group, rather than using `.each()`. Both patterns work — `.each()` keeps related elements grouped together in one pass; separate chains keep each element type's logic isolated. Neither is "more correct"; pick whichever reads clearer for what you're building.

Reload the page. You should have one card per city, each showing its name, country, and formatted population.

## Adding a footer

Not everything needs to be data-driven. A footer with the current year is a one-off:

```JS
d3.select('#footer')
  .append('p')
  .text(`© ${new Date().getFullYear()} My D3 Website`)
```

## Challenges

**Challenge:** Add more links to the `links` array. Give each nav link an `id` on a matching section of the page (`<section id="home">`, etc.) so clicking a link actually scrolls to that section.

**Challenge:** Load a different dataset (one you haven't used yet) and turn it into its own grid of cards.

**Challenge:** Give each card a click handler (`.on('click', ...)`, same as tutorial 18) that expands it to show more detail, or logs the city's data to the console.

**Stretch Challenge:** Combine this with the interaction ideas from tutorial 12. Add a dropdown or a set of buttons — one per country — that filters which cards are visible when clicked. You'll need to either re-run the join with a filtered array, or toggle a CSS class/style on cards that don't match.

## Check Your Understanding

**Q1.** Why does `.selectAll('a').data(links).enter().append('a')` work for nav links using the exact same pattern that built circles and bars in every earlier tutorial?

<details><summary>Answer</summary>

D3's selection/data/enter/append machinery doesn't know or care what kind of element it's creating — a `<circle>`, an `<a>`, and a `<div>` are all just DOM nodes to append. The "join a piece of data to an element" mechanism is completely generic; only the element type and the attributes you set afterward change based on what you're building.

</details>

**Q2.** If `loadCities()` accidentally ran twice — say, called once directly and once from a button click handler — what would happen to the page?

<details><summary>Answer</summary>

You'd get duplicate cards. `.selectAll('.card').data(data).enter()` only knows about `.card` elements that exist *at the moment it runs*. Run it a second time with the same data, and D3 has no matching existing elements to reuse (this call has no memory of the previous call), so `.enter()` treats every row as brand new and appends a second full set. This is exactly the problem tutorial 19's `.join()` update/exit pattern is built to prevent — worth revisiting if you build the filtering stretch challenge above.

</details>

**Q3.** Given the honesty check at the top of this tutorial, when would using D3 like this actually make sense on a real project, versus reaching for React or plain HTML?

<details><summary>Answer</summary>

There's no universally right answer, but a reasonable line: D3 shines when you have a chunk of a page that's genuinely data-driven — a pricing table, a portfolio grid, a leaderboard — sitting inside an otherwise normal, hand-built or framework-built site. It's a poor fit for the site's overall structure, navigation, or anything needing state management across many interacting pieces, which is exactly what frameworks like React (tutorial 16) exist to handle.

</details>

## Conclusion

In this tutorial you used D3 to build ordinary HTML page structure — a nav bar, a data-driven content grid, a footer — using nothing but the selection, data, enter, and append pattern you first learned in tutorial 01. Nothing about that pattern is chart-specific; "Data-Driven Documents" was never only about charts, charts are just the most common use. Understanding that boundary — what D3 actually is (a data-binding and DOM-update library) versus what a full framework additionally provides — is worth having clear in your head, whether or not you ever build a page this way again.
