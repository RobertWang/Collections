> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zserge.com](https://zserge.com/posts/awfice/)

> Let's build a tiny browser-based office suite - a text editor, a speadsheet, a drawing app and a pres......

We are all familiar with a traditional office suite - a word processor, a spreadsheet, a presentation program, maybe a diagramming or note-taking app. We have seen it all in Microsoft Office and Google Docs. Those are really powerful and large. But what would be the most minimal amount of code required to build an office suite?

platform
--------

Obviously, our office suite won’t be a desktop GUI app - those require plenty of code and efforts to build one. Same applies to native mobile apps. We might consider building a console (terminal-based) app, and in fact there are already absurdly small [text editors](https://github.com/antirez/kilo) or [spreadsheets](https://github.com/c00kiemon5ter/ioccc-obfuscated-c-contest/blob/master/2000/jarijyrki.c), but it would be much easier if we targeted a browser.

Browsers already come with a decent rich-text editor (contenteditable) and are really good (although, unsafe) [evaluator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval) for math expressions.

Now, how small can we get?

text editor
-----------

This is in fact the “app” I have been using for years:

Yes, that’s it. Moreover, one can turn it into a self-contained URL and that’s how I use it when I need a scratchpad for quick notes:

```
data:text/html,<html contenteditable>
```

Try pasting this into your URL bar. If your browser is nice with you, you should be able to use `Ctrl+B` or `Ctrl+I` to make text look bold or italic.

We can enhance it a bit more by adding some style (yes, I believe that [some small typographic improvements](http://bettermotherfuckingwebsite.com/) matter):

```
data:text/html,<body contenteditable>
```

I have added this to bookmarks and now my zero-weight text editor is one keypress away from me. You might also use it as a temporary clipboard to paste text or even pictures.

Of course, this can be further extended to support various heading styles, lists or indentation.

You can save your text by saving the whole HTML as a file, or by printing it on paper.

presentations
-------------

Some time ago I already made a [simple presentation tool](https://github.com/trikita/slide-html), which is a self-contained HTML file that one can edit (like markdown text) and it will render in a colorful [Takahashi-style](https://en.wikipedia.org/wiki/Takahashi_method) presentation.

This time, as we continue talking about `contenteditable`, we will make a WYSYWIG slide editor instead. First of all, let’s create several empty slides that are editable:

```
<body><script>
for (let i=0; i<50; i++) {
  document.body.innerHTML += `
    <div>
      <div contenteditable>
      </div>
    </div>`;
}
</script>
```

The number of 50 is arbitrary, but I don’t remember ever using more slides than this. Each outer div is a slide with a thin outline. The trick with width and padding-top is to keep the slide aspect ratio. Try changing the values to see how that affects the layout. Each inner div is a basic rich text editor, with a fairly large font to be readable from the projector screen.

Good enough. But we want to have headers and lists on our slides, don’t we?

Let’s add some hotkeys:

```
document.querySelectorAll("div>div").forEach(el => {
  el.onkeydown = e=> {
    // `code` will be false if Ctrl or Alt are not pressed
    // `code` will be 0..8 for numeric keys 1..9
    // `code` will be some other numeric value if another key is pressed
    // with Ctrl+Alt hold.
    const code = e.ctrlKey && e.altKey && e.keyCode-49;
    // Find the suitable rich text command, or undefined if the key
    // is out of range
    x = ["formatBlock", "formatBlock", "justifyLeft", "justifyCenter", 
         "justifyRight", "outdent", "indent", "insertUnorderedList"][n];
    // Find command parameter (only for 1 and 2)
    y = ["<h1>", "<div>"][n];
    // Send the command and the parameter (if any) to the editor
    if (x) {
      document.execCommand(x, false, y);
    }
  };
});
```

Now if we press `Ctrl+Alt+1` inside the slide - we make the selected text a header. Or if we press `Ctrl+Alt+2` we turn it back to normal. `Ctrl+Alt+3`..`Ctrl+Alt+5` change the alignment, and `6` and `7` change the indentation. `8` starts a list. `9` is left for your own needs, feel free to customize. The full list of `contenteditable` operations can be found on [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand).

Squeezing the code above a little bit and turning it into a data URL would result in the following ~600 bytes long rich slide editor:

```
data:text/html,<style>@page{size: 6in 8in landscape;}</style><body><script>d=document;for(i=0;i<50;i++)d.body.innerHTML+='<div><div contenteditable).forEach(e=>e.onkeydown=e=>{n=e.ctrlKey&&e.altKey&&e.keyCode-49,x=["formatBlock","formatBlock","justifyLeft","justifyCenter","justifyRight","outdent","indent","insertUnorderedList"][n],y=["<h1>","<div>"][n],x&&document.execCommand(x,!1,y)})</script>
```

The slides can be exported to PDF by printing into the file, and from there could be shown on any computer.

quick drawing
-------------

A while ago I’ve built [https://onthesamepage.online](https://onthesamepage.online/) to quickly sketch ideas in collaboration with other people, but despite its simplicity it’s still larger than what we do here.

As a bare minimum, we can only draw lines on canvas. We need a `<canvas>` elements, a few mouse/touch handlers and a flag to indicate that mouse movement is actually drawing when the mouse is pressed.

Here worth mentioning that elements with an id can be accessed as window[id] or window.id. The thing that wasn’t standardized for a long time and has been a hack from IE, now has become a [standard](https://html.spec.whatwg.org/multipage/window-object.html#named-access-on-the-window-object).

Also, I moved cursor position handling to separate short functions to reuse them in mousedown and mousemove handlers. Finally, I reset the margins of the body elements to make our canvas full screen.

The minified code is roughly 400 bytes and allows you to draw with your mouse, nothing more, nothing less:

```
<canvas>
<script>
d=document, // shortcut for document
d.body.style.margin=0, // reset style
f=0, // mouse-down flag
c=v.getContext("2d"), // canvas context
v.width=innerWidth, // make canvas element fullscreen
v.height=innerHeight,
c.lineWidth=2, // make lines a bit wider
x=e=>e.clientX||e.touches[0].clientX, // get X position from mouse/touch
y=e=>e.clientY||e.touches[0].clientY, // get Y position from mouse/touch
d.onmousedown=d.ontouchstart=e=>{f=1,e.preventDefault(),c.moveTo(x(e),y(e)),c.beginPath()},
d.onmousemove=d.ontouchmove=e=>{f&&(c.lineTo(x(e),y(e)),c.stroke())},
d.onmouseup=d.ontouchend=e=>f=0
</script>
```

Or, as a short one-liner bookmark:

```
data:text/html,<canvas><script>d=document,d.body.style.margin=0,f=0,c=v.getContext("2d"),v.width=innerWidth,v.height=innerHeight,c.lineWidth=2,x=e=>e.clientX||e.touches[0].clientX,y=e=>e.clientY||e.touches[0].clientY,d.onmousedown=d.ontouchstart=e=>{f=1,e.preventDefault(),c.moveTo(x(e),y(e)),c.beginPath()},d.onmousemove=d.ontouchmove=e=>{f&&(c.lineTo(x(e),y(e)),c.stroke())},d.onmouseup=d.ontouchend=e=>f=0</script>
```

spreadsheet
-----------

This would probably be the most complex one and the largest one, but we will try to stay below the limit of 1KB per app.

The layout would be simple. HTML comes with tables, so why don’t we use them. As the spreadsheet cells are normally addressable by “letter” + “number”, let’s restrict our table to 26x100 cells. It makes sense to create rows and cells dynamically in a loop. Some basic styling would make our spreadsheet look nicer:

```
<table>

t.style.borderCollapse="collapse"; // remove gaps between cells
for (let i = 0; i < 101; i++) {
  const row = t.insertRow(-1);
  for (let j = 0; j < 27; j++) {
    // convert column index j to a letter (char code of "A" is 65)
    const letter = String.fromCharCode(65+j-1); // 1=A, 2=B, 3=C etc
    const cell = row.insertCell(-1);
    cell.style.border = "1px solid silver"; // make thin grey border
    cell.style.textAlign = "right";         // right-align, like excel
    if (i > 0 && j > 0) {
      // add identifiable input field, this is where formula is entered
      const field = document.createElement('input');
      field.id = letter + i; // i.e, "B3"
      cell.appendChild(field);
    } else if (i > 0) {
      // Row numbers
      cell.innerHTML = i;
    } else if (j > 0) {
      // Column letters
      cell.innerHTML = letter;
    }
  }
}
```

Now we have a large grid of cells, with rows and columns. Time to add an expression evaluator. We can achieve a hacky, but mostly working evaluator with 3 arrays - an array of all input fields (to get their actual entered values, number or formulas), an array that has a smart getter, calling eval() if a variable is requested and it is linked to an input field with the formula, and a cache of the last entered values for each field:

```
inputs = []; // assume that we did inputs.push(field) for each field in the loop above
data = {}; // smart data accessing object
cache = {}; // cache

// Re-calculate all fields
const calc = () => {
  inputs.map(field => {
    try {
      field.value = D[field.id];
    } catch (e) { /* ignore */}
  });
};

// We also need to customize our field initialization code:
field.onfocus = () => {
  // When element is focused - replace its calculated value with its formula
  field.value = cache[field.id] || "";
};
field.onblur = () => {
  // When element loses focus - put formula in cache, and re-calculate everything
  cache[field.id] = field.value;
  calc();
};
// Smart getter for a field, evaluates formula if needed
const get = () => {
  let value = cache[field.id] || "";
  if(value.chatAt(0) == "=") {
    // evaluate the formula using "with" hack:
    with(data) return eval(value.substring(1));
  } else {
    // return value as it is, convert to number if possible:
    return isNaN(parseFloat(value)) ? value : parseFloat(value);
  }
};
// Add smart getter to the data array for both upper and lower case variants:
Object.defineProperty(data, field.id, {get}),
Object.defineProperty(data, field.id.toLowerCase(), {get})
```

Now the spreadsheet should work, if you put, for example, “42” into A1 and “=A1+3” into A2 - you should see “45” when you move the focus away from A2.

After carefully minizing the code above, we get the following ~800 byte working spreadsheet:

```
data:text/html,<table><script>for(I=[],D={},C={},calc=()=>I.forEach(e=>{try{e.value=D[e.id]}catch(e){}}),t.style.borderCollapse="collapse",i=0;i<101;i++)for(r=t.insertRow(-1),j=0;j<27;j++)c=String.fromCharCode(65+j-1),d=r.insertCell(-1),d.style.border="1px solid gray",d.style.textAlign="right",d.innerHTML=i?j?"":i:c,i*j&&I.push(d.appendChild((f=>(f.id=c+i,f.style.border="none",f.style.width="4rem",f.style.textAlign="right",f.onfocus=e=>f.value=C[f.id]||"",f.onblur=e=>{C[f.id]=f.value,calc()},get=()=>{let v=C[f.id]||"";if("="!=v.charAt(0))return isNaN(parseFloat(v))?v:parseFloat(v);with(D)return eval(v.substring(1))},Object.defineProperty(D,f.id,{get:get}),Object.defineProperty(D,f.id.toLowerCase(),{get:get}),f))(document.createElement("input"))))</script>
```

are you serious?
----------------

Well, in no way it’s a replacement for a proper office suite. But it’s a good demonstration of minimalism and tiny code. All these apps are ephemeral, they lose their state if you refresh the page and it looks like there is no way for data URLs to keep their state. But they might be helpful as quick bookmarks if you need to calculate a few bits or draft some quick note without opening heavy “real” office apps. As a bonus, all these tiny apps are ultimately respectful to your privacy and do no share your data (do not store it either).

So, yes, it is more of a joke than a serious application, but still, I created a repo for these tiny apps in case anyone would like to use them or customize further for their own needs: [http://github.com/zserge/awfice](http://github.com/zserge/awfice). PRs and further improvements are welcome!

I hope you’ve enjoyed this article. You can follow – and contribute to – on [Github](https://github.com/zserge), [Twitter](https://twitter.com/zsergo) or subscribe via [rss](https://zserge.com/rss.xml).

_Oct 11, 2020_

See also: [Let's make the worst React ever!](https://zserge.com/posts/worst-react-ever/) and [more](https://zserge.com/posts/).