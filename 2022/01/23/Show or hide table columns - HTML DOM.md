> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [htmldom.dev](https://htmldom.dev/show-or-hide-table-columns/)

> Show or hide table columns

This post shows how to show or hide any columns of a table by righ-clicking its header. The markup consists of the table and a menu that allows user to toggle columns:

```
<table>

    ...

</table>

<ul></ul>
```

### Build the menu

Since the user can toggle any columns, the menu is constructed by items which each of them contains a checkbox to toggle particular column:

```
<ul>

    <li>

        <label> <input type="checkbox" /> Label of first column </label>

    </li>

</ul>
```

**Good to know**

If a `label` has a single checkbox and a text node, clicking the text will force the checkbox to be changed

To build up the menu items, we [loop over](https://htmldom.dev/loop-over-a-nodelist/) the table headers, and [create](https://htmldom.dev/create-an-element/) a menu item for each of them:

```
const menu = document.getElementById('menu');

const table = document.getElementById('table');

const headers = [].slice.call(table.querySelectorAll('th'));

headers.forEach(function (th, index) {

    const li = document.createElement('li');

    const label = document.createElement('label');

    const checkbox = document.createElement('input');

    checkbox.setAttribute('type', 'checkbox');

    const text = document.createTextNode(th.textContent);

    label.appendChild(checkbox);

    label.appendChild(text);

    li.appendChild(label);

    menu.appendChild(li);

});
```

To toggle the column when clicking the checkbox, we have to [handle](https://htmldom.dev/attach-or-detach-an-event-handler/) the `change` event:

```
headers.forEach(function(th, index) {

    ...

    checkbox.addEventListener('change', function(e) {

        e.target.checked ? showColumn(index) : hideColumn(index);

    });

});
```

The `showColumn` and `hideColumn` functions will show or hide the associated column which is defined by the column index. We'll see how to implement them later.

### Toggle the menu

The menu will be used to toggle table columns, but we need to toggle the menu first.

*   It'll be shown when right-clicking the table header
*   It'll be hidden when user clicks outside of the menu

```
const thead = table.querySelector('thead');

thead.addEventListener('contextmenu', function(e) {

    e.preventDefault();

    ...

    document.addEventListener('click', documentClickHandler);

});

const documentClickHandler = function(e) {

    ...

};
```

### Toggle table columns

When clicking a menu item, we will toggle the column determined by given index. In order to query all cells in the given column, we [add](https://htmldom.dev/get-set-and-remove-data-attributes/) a custom `data` attribute to each cell:

```
const numColumns = headers.length;

const cells = [].slice.call(table.querySelectorAll('th, td'));

cells.forEach(function (cell, index) {

    cell.setAttribute('data-column-index', index % numColumns);

});
```

To hide all cells in given column, we find all cells based on the `data-column-index` attribute, and [hide](https://htmldom.dev/show-or-hide-an-element/) each of them:

```
const hideColumn = function (index) {

    cells

        .filter(function (cell) {

            return cell.getAttribute('data-column-index') === `${index}`;

        })

        .forEach(function (cell) {

            cell.style.display = 'none';

        });

};
```

By taking the advantage of using `data` attribute, it's so easy to show the column:

```
const showColumn = function (index) {

    cells

        .filter(function (cell) {

            return cell.getAttribute('data-column-index') === `${index}`;

        })

        .forEach(function (cell) {

            cell.style.display = '';

        });

};
```

### Don't allow to hide the last column

As the title of section, we should prevent the last remaining column to be hidden. The associated checkbox has to be disabled.

To do so, we have to associate each checkbox in menu with the column by continuing using the `data` attribute. Let's modify the building menu code a little bit:

```
headers.forEach(function(th, index) {

    ...

    checkbox.setAttribute('data-column-index', index);

});
```

When each column is hidden, we add a custom attribute to indicate that the column is already hidden:

```
const hideColumn = function(index) {

    cells

        .filter(function(cell) {

            ...

        })

        .forEach(function(cell) {

            ...

            cell.setAttribute('data-shown', 'false');

        });

};
```

Then we can count how many columns are hidden, and if there's only one remaining column, we will disable the associated checkbox:

```
const hideColumn = function (index) {

    const numHiddenCols = headers.filter(function (th) {

        return th.getAttribute('data-shown') === 'false';

    }).length;

    if (numHiddenCols === numColumns - 1) {

        const shownColumnIndex = thead.querySelector('[data-shown="true"]').getAttribute('data-column-index');

        const checkbox = menu.querySelector(`[type="checkbox"][data-column-index="${shownColumnIndex}"]`);

        checkbox.setAttribute('disabled', 'true');

    }

};
```

To get it working completely, we need to initialize the `data-shown` attribute for each cell, and turn it back to `true` when showing a column:

```
cells.forEach(function(cell, index) {

    cell.setAttribute('data-shown', 'true');

});

const showColumn = function(index) {

    cells

        .filter(function(cell) {

            ...

        })

        .forEach(function(cell) {

            ...

            cell.setAttribute('data-shown', 'true');

        });

    menu.querySelectorAll(`[type="checkbox"][disabled]`)

        .forEach(function(checkbox) {

            checkbox.removeAttribute('disabled');

        });

};
```

In the demo below, right-click the headers to show or hide any columns. Enjoy!

### Demo

Show or hide table columns

### See also

*   [Append to an element](https://htmldom.dev/append-to-an-element/)
*   [Attach or detach an event handler](https://htmldom.dev/attach-or-detach-an-event-handler/)
*   [Calculate the mouse position relative to an element](https://htmldom.dev/calculate-the-mouse-position-relative-to-an-element/)
*   [Create an element](https://htmldom.dev/create-an-element/)
*   [Get set and remove data attributes](https://htmldom.dev/get-set-and-remove-data-attributes/)
*   [Loop over a nodelist](https://htmldom.dev/loop-over-a-nodelist/)
*   [Prevent the default action of an event](https://htmldom.dev/prevent-the-default-action-of-an-event/)
*   [Show a custom context menu at clicked position](https://htmldom.dev/show-a-custom-context-menu-at-clicked-position/)
*   [Show or hide an element](https://htmldom.dev/show-or-hide-an-element/)