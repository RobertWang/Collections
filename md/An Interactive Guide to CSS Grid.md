> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.joshwcomeau.com](https://www.joshwcomeau.com/css/interactive-guide-to-grid/)

> CSS Grid is an incredibly powerful tool for building layouts on the web, but like all powerful tools,......

Introduction
------------

CSS Grid is one of the most amazing parts of the CSS language. It gives us a ton of new tools we can use to create sophisticated and fluid layouts.

It's also _surprisingly complex._ It took me quite a while to truly become comfortable with CSS Grid!

In this tutorial, I'm going to share the biggest 💡 lightbulb moments I've had in my own journey with CSS Grid. You'll learn the fundamentals of this layout mode, and see how to do some pretty cool stuff with it. ✨

CSS is comprised of several different [layout algorithms](https://www.joshwcomeau.com/css/understanding-layout-algorithms/), each designed for different types of user interfaces. The default layout algorithm, Flow layout, is designed for digital documents. Table layout is designed for tabular data. Flexbox is designed for distributing items along a single axis.

CSS Grid is the latest and greatest layout algorithm. It's _incredibly_ powerful: we can use it to build complex layouts that fluidly adapt based on a number of constraints.

The most unusual part of CSS Grid, in my opinion, is that the grid _structure_, the rows and columns, are defined **purely in CSS:**

With CSS Grid, a single DOM node is sub-divided into rows and columns. In this tutorial, we're highlighting the rows/columns with dashed lines, but in reality, they're invisible.

_This is super weird!_ In every other layout mode, the only way to create compartments like this is by adding more DOM nodes. In Table layout, for example, each row is created with a `<tr>`, and each cell within that row using `<td>` or `<th>`:

```
html

<table>

  <tbody>

    <tr>

      <td></td>

      <td></td>

      <td></td>

    </tr>

    <tr>

      <td></td>

      <td></td>

      <td></td>

    </tr>

  </tbody>

</table>

```

Unlike Table layout, CSS Grid lets us manage the layout entirely from within CSS. We can slice up the container however we wish, creating compartments that our grid children can use as anchors.

We opt in to the Grid layout mode with the `display` property:

```
css

.wrapper {

  display: grid;

}

```

By default, CSS Grid uses a single column, and will create rows as needed, based on the number of children. This is known as an _implicit grid_, since we aren't explicitly defining any structure.

Here's how this works:

```
.parent {

  display: grid;

  height: 300px;

}        

```

Show Perspective

By default, CSS Grid will create a single-column layout. We can specify columns using the `grid-template-columns` property:

### Code Playground

```
<style>

  .parent {

    display: grid;

    grid-template-columns: 25% 75%;

  }

</style>

<div>

  <div>1</div>

  <div>2</div>

</div>

```

By passing two values to `grid-template-columns` — `25%` and `75%` — I'm telling the CSS Grid algorithm to slice the element up into two columns.

Columns can be defined using any valid [CSS <length-percentage> value](https://developer.mozilla.org/en-US/docs/Web/CSS/length-percentage), including pixels, rems, viewport units, and so on. Additionally, we also gain access to a new unit, the `fr` unit:

### Code Playground

```
<style>

  .parent {

    display: grid;

    grid-template-columns: 1fr 3fr;

  }

</style>

<div>

  <div>1</div>

  <div>2</div>

</div>

```

`fr` stands for “fraction”. In this example, we're saying that the first column should consume 1 unit of space, while the second column consumes 3 units of space. That means there are 4 total units of space, and this becomes the denominator. The first column eats up ¼ of the available space, while the second column consumes ¾.

The `fr` unit brings Flexbox-style flexibility to CSS Grid. Percentages and `<length>` values create hard constraints, while `fr` columns are free to grow and shrink as required, to contain their contents.

**Try shrinking this container to see the difference:**

![](https://www.joshwcomeau.com/images/shadow-palette/ghost.png)

![](https://www.joshwcomeau.com/images/shadow-palette/ghost.png)

```
.parent {

  display: grid;

  grid-template-columns: 1fr 3fr;

}

```

```
.parent {

  display: grid;

  grid-template-columns: 1fr 3fr;

  gap: 16px;

}

```

Notice how the contents spill outside the grid parent when using percentage-based columns? This happens because percentages are calculated using the _total_ grid area. The two columns consume 100% of the parent's content area, and they aren't allowed to shrink. When we add 16px of `gap`, the columns have no choice but to spill beyond the container.

The `fr` unit, by contrast, is calculated based on the _extra_ space. In this case, the extra space has been reduced by 16px, for the `gap`. The CSS Grid algorithm distributes the remaining space between the two grid columns.

What happens if we add more than two children to a two-column grid?

Well, let's give it a shot:

### Code Playground

```
<style>

  .parent {

    display: grid;

    grid-template-columns: 1fr 3fr;

  }

</style>

<div>

  <div>1</div>

  <div>2</div>

  <div>3</div>

</div>

```

Interesting! Our grid gains a second row. The grid algorithm wants to ensure that every child has its own grid cell. It’ll spawn new rows as-needed to fulfill this goal. This is handy in situations where we have a variable number of items (eg. a photo grid), and we want the grid to expand automatically.

In other situations, though, we want to define the rows explicitly, to create a specific layout. We can do that with the `grid-template-rows` property:

### Code Playground

```
<style>

  .parent {

    display: grid;

    grid-template-columns: 1fr 3fr;

    grid-template-rows: 5rem 1fr;

  }

</style>

<div>

  <div></div>

  <div></div>

  <div></div>

  <div></div>

</div>

```

By defining both `grid-template-rows` and `grid-template-columns`, we've created an explicit grid. This is perfect for building page layouts, like the “Holy Grail”? layout at the top of this tutorial.

Let's suppose we're building a calendar:

CSS Grid is a wonderful tool for this sort of thing. We can structure it as a 7-column grid, with each column consuming 1 unit of space:

```
css

.calendar {

  display: grid;

  grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;

}

```

This _works_, but it's a bit annoying to have to count each of those `1fr`’s. Imagine if we had 50 columns!

Fortunately, there's a nicer way to solve for this:

```
css

.calendar {

  display: grid;

  grid-template-columns: repeat(7, 1fr);

}

```

The `repeat` function will do the copy/pasting for us. We're saying we want 7 columns that are each `1fr` wide.

Here's the playground showing the full code, if you're curious:

### Code Playground

```
<style>

  .calendar {

    display: grid;

    grid-template-columns:

      repeat(7, 1fr);

    gap: 4px;

  }

</style>

<ol>

  <li>1</li>

  <li>2</li>

  <li>3</li>

  <li>4</li>

  <li>5</li>

  <li>6</li>

  <li>7</li>

  <li>8</li>

  <li>9</li>

  <li>10</li>

  <li>11</li>

  <li>12</li>

  <li>13</li>

  <li>14</li>

  <li>15</li>

  <li>16</li>

  <li>17</li>

  <li>18</li>

  <li>19</li>

  <li>20</li>

  <li>21</li>

  <li>22</li>

  <li>23</li>

  <li>24</li>

  <li>25</li>

  <li>26</li>

  <li>27</li>

  <li>28</li>

  <li>29</li>

  <li>30</li>

  <li>31</li>

</ol>

```

By default, the CSS Grid algorithm will assign each child to the first unoccupied grid cell, much like how a tradesperson might lay tiles in a bathroom floor.

**Here's the cool thing though:** we can assign our items to whichever cells we want! Children can even span across multiple rows/columns.

Here's an interactive demo that shows how this works. _Click/press and drag_ to place a child in the grid:

```
.parent {

  display: grid;

  grid-template-columns:

    repeat(4, 1fr);

  grid-template-rows:

    repeat(4, 1fr);

}

.child {

}

```

The `grid-row` and `grid-column` properties allow us to specify which track(s) our grid child should occupy.

If we want the child to occupy a single row or column, we can specify it by its number. `grid-column: 3` will set the child to sit in the third column.

Grid children can also stretch across multiple rows/columns. The syntax for this uses a slash to delineate start and end:

```
css

.child {

  grid-column: 1 / 4;

}

```

At first glance, this looks like a fraction, ¼. In CSS, though, the slash character is not used for division, it's used to separate groups of values. In this case, it allows us to set the start and end columns in a single declaration.

It's essentially a shorthand for this:

```
css

.child {

  grid-column-start: 1;

  grid-column-end: 4;

}

```

**There's a sneaky gotcha here:** The numbers we're providing are based on the column _lines_, not the column indexes.

It'll be easiest to understand this gotcha with a diagram:

```
.child {

  grid-row: 2 / 4;

  grid-column: 1 / 4;

}

```

Using what we've learned so far, we could structure it like this:

```
css

.grid {

  display: grid;

  grid-template-columns: 2fr 5fr;

  grid-template-rows: 50px 1fr;

}

.sidebar {

  grid-column: 1;

  grid-row: 1 / 3;

}

header {

  grid-column: 2;

  grid-row: 1;

}

main {

  grid-column: 2;

  grid-row: 2;

}

```

This works, but there's a more ergonomic way to do this: _grid areas._

Here's what it looks like:

```
.parent {

  display: grid;

  grid-template-columns: 2fr 5fr;

  grid-template-rows: 50px 1fr;

  grid-template-areas:

    'sidebar header'

    'sidebar main';

}

.child {

  grid-area: sidebar;

}

```

Like before, we're defining the grid structure with `grid-template-columns` and `grid-template-rows`. But then, we have this curious declaration:

```
css

.parent {

  grid-template-areas:

    'sidebar header'

    'sidebar main';

}

```

**Here's how this works:** We're drawing out the grid we want to create, almost as if we were making ASCII art?. Each line represents a row, and each word is a name we're giving to a particular slice of the grid. See how it sorta looks like the grid, visually?

Then, instead of assigning a child with `grid-column` and `grid-row`, we assign it with `grid-area`!

When we want a particular area to span multiple rows or columns, we can repeat the name of that area in our template. In this example, the “sidebar” area spans both rows, and so we write `sidebar` for both cells in the first column.

**Should we use areas, or rows/columns?** When building explicit layouts like this, I really like using areas. It allows me to give semantic meaning to my grid assignments, instead of using inscrutable row/column numbers. That said, areas work best when the grid has a fixed number of rows and columns. `grid-column` and `grid-row` can be useful for implicit grids.

There's a big gotcha when it comes to grid assignments: **tab order will still be based on _DOM position,_ not grid position.**

It'll be easier to explain with an example. In this playground, I've set up a group of buttons, and arranged them with CSS Grid:

### Code Playground

```
<div>

  <button>

    One

  </button>

  <button>

    Four

  </button>

  <button>

    Six

  </button>

  <button>

    Two

  </button>

  <button>

    Five

  </button>

  <button>

    Three

  </button>

</div>

```

In the “RESULT” pane, the buttons appear to be in order. By reading from left to right, and from top to bottom, we go from one to six.

**If you're using a device with a keyboard, try to tab through these buttons.** You can do this by clicking the first button in the top left (“One”), and then pressing Tab to move through the buttons one at a time.

You should see something like this:

The focus outline jumps around the page without rhyme or reason, from the user's perspective. This happens because the buttons are being focused based on the order they appear in the DOM.

To fix this, we should re-order the grid children in the DOM so that they match the visual order, so that I can tab through from left to right, and from top to bottom.

In all the examples we've seen so far, our columns and rows stretch to fill the entire grid container. This doesn't need to be the case, however!

For example, let's suppose we define two columns that are each 90px wide. As long as the grid parent is larger than 180px, there will be some dead space at the end:

```
.parent {

  display: grid;

  grid-template-columns: 90px 90px;

  justify-content: start;

}

```

Show Perspective

```
.parent {

  display: grid;

  grid-template-columns: 90px 90px;

  justify-content: start;

  justify-items: stretch;

}

```

Show Perspective

```
.parent {

  display: grid;

  grid-template-columns: 90px 90px;

  justify-content: start;

}

.one {

  justify-self: stretch;

}

```

Show Perspective

```
.parent {

  display: grid;

  place-content: center;

}

```

Show Perspective

The `place-content` property is a shorthand. It's syntactic sugar for this:

```
css

.parent {

  justify-content: center;

  align-content: center;

}

```

As we've learned, `justify-content` controls the position of columns. `align-content` controls the position of rows. In this situation, we have an implicit grid with a single child, and so we wind up with a 1×1 grid. `place-content: center` pushes both the row and column to the center.

There are lots of ways to center a div in modern CSS, but this is the only way I know of that only requires two CSS declarations!

In this tutorial, we've covered some of the most fundamental parts of the CSS Grid layout algorithm, but honestly, there's _so much more stuff_ we haven't talked about!

If you found this blog post helpful, you might be interested to know that I've created a comprehensive learning resource that goes _way deeper_. It's called [CSS for JavaScript Developers](https://css-for-js.dev/).

[![](https://www.joshwcomeau.com/images/the-importance-of-learning-css/css-for-js-banner.png)](https://css-for-js.dev/)

The course uses the same technologies as my blog, and so it's chock full of interactive explanations. But there are also bite-sized videos, practice exercises, real-world-inspired projects, and even a few mini-games.

If you found this blog post helpful, _you'll love the course_. It follows a similar approach, but for the entire CSS language, and with hands-on practice to make sure you're actually developing new skills.

It's specifically built for folks who use a JS framework like React/Angular/Vue. 80% of the course focuses on CSS fundamentals, but we also see how to integrate those fundamentals into a modern JS application, how to structure our CSS, stuff like that.

If you struggle with CSS, I hope you'll check it out. Gaining confidence with CSS is _game-changing_, especially if you're already comfortable with HTML and JS. When you complete the holy trinity, it becomes so much easier to stay in flow, to truly enjoy developing web applications.

You can learn more here:

I hope you found this tutorial useful. ❤️

### Last Updated

November 22nd, 2023