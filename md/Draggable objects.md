> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.redblobgames.com](https://www.redblobgames.com/making-of/draggable/)

> Many of my interactive pages have a draggable object. I want the reader to move the object around, an......

4 Jan 2023

Many of my interactive pages have a _draggable object_. I want the reader to move the object around, and I want the diagram to respond in some way. Here I’ll document the code I use to make this work with both mouse and touch input, using browser features that are widely supported since 2020. Here’s a common case I want to support:

Drag me Drag the circle with mouse or touch

This is the simple model in my head:

![](data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8IS0tIEdlbmVyYXRlZCBieSBncmFwaHZpeiB2ZXJzaW9uIDcuMS4wICgyMDIzMDEyMS4xOTU2KQogLS0+CjwhLS0gUGFnZXM6IDEgLS0+Cjxzdmcgd2lkdGg9IjI3MXB0IiBoZWlnaHQ9IjEyMnB0Igogdmlld0JveD0iMC4wMCAwLjAwIDI3MS4zMCAxMjEuNjUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPgo8ZyBpZD0iZ3JhcGgwIiBjbGFzcz0iZ3JhcGgiIHRyYW5zZm9ybT0ic2NhbGUoMSAxKSByb3RhdGUoMCkgdHJhbnNsYXRlKDQgMTE3LjY1KSI+Cjxwb2x5Z29uIGZpbGw9IndoaXRlIiBzdHJva2U9Im5vbmUiIHBvaW50cz0iLTQsNCAtNCwtMTE3LjY1IDI2Ny4zLC0xMTcuNjUgMjY3LjMsNCAtNCw0Ii8+CjwhLS0gbm90XG5kcmFnZ2luZyAtLT4KPGcgaWQ9Im5vZGUxIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub3RcbmRyYWdnaW5nPC90aXRsZT4KPGVsbGlwc2UgZmlsbD0iI2VlZWVlZSIgc3Ryb2tlPSIjYWFhYWFhIiBjeD0iNDUuMjUiIGN5PSItNDUuMjUiIHJ4PSI0NS4wMSIgcnk9IjQ1LjAxIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjQ1LjI1IiB5PSItNDguNjUiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSIxMi4wMCI+bm90PC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0NS4yNSIgeT0iLTM1LjY1IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iMTIuMDAiPmRyYWdnaW5nPC90ZXh0Pgo8L2c+CjwhLS0gZHJhZ2dpbmcgLS0+CjxnIGlkPSJub2RlMiIgY2xhc3M9Im5vZGUiPgo8dGl0bGU+ZHJhZ2dpbmc8L3RpdGxlPgo8ZWxsaXBzZSBmaWxsPSIjZWVlZWVlIiBzdHJva2U9IiNhYWFhYWEiIGN4PSIyMjMuOTEiIGN5PSItNDUuMjUiIHJ4PSIzOS4zIiByeT0iMzkuMyIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyMjMuOTEiIHk9Ii00Mi4xNSIgZm9udC1mYW1pbHk9IkhlbHZldGljYSxzYW5zLVNlcmlmIiBmb250LXNpemU9IjEyLjAwIj5kcmFnZ2luZzwvdGV4dD4KPC9nPgo8IS0tIG5vdFxuZHJhZ2dpbmcmIzQ1OyZndDtkcmFnZ2luZyAtLT4KPGcgaWQ9ImVkZ2UxIiBjbGFzcz0iZWRnZSI+Cjx0aXRsZT5ub3RcbmRyYWdnaW5nJiM0NTsmZ3Q7ZHJhZ2dpbmc8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiM5OTk5OTkiIGQ9Ik05MC4zOCwtNDUuMjVDMTE1LjM3LC00NS4yNSAxNDYuNzIsLTQ1LjI1IDE3Mi42NiwtNDUuMjUiLz4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjOTk5OTk5IiBwb2ludHM9IjE3Mi42NCwtNDguNzUgMTgyLjY0LC00NS4yNSAxNzIuNjQsLTQxLjc1IDE3Mi42NCwtNDguNzUiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTM3LjUxIiB5PSItNDguMjUiIGZvbnQtZmFtaWx5PSJDb3VyaWVyLG1vbm9zcGFjZSIgZm9udC1zaXplPSIxMC4wMCIgZmlsbD0iIzQ0NDQyMiI+cG9pbnRlcmRvd248L3RleHQ+CjwvZz4KPCEtLSBkcmFnZ2luZyYjNDU7Jmd0O25vdFxuZHJhZ2dpbmcgLS0+CjxnIGlkPSJlZGdlMyIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+ZHJhZ2dpbmcmIzQ1OyZndDtub3RcbmRyYWdnaW5nPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjOTk5OTk5IiBkPSJNMTg1Ljg3LC0zMy45NEMxNzkuNDcsLTMyLjQxIDE3Mi44NCwtMzEuMDggMTY2LjUxLC0zMC4yNSAxNDAuOTUsLTI2LjkxIDEzNC4xMSwtMjcuMjEgMTA4LjUxLC0zMC4yNSAxMDUuODUsLTMwLjU3IDEwMy4xNSwtMzAuOTYgMTAwLjQzLC0zMS40Ii8+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzk5OTk5OSIgcG9pbnRzPSI5OS44OSwtMjcuOTQgOTAuNzEsLTMzLjIzIDEwMS4xOCwtMzQuODIgOTkuODksLTI3Ljk0Ii8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjEzNy41MSIgeT0iLTMzLjI1IiBmb250LWZhbWlseT0iQ291cmllcixtb25vc3BhY2UiIGZvbnQtc2l6ZT0iMTAuMDAiIGZpbGw9IiM0NDQ0MjIiPnBvaW50ZXJ1cDwvdGV4dD4KPC9nPgo8IS0tIGRyYWdnaW5nJiM0NTsmZ3Q7ZHJhZ2dpbmcgLS0+CjxnIGlkPSJlZGdlMiIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+ZHJhZ2dpbmcmIzQ1OyZndDtkcmFnZ2luZzwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzk5OTk5OSIgZD0iTTIwNy43MSwtODEuNDFDMjA3Ljc5LC05My4yNCAyMTMuMTksLTEwMi42NSAyMjMuOTEsLTEwMi42NSAyMzAuOTQsLTEwMi42NSAyMzUuNjgsLTk4LjYgMjM4LjE0LC05Mi40NyIvPgo8cG9seWdvbiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiM5OTk5OTkiIHBvaW50cz0iMjQxLjUzLC05My4zNiAyMzkuODQsLTgyLjkgMjM0LjY0LC05Mi4xMyAyNDEuNTMsLTkzLjM2Ii8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjIyMy45MSIgeT0iLTEwNS42NSIgZm9udC1mYW1pbHk9IkNvdXJpZXIsbW9ub3NwYWNlIiBmb250LXNpemU9IjEwLjAwIiBmaWxsPSIjNDQ0NDIyIj5wb2ludGVybW92ZTwvdGV4dD4KPC9nPgo8L2c+Cjwvc3ZnPgo=)

However it’s not so simple! Mouses have multiple buttons. Touch events can include multiple fingers. Events can go to multiple destinations. Right click can trigger the context menu. I ended up with this basic recipe:

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

Text inside draggable Multiple fingers on same draggable Nested draggable elements

Table of Contents
-----------------

Like other recipes, it’s something that works for many cases, but is **meant to be modified**. On the rest of the page I’ll show how I got here and then variants of this recipe, including [how to handle text selection](#variant-text). I also use this same recipe to handle situations where I’m _not_ dragging an object around, like dragging this number left/right: , or [painting on a canvas](https://www.redblobgames.com/making-of/draggable/targets.html#canvas-drag-to-paint), or [constrained dragging](https://www.redblobgames.com/articles/curved-paths/making-of.html).

This recipe is for the _event handling_ but there’s a second half which is _connecting_ it to some diagram state. I have a [basic example](https://jsfiddle.net/6ghmatps/)[1] showing the connecting code, and will add more examples in the future.

I’ve tested this code on Gecko/Firefox (Mac, Windows, Linux, Android), Blink/Chrome (Mac, Windows, Linux, Android), and WebKit/Safari (Mac, iPhone, iPad). I have _not_ tested on hoverable stylus, hybrid touch+mouse devices, or voice input.

Also see my other pages:

1.  [List of edge cases and list of dragging tests](https://www.redblobgames.com/making-of/draggable/targets.html)
2.  [Test page to log the events, and unorganized notes](https://www.redblobgames.com/x/2251-draggable/)
3.  [How I handled dragging events 2015–2018](https://www.redblobgames.com/x/1845-draggable/)

Note that this is _not_ the HTML [Drag and Drop API](https://www.redblobgames.com/x/2310-drag-drop-api/), which involves dragging an element _onto_ another element. For my diagrams, I’m dragging but _not_ dropping, and the scrubbable number example shows how I’m not necessarily even moving something around. So I need to read the mouse/touch events directly.

 1  [🖱️ Mouse events only](#mouse-events)[#](#mouse-events)
------------------------------------------------------------

When I first started implementing interactive diagrams ~20 years ago, touch devices weren’t common. I used `mousedown`, `mouseup`, and `mousemove` event handlers on the draggable element. If the move occurs while dragging, move the circle to the mouse position.

![](data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8IS0tIEdlbmVyYXRlZCBieSBncmFwaHZpeiB2ZXJzaW9uIDcuMS4wICgyMDIzMDEyMS4xOTU2KQogLS0+CjwhLS0gUGFnZXM6IDEgLS0+Cjxzdmcgd2lkdGg9IjI2MXB0IiBoZWlnaHQ9IjEyOHB0Igogdmlld0JveD0iMC4wMCAwLjAwIDI2MS4zMCAxMjcuNTEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPgo8ZyBpZD0iZ3JhcGgwIiBjbGFzcz0iZ3JhcGgiIHRyYW5zZm9ybT0ic2NhbGUoMSAxKSByb3RhdGUoMCkgdHJhbnNsYXRlKDQgMTIzLjUxKSI+Cjxwb2x5Z29uIGZpbGw9IndoaXRlIiBzdHJva2U9Im5vbmUiIHBvaW50cz0iLTQsNCAtNCwtMTIzLjUxIDI1Ny4zLC0xMjMuNTEgMjU3LjMsNCAtNCw0Ii8+CjwhLS0gbm90XG5kcmFnZ2luZyAtLT4KPGcgaWQ9Im5vZGUxIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5ub3RcbmRyYWdnaW5nPC90aXRsZT4KPGVsbGlwc2UgZmlsbD0iI2VlZWVlZSIgc3Ryb2tlPSIjYWFhYWFhIiBjeD0iNDUuMjUiIGN5PSItNDUuMjUiIHJ4PSI0NS4wMSIgcnk9IjQ1LjAxIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjQ1LjI1IiB5PSItNDguNjUiIGZvbnQtZmFtaWx5PSJIZWx2ZXRpY2Esc2Fucy1TZXJpZiIgZm9udC1zaXplPSIxMi4wMCI+bm90PC90ZXh0Pgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0NS4yNSIgeT0iLTM1LjY1IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iMTIuMDAiPmRyYWdnaW5nPC90ZXh0Pgo8L2c+CjwhLS0gbm90XG5kcmFnZ2luZyYjNDU7Jmd0O25vdFxuZHJhZ2dpbmcgLS0+CjxnIGlkPSJlZGdlMSIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+bm90XG5kcmFnZ2luZyYjNDU7Jmd0O25vdFxuZHJhZ2dpbmc8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiM5OTk5OTkiIGQ9Ik0yOC45OCwtODcuNTNDMjkuNjksLTk5LjQgMzUuMTEsLTEwOC41MSA0NS4yNSwtMTA4LjUxIDUxLjkxLC0xMDguNTEgNTYuNTMsLTEwNC41OSA1OS4xMywtOTguNTMiLz4KPHBvbHlnb24gZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjOTk5OTk5IiBwb2ludHM9IjYyLjQ5LC05OS41MiA2MS4yMSwtODkuMDEgNTUuNjYsLTk4LjAzIDYyLjQ5LC05OS41MiIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0NS4yNSIgeT0iLTExMS41MSIgZm9udC1mYW1pbHk9IkNvdXJpZXIsbW9ub3NwYWNlIiBmb250LXNpemU9IjEwLjAwIiBmaWxsPSIjNDQ0NDIyIj5tb3VzZW1vdmU8L3RleHQ+CjwvZz4KPCEtLSBkcmFnZ2luZyAtLT4KPGcgaWQ9Im5vZGUyIiBjbGFzcz0ibm9kZSI+Cjx0aXRsZT5kcmFnZ2luZzwvdGl0bGU+CjxlbGxpcHNlIGZpbGw9IiNlZWVlZWUiIHN0cm9rZT0iI2FhYWFhYSIgY3g9IjIxMy45MSIgY3k9Ii00NS4yNSIgcng9IjM5LjMiIHJ5PSIzOS4zIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjIxMy45MSIgeT0iLTQyLjE1IiBmb250LWZhbWlseT0iSGVsdmV0aWNhLHNhbnMtU2VyaWYiIGZvbnQtc2l6ZT0iMTIuMDAiPmRyYWdnaW5nPC90ZXh0Pgo8L2c+CjwhLS0gbm90XG5kcmFnZ2luZyYjNDU7Jmd0O2RyYWdnaW5nIC0tPgo8ZyBpZD0iZWRnZTIiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPm5vdFxuZHJhZ2dpbmcmIzQ1OyZndDtkcmFnZ2luZzwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iIzk5OTk5OSIgZD0iTTkwLjQ4LC00NS4yNUMxMTIuNzUsLTQ1LjI1IDEzOS44NCwtNDUuMjUgMTYyLjg4LC00NS4yNSIvPgo8cG9seWdvbiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiM5OTk5OTkiIHBvaW50cz0iMTYyLjY0LC00OC43NSAxNzIuNjQsLTQ1LjI1IDE2Mi42NCwtNDEuNzUgMTYyLjY0LC00OC43NSIvPgo8dGV4dCB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxMzIuNTEiIHk9Ii00OC4yNSIgZm9udC1mYW1pbHk9IkNvdXJpZXIsbW9ub3NwYWNlIiBmb250LXNpemU9IjEwLjAwIiBmaWxsPSIjNDQ0NDIyIj5tb3VzZWRvd248L3RleHQ+CjwvZz4KPCEtLSBkcmFnZ2luZyYjNDU7Jmd0O25vdFxuZHJhZ2dpbmcgLS0+CjxnIGlkPSJlZGdlNCIgY2xhc3M9ImVkZ2UiPgo8dGl0bGU+ZHJhZ2dpbmcmIzQ1OyZndDtub3RcbmRyYWdnaW5nPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSIjOTk5OTk5IiBkPSJNMTc1Ljg3LC0zMy45NEMxNjkuNDcsLTMyLjQxIDE2Mi44NCwtMzEuMDggMTU2LjUxLC0zMC4yNSAxMzUuMzYsLTI3LjQ5IDEyOS42OSwtMjcuNzQgMTA4LjUxLC0zMC4yNSAxMDUuODUsLTMwLjU3IDEwMy4xNSwtMzAuOTYgMTAwLjQzLC0zMS40Ii8+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzk5OTk5OSIgcG9pbnRzPSI5OS44OSwtMjcuOTQgOTAuNzEsLTMzLjIzIDEwMS4xOCwtMzQuODIgOTkuODksLTI3Ljk0Ii8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjEzMi41MSIgeT0iLTMzLjI1IiBmb250LWZhbWlseT0iQ291cmllcixtb25vc3BhY2UiIGZvbnQtc2l6ZT0iMTAuMDAiIGZpbGw9IiM0NDQ0MjIiPm1vdXNldXA8L3RleHQ+CjwvZz4KPCEtLSBkcmFnZ2luZyYjNDU7Jmd0O2RyYWdnaW5nIC0tPgo8ZyBpZD0iZWRnZTMiIGNsYXNzPSJlZGdlIj4KPHRpdGxlPmRyYWdnaW5nJiM0NTsmZ3Q7ZHJhZ2dpbmc8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9IiM5OTk5OTkiIGQ9Ik0xOTguNjUsLTgxLjk3QzE5OC44OSwtOTMuNTMgMjAzLjk3LC0xMDIuNjUgMjEzLjkxLC0xMDIuNjUgMjIwLjI3LC0xMDIuNjUgMjI0LjY1LC05OC45MSAyMjcuMDMsLTkzLjE4Ii8+Cjxwb2x5Z29uIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzk5OTk5OSIgcG9pbnRzPSIyMzAuNDUsLTkzLjkzIDIyOC44OCwtODMuNDUgMjIzLjU3LC05Mi42MiAyMzAuNDUsLTkzLjkzIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjIxMy45MSIgeT0iLTEwNS42NSIgZm9udC1mYW1pbHk9IkNvdXJpZXIsbW9ub3NwYWNlIiBmb250LXNpemU9IjEwLjAwIiBmaWxsPSIjNDQ0NDIyIj5tb3VzZW1vdmU8L3RleHQ+CjwvZz4KPC9nPgo8L3N2Zz4K)

```
function makeDraggableMouseLocal(state, el) {
    function mousedown(_event) {
        state.dragging = true;
    }
    function mouseup(_event) {
        state.dragging = null;
    }
    function mousemove(event) {
        if (!state.dragging) return;
        state.pos = state.eventToCoordinates(event);
    }
    
    el.addEventListener('mousedown', mousedown);
    el.addEventListener('mouseup', mouseup);
    el.addEventListener('mousemove', mousemove);
}

```

**Try** the demo with a mouse:

Drag me Drag using mouse events on circle

This might seem like it works but it works poorly.

*   If you move the pointer quickly it is no longer over the circle, it stops receiving events.
*   If you release the button while not on the circle, it will get stuck in the “dragging” state.

To fix these problems use `mousedown` on the circle to add `mousemove` and `mouseup` on the _document_. Then on `mouseup` remove the `mousemove` and `mouseup` from the document.

![](https://www.redblobgames.com/making-of/draggable/build/mouse-document.svg)

```
function makeDraggableMouseGlobal(state, el) {
    function globalMousemove(event) {
        state.pos = state.eventToCoordinates(event);
        state.dragging = true;
    }
    function globalMouseup(_event) {
        document.removeEventListener('mousemove', globalMousemove);
        document.removeEventListener('mouseup', globalMouseup);
        state.dragging = null;
    }
    
    function mousedown(event) {
        document.addEventListener('mousemove', globalMousemove);
        document.addEventListener('mouseup', globalMouseup);
    }
    
    el.addEventListener('mousedown', mousedown);
}

```

Try it out. It works better.

Drag me Drag using mouse events on document

This code doesn’t handle touch events.

 2  [👆 Touch events](#touch-events)[#](#touch-events)
------------------------------------------------------

Mouse events use `mousedown`, `mouseup`, `mousemove`. Touch events instead use `touchstart`, `touchend`, `touchmove`. They behave a little differently. Touch events automatically _capture_ on `touchstart` and direct all `touchmove` events to the original element. This means we _don’t_ have to temporarily put an event handler on `document`. We can go back to the simpler logic in the first mouse example. If for any reason the browser needs to cancel the touch sequence, it sends `touchcancel`.

![](https://www.redblobgames.com/making-of/draggable/build/touch.svg)

```
function makeDraggableTouch(state, el) {
    function start(event) {
        event.preventDefault(); 
        state.dragging = true;
    }
    function end(_event) {
        state.dragging = false;
    }
    function move(event) {
        if (!state.dragging) return;
        state.pos = state.eventToCoordinates(event.changedTouches[0]);
    }
    
    el.addEventListener('touchstart', start);
    el.addEventListener('touchend', end);
    el.addEventListener('touchcancel', end);
    el.addEventListener('touchmove', move);
}

```

**Try** the demo with a touch device:

Drag me Drag using touch events

This code doesn’t handle mouse events.

 3  [🖱️👆 Pointer events](#pointer-events)[#](#pointer-events)
---------------------------------------------------------------

Handling both mouse and touch events requires lots of event handlers, and that’s what I used before 2021. Details→

From 2011 to 2014 I used [d3-drag](https://github.com/d3/d3-drag)[2] in projects where I used d3. For my non-d3 projects, I ended up developing my own mouse+touch code, which I wrote about [in 2018](https://www.redblobgames.com/x/1845-draggable/).

By 2012 MS IE had added support for [pointer events](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events)[3] which unify and simplify mouse+touch handling. [Chrome added support in 2017; Firefox in 2018; Safari in 2020](https://caniuse.com/pointer)[4].

Over the years browsers have changed the rules, including in 2017 when [Chrome changed some events to default to passive mode](https://developer.chrome.com/blog/scrolling-intervention/)[5] which causes the page to scroll while trying to drag the object. This [broke some pages](https://github.com/WICG/interventions/issues/18#issuecomment-276531695)[6]. Safari [made this change in 2018](https://github.com/WICG/interventions/issues/18#issuecomment-368703063)[7]. Firefox also [made this change in 2018](https://bugzilla.mozilla.org/show_bug.cgi?id=1449268)[8].

![](https://www.redblobgames.com/making-of/draggable/build/mouse-and-touch.svg)

Pointer events attempt to unify mouse and touch events. The [pointer capture](https://developer.mozilla.org/en-US/docs/Web/API/Element/setPointerCapture)[9] feature lets us use the simpler logic that doesn’t require us to add/remove global event handlers to the document like we had to with mouse events.

![](https://www.redblobgames.com/making-of/draggable/build/pointer.svg)

This recipe is the starting point:

```
function makeDraggable(state, el) {
    function start(event) {
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = true;
    }

    function end(event) {
        state.dragging = false;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x, y};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
}

```

Much simpler! However, I almost always want to handle some extras, so I start with this instead:

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

**Try** the demo with either a mouse or touch device:

Drag me Drag using pointer events

Let’s look at each of the extras.

###  3.1. 🖱️ Fix: capture the mouse[#](#fix-capture)

The pointer capture feature lets us track the pointer even when it’s not on the circle, the diagram, or even the browser window. With mouse events we had to put event handlers on `document`, but pointer capture is simpler.

<table><colgroup><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th></tr></thead><tbody><tr><td>drag quickly back and forth</td><td>drag stops</td><td>yes ⛌</td><td>no ✓</td></tr><tr><td>drag outside diagram, come back in</td><td>drag stops</td><td>yes ⛌</td><td>no ✓</td></tr><tr><td>drag outside diagram, let go</td><td>drag stops</td><td>no ⛌</td><td>yes ✓</td></tr><tr><td>drag outside diagram, let go, come back in</td><td>drag stops</td><td>no ⛌</td><td>yes ✓</td></tr><tr><td>drag, alt+tab to another window</td><td>drag stops</td><td>no ⛌</td><td>yes ✓</td></tr></tbody></table>Drag 1 Drag 2 Dragging without and with pointer capture

**Try** this demo with a mouse.

*   Circle 1 doesn’t use pointer capture on mouse. Pointer capture is the default on touch devices.
*   Circle 2 uses pointer capture on both mouse and touch devices.

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

###  3.2. 👆 Fix: scrolling with touch[#](#fix-scroll)

On touch devices, single-finger drag will scroll the page. But single-finger drag _also_ drags the circle. By default, it will do _both_! The simplest fix is to add CSS touch-action: none on the diagram. But this prevents scrolling _anywhere_ in the diagram:

Drag 1 Stop touch from scrolling anywhere on the diagram

**Try** dragging the circle on a touch device. It shouldn’t scroll. But then try scrolling by dragging the diagram. It doesn’t scroll either, but I want it to. I want to stop scrolling _only_ if dragging the circle, not when dragging the diagram.

<table><colgroup><col><col><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th><th scope="col">Circle 3</th><th scope="col">Circle 4</th></tr></thead><tbody><tr><td>drag circle</td><td>page scrolls</td><td>no ✓</td><td>yes ⛌</td><td>yes ⛌</td><td>no ✓</td></tr><tr><td>drag diagram</td><td>page scrolls</td><td>no ⛌</td><td>yes ✓</td><td>yes ✓</td><td>yes ✓</td></tr></tbody></table>Drag 2 Drag 3 Drag 4 Dragging affects scrolling

**Try** these on a touch device.

*   Circle 1 (touch-action: none on the diagram) stops scrolling on the circle and also on the diagram.
*   Circle 2 (default) doesn’t stop scrolling on either.
*   Circle 3 (touch-action: none on the circle only) behaves badly. It looks like the CSS has to be on the diagram to have an effect; applying it only to the circle is not enough.
*   Circle 4 (.preventDefault() on `touchstart`) behaves the way I want, and this is the code for it:

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

I use the .preventDefault() solution. Note that it needs to be on `touchstart`, not on `pointerstart`. It works in most situations but I’ve run into situations where it did not stop scrolling on certain systems.

###  3.3. 🖱️ Feature: handle drag offset[#](#feature-offset)

This isn’t necessary but it makes dragging feel nicer. If you pick up the edge of an object then you want to keep holding it at _that_ point, not from the center of the object. The solution is to remember where the center is relative to where the drag started. Then when moving the object, add that offset back in.

<table><colgroup><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th></tr></thead><tbody><tr><td>drag from edge of circle</td><td>circle jumps</td><td>yes ⛌</td><td>no ✓</td></tr></tbody></table>Drag 1 Drag 2 Dragging feels better if relative to the initial pickup point

**Try** with the mouse: drag the circle from the edge. Watch Circle 1 jump whereas Circle 2 does not. The same effect happens on touch devices but your finger might hide the jump. The fix is to change the `dragging` state from `true` / `false` to the relative position where the object was picked up, and then use that offset when later setting the position:

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

Tracking the offset makes dragging feel better. I’ve also written about this [on my page about little details](https://www.redblobgames.com/making-of/little-things/#drag-point).

Context menus are different across platforms, and that makes handling it tricky. I want to allow context menus without them interfering with dragging the circle.

<table><colgroup><col><col></colgroup><thead><tr><th scope="col">System</th><th scope="col">Activation</th></tr></thead><tbody><tr><td>Windows</td><td>right click (down+up), <kbd>Shift</kbd> + <kbd>F10</kbd> key</td></tr><tr><td>Linux</td><td>right button down, <kbd>Shift</kbd> + <kbd>F10</kbd> key</td></tr><tr><td>Mac</td><td>right button down, <kbd>Ctrl</kbd> + left click</td></tr><tr><td>iOS</td><td>long press on text only</td></tr><tr><td>Android</td><td>long press on anything</td></tr></tbody></table>

One problem is that I will see a `pointerdown` event and only _sometimes_ a `pointerup` event. That means I might think the button is still down when it’s not. It’s frustrating! I realized that I should only set the dragging state on _left_ mouse button, and ignore the right mouse button. Then I don’t have to worry about most of the differences.

I made some notes during testing, but most of them don’t matter for my use case.

Across platforms, it looks like Firefox lets the page see events outside the menu overlay, whereas Chrome doesn’t let the page see any events while the menu is up.

Windows, right click, no capture:

Firefox, Chrome, Edge

`pointerdown`, `pointerup`, `auxclick`, `contextmenu`

Windows, right click, capture:

Firefox

`pointerdown`, `gotpointercapture`, `pointerup`, `lostpointercapture`, `auxclick`, `contextmenu`

Chrome, Edge

`pointerdown`, `gotpointercapture`, `pointerup`, `auxclick`, `lostpointercapture`, `contextmenu`

Linux right click, no capture:

Firefox

`pointerdown`, `contextmenu`, `pointermove` while menu is up

Chrome

`pointerdown`, `contextmenu`, no `pointermove` while menu is up

Linux hold right down, no capture:

Firefox

`pointerdown`, `contextmenu`, `pointermove` while menu is up

Chrome

`pointerdown`, `contextmenu`, no `pointermove` while menu is up

Linux right click, capture:

Firefox

`pointerdown`, `contextmenu`, `gotpointercapture`, `pointermove` while menu is up tells us button released

Chrome

`pointerdown`, `contextmenu`, `gotpointercapture`; not until another click do we get `pointerup`, `lostpointercapture`

Linux hold right down, capture:

Firefox

`pointerdown`, `contextmenu`, `gotpointercapture`, `pointermove` while menu is up tells us button released; when releasing button, menu stays up but we get `pointerup`, `lostpointercapture`

Chrome

`pointerdown`, `contextmenu`, `gotpointercapture`, no `pointermove` while menu is up; when releasing button, menu stays up but we don’t get `pointerup`; not until another click do we get `pointerup`, click, `lostpointercapture`

Mac, ctrl + left click:

Firefox

`pointermove` with buttons≠0, `contextmenu` (no `pointerdown` or `pointerup`)

Chrome

`pointerdown` with button=left, `contextmenu` (no `pointerup`)

Safari

`pointerdown` with button=left, `contextmenu` (no `pointerup`); but subsequent clicks only fire `contextmenu`

Mac, right button down:

Firefox

`pointerdown` with button=right, `contextmenu` (no `pointerup`)

Chrome

`pointerdown` with button=right, `contextmenu` (no `pointerup`)

Safari

`pointerdown` with button=right, `contextmenu` (no `pointerup`); but subsequent right clicks only fire `contextmenu`

If we capture events on `pointerdown`, Firefox and Safari will keep the capture even after the button is released. Chrome will keep capture until you move the mouse, and then it will release capture. [This seems like a Firefox/Safari bug to me, as pointer capture is supposed to be automatically released on mouse up]

It’s frustrating that on Mac, there’s no `pointerup` or `pointercapture` when releasing the mouse button. On Linux, the `pointerup` only shows up if you click to exit the context menu. It doesn’t show up if you press Esc to exit. The workaround is to watch `pointermove` events to see when no buttons are set. Windows doesn’t seem to have these issues, as both `pointerdown` and `pointerup` are delivered before the context menu.

Android, long press:

Firefox

`pointerdown`, get capture, `contextmenu`, `pointerup`, lose capture

Chrome

`pointerdown`, get capture, `contextmenu`, `pointerup` or `pointercancel` (if the finger moves at all, this starts a scroll event which cancels the captured pointer), lose capture

What are my options?

*   [The spec says about pointerdown](https://www.w3.org/TR/pointerevents/#the-pointerdown-event)[10] that `preventDefault()`_not_ stop click or `contextmenu` events. I can `preventDefault()` on `contextmenu` to prevent the menu. But I still want to get `pointerup` and/or `pointercancel`! I think I have to treat `contextmenu` as the up event which means I’ll get multiple up events on Windows.
*   [The spec says about the button property](https://w3c.github.io/pointerevents/#the-button-property)[11] that `button` = 0 indicates the primary button. This is how I will exclude the middle and right buttons. But I still get a `pointerdown.left` on Mac/Chrome and Mac/Safari (but not on Mac/Firefox) so I also have to check for the Ctrl key.
*   Button changes not communicated through `pointerdown` or `pointerup` can still be sent on `pointermove`. It’s mentioned as a workaround on [W3C’s pointerevents issues page](https://github.com/w3c/pointerevents/issues/408)[12].

<table><colgroup><col><col><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th><th scope="col">Circle 3</th><th scope="col">Circle 4</th></tr></thead><tbody><tr><td>right click</td><td>circle turns blue</td><td>yes ⛌</td><td>yes</td><td>no ✓</td><td>no ✓</td></tr><tr><td>right click</td><td>context menu</td><td>yes ⛌</td><td>no</td><td>no ✓</td><td>no ✓</td></tr><tr><td>middle click</td><td>circle turns blue</td><td>yes ⛌</td><td>yes ⛌</td><td>no ✓</td><td>no ✓</td></tr><tr><td>right drag</td><td>circle is blue</td><td>yes ⛌</td><td>yes</td><td>no ✓</td><td>no ✓</td></tr><tr><td>middle drag</td><td>circle is blue</td><td>yes ⛌</td><td>yes</td><td>no ✓</td><td>no ✓</td></tr><tr><td>ctrl+click (mac)</td><td>circle turns blue¹</td><td>yes ⛌</td><td>no ✓</td><td>yes ⛌</td><td>no ✓</td></tr><tr><td>ctrl+click (mac)</td><td>context menu</td><td>yes ⛌</td><td>no ✓</td><td>yes ⛌</td><td>no ✓</td></tr></tbody></table>

¹ it will turn blue in Chrome and Safari but not in Firefox, which treats Ctrl + click differently

**Try** with the mouse: right click or drag on the circles. Try dismissing the menu with a click elsewhere, or by pressing Esc. Behavior varies across browsers and operating systems.

*   Circle 1 sometimes get stuck in a dragging state.
*   Circle 2 uses .preventDefault() on `contextmenu`. This allows the right button to be used for dragging. However, it interferes with the default operation of middle click or drag, which is used for scrolling on some systems.
*   Circle 3 drags only with the left button, but on Mac Ctrl + click on Chrome/Safari will trigger drag.
*   Circle 4 drags only with the left button, if Ctrl isn’t pressed.

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        if (event.ctrlKey) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = true;
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = false;
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x, y};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

 4  [🖱️ Variant: draggable text/images](#variant-text)[#](#variant-text)
-------------------------------------------------------------------------

These changes are needed if you have text or images inside your draggable element:

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
        el.style.userSelect = 'none'; 
        el.style.webkitUserSelect = 'none'; 
    }

    function end(event) {
        state.dragging = null;
        el.style.userSelect = ''; 
        el.style.webkitUserSelect = ''; 
    }

    function move(event) {
        if (!state.dragging) return;
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
    el.addEventListener('dragstart', (e) => e.preventDefault());
}

```

###  4.1. 🖱️ Fix: text selection[#](#fix-user-select)

When dragging the circle, the text inside gets selected sometimes. To fix this, use CSS user-select: none on the circle. There are two choices: either we can apply it _all_ the time, or apply it _only_ while dragging. If I apply it all the time, then the text won’t ever be selectable.

<table><colgroup><col><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th><th scope="col">Circle 3</th></tr></thead><tbody><tr><td>drag circle</td><td>text is selected</td><td>yes ⛌</td><td>no ✓</td><td>no ✓</td></tr><tr><td>select all text</td><td>text is selected</td><td>yes</td><td>no</td><td>yes</td></tr></tbody></table>Drag text 1 Drag text 2 Drag text 3 Dragging affects text selection

**Try** dragging quickly with the mouse. **Try** selecting all text on the page to see whether the text inside the circle is selectable when not dragging.

*   Circle 1 never uses user-select: none
*   Circle 2 always uses user-select: none
*   Circle 3 uses user-select: none only when dragging

I think either Circle 2 or Circle 3’s behavior is a reasonable choice. Note that as of early 2023, Safari _still_  [doesn’t support the unprefixed version](https://caniuse.com/?search=user-select)[13] ([tracking bug](https://bugs.webkit.org/show_bug.cgi?id=208677)[14]), so we have to also set the prefixed version.

###  4.2. 🖱️ Fix: text and image drag[#](#fix-systemdrag)

Windows, Linux, and Mac support inter-application _drag and drop_ of text and images, and an alternative to copy/paste. This interferes with the object dragging on my pages. The fix is to preventDefault() on `dragstart`.

<table><colgroup><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th></tr></thead><tbody><tr><td>select text, drag circle</td><td>page text drags</td><td>yes ⛌</td><td>no ✓</td></tr></tbody></table>

**Select text** →

[from here 1 2 to here]

←

Selected text interferes with dragging

**Try** this demo with a mouse. around the diagram, then drag Circle 1. On most desktop systems I’ve tested, text or image dragging takes priority over the circle dragging by default. Circle 2 prioritizes the circle dragging. Behavior varies a little bit across browsers and operating systems. The fix is one extra line:

 5  [More cases](#more-cases)[#](#more-cases)
---------------------------------------------

###  5.1. 👆 Feature: simultaneous dragging[#](#feature-simultaneous-dragging)

I think this is an edge case, but I was curious what it would take to support. Can we drag multiple objects at once, using different fingers or different mice?

For touch, the code I presented should already work! Go back to one of the previous demos and try it. However the code doesn’t handle using two fingers to drag the _same_ object. The fix is when handling `pointerdown`, save event.pointerId to `state.dragging`. Then when handling `pointermove`, ignore the even if it’s not the same `pointerId`. I don’t have that implemented here, but try it out [on my canvas dragging test](https://www.redblobgames.com/making-of/draggable/targets.html#canvas-drag-a-handle).

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        state.pointerId = event.pointerId; 
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        if (state.pointerId !== event.pointerId) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

What about mice? The [Pointer Events spec](https://www.w3.org/TR/pointerevents/#the-primary-pointer)[15] says

> Current operating systems and user agents don’t usually have a concept of multiple mouse inputs. When more than one mouse device is present (for instance, on a laptop with both a trackpad and an external mouse), all mouse devices are generally treated as a single device - movements on any of the devices are translated to movement of a single mouse pointer, and there is no distinction between button presses on different mouse devices. For this reason, there will usually only be a single mouse pointer, and that pointer will be primary.

I think there isn’t any way to drag different objects with different mice.

###  5.2. 🖱️ Edge case: chorded button presses[#](#fix-chords)

So here’s a tricky one. If you are using multiple buttons at the same time, what happens? Mouse Events send `mousedown` for each button press and `mouseup` for each button release. But Pointer Events work differently. The [Pointer Events spec](https://www.w3.org/TR/pointerevents/#chorded-button-interactions)[16] says that the _first_ button that was pressed leads to a `pointerdown` event, and the _last_ one that was released leads to a `pointerup` event. But that means we might get a up event on a different button than the down event!

![](https://www.redblobgames.com/making-of/draggable/build/multiple-buttons.svg)<table><colgroup><col><col><col><col></colgroup><thead><tr><th scope="col">Try this</th><th scope="col">Watch for</th><th scope="col">Circle 1</th><th scope="col">Circle 2</th></tr></thead><tbody><tr><td>left down, right down, left up</td><td>dragging</td><td>yes ⛌</td><td>no ✓</td></tr></tbody></table>Drag 1 Drag 2 Multiple button presses is tricky

**Try** with the mouse: press the left button, press the right button (this may bring up a context menu but ignore it), then release the left button. Is the circle still dragging?

The fix is to check the button state in `pointermove`:

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        if (!(event.buttons & 1)) return end(event); 
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

Separately, the _pointer capture_ continues until you release _all_ the buttons, unless you explicitly release capture. I’m not handling this or many other edge cases.

###  5.3. 🖱️👆 Variant: nested dragging[#](#variant-nesting)

If the draggable element contains _another_ draggable element inside of it, both elements will handle the dragging. The fix is to add .stopPropagation() to prevent the inner draggable from passing events up to outer draggable. I don’t have a demo here, but [I made one elsewhere](https://play.vuejs.org/#eNq1Vk1zmzAQ/SsqkxnsqUOcyY06nbZpDumhzbQ5JDMcQswalICkkYSNS/nvXUngQOLaTT/sA9Lbt2L37bJQe++FCJYleKE3U3NJhSYKdCneRowWgktNaiJhQRqykLwgPlL9iEVszpnSpCKnxjqajjtk/QxZ0URnLXo8fcQzoGmmB4bOlMg4RUMdMWLXKWVpSBZxrmBiMMEp0yATvmIjWALTY8clRGdUBZ2LOSKpQlIFyzgvgRwSSw7mOcXL9YQk65CstxpvmjfuwBYspcTrVSxT0AEKdOkiOIuFLiW4III2rItkbJ2bfqyl2BOpze65X8GX7fEbT7ogo1cD9zFqiHGwNuYu39NhvuT18KZBUrUOnQZDh5vnDutNgE1XKwmKfjeu/6Raq9D1yy8rloVt4/y3snXOSnNxKbmI01hTzkZ/V9QXnPonJd91OBlIurcpVq3TQOa9nZH1O2N25EYJDhHcaChEHmvAHSGzhC5JqPQ6h9PIq7WMmVpwWYTk1q4NcXRQV42oJgf1Gi/j24lLACkHtV0hiqAL0KJu6eC7eP6QSl6yJCT+GvKcr/wJ8UsF8lBBDnPto4FxBggLrqiRChEJeG+6NCj2jeISMVMKv4k8pwj+3vWaOchhoTEJI0LXRAbfQi/FE14ptrDM3Z7wDNRnal7OM6VjiW0rbUnQoUewGhPy0QzQAkjt9PoxbZrrulXJbBzrSS16WsR3iuelNlpIJ7I/xfUd15pjqeymrYl/PBUVbrtqdPtBGSQkfVnZSsGhGxx71XW039F3yNyl8JD5Uo03Ks+OUEDX1e1qdtRrdvf3Jp5WOCoXNA3uFWf4orUPdeTNeSFoDvKLMKqryAu7xz3yYtO2nyymZenmqPXJYP6wBb9XlcEi7xJzA2kS2tgwExx/znz+7TNUuN4YC56UObJ3GL+CbQaM0dE+YE0x7B7PRnthPxdwGFyp80oDU11SJtDHIRd5+AlxtiP1x3BPghPrhzPFa34CpcvwOQ==)[17], where the red draggable is a child of the yellow draggable.

```
function makeDraggable(state, el) {
    function start(event) {
        if (event.button !== 0) return; 
        event.stopPropagation(); 
        let {x, y} = state.eventToCoordinates(event);
        state.dragging = {dx: state.pos.x - x, dy: state.pos.y - y};
        el.setPointerCapture(event.pointerId);
    }

    function end(event) {
        state.dragging = null;
    }

    function move(event) {
        if (!state.dragging) return;
        event.stopPropagation(); 
        let {x, y} = state.eventToCoordinates(event);
        state.pos = {x: x + state.dragging.dx, y: y + state.dragging.dy};
    }
        
    el.addEventListener('pointerdown', start);
    el.addEventListener('pointerup', end);
    el.addEventListener('pointercancel', end);
    el.addEventListener('pointermove', move)
    el.addEventListener('touchstart', (e) => e.preventDefault());
}

```

###  5.4. 🖱️👆 Variant: dragging on canvas[#](#variant-canvas)

I normally work with SVG, but if working with a `<canvas>` (either 2D Canvas or WebGL), I can’t set the event handlers or mouse pointer shape on the _draggable_ element only. So I set the event handler on the `<canvas>` and then:

1.  `pointerdown`, `touchstart`, `dragstart`: early return if not over a draggable object
2.  `pointermove`: set the cursor based on whether it’s over a draggable object

I have a [demo](https://www.redblobgames.com/making-of/draggable/targets.html#canvas-drag-to-paint) but haven’t written up the code yet.

###  5.5. Variant: hover with mouse[#](#variant-hover)

Sometimes I want to act on _hover_ with the mouse (no buttons pressed) but that doesn’t work with touch devices, so I use _drag_ with touch. The standard recipe at the top of the page expects drag for both mouse and touch. To make it hover with mouse and drag for touch, I modify the recipe by removing the if (!state.dragging) line from `pointermove`.

For a demo, see my [Responsive Design page](https://www.redblobgames.com/making-of/responsive-design/#visualizations)[18]. With a mouse, you can hover over the rows to change the layout. With touch, you can drag to change the layout. I also do this on the [Hexagons Guide](https://www.redblobgames.com/grids/hexagons/). Many diagrams work with mouse hover, but on touch devices they work with touch drag.

Do I need to release the capture? **Yes**, sometimes. When moving the mouse from item A to item B, if I want to highlight A and then highlight B, hover works as expected. But with touch, dragging from A to B sends the move event to A. By releasing the capture, the move event will go to B.

###  5.6. Variant: toggle paint

For the [Rounded Cell Painter](https://www.redblobgames.com/x/1825-rounded-cell-painter/)[19], the `pointerdown` captures the initial paint color, and then all `pointermove` events will use that same paint color, until `pointerup`. To do this, the `state.dragging` variable should contain not only the initial `x`, `y` but also the initial paint color.

###  5.7. TODO: lostpointercapture

I know that `lostpointercapture` can be used to detect that we lost pointer capture, but I haven’t yet figured out all the situations in which it fires, and what I should do about them.

 6  [Vue component](#vue)[#](#vue)
----------------------------------

Here’s an attempt at a `<Draggable>` component in Vue v3, Options API:

```
<template>
  <g
    :transform="`translate(${modelValue.x},${modelValue.y})`"
    @pointerdown.left="start"
    @pointerup="end" @pointercancel="end"
    @pointermove="dragging ? move($event) : null"
    @touchstart.prevent="" @dragstart.prevent=""
    :class="{dragging}">
    <slot />
  </g>
</template>

<script>
  import {convertPixelToSvgCoord} from './svgcoords.js';
  
  export default {
    props: {modelValue: Object}, // should be {x, y}
    data() { return { dragging: false }; },
    methods: {
      start(event) {
        if (event.ctrlKey) return;
        let {x, y} = convertPixelToSvgCoord(event);
        this.dragging = {dx: this.modelValue.x - x, dy: this.modelValue.y - y};
        event.currentTarget.setPointerCapture(event.pointerId);
      },
      end(_event) {
        this.dragging = null;
      },
      move(event) {
        let {x, y} = convertPixelToSvgCoord(event);
        this.$emit('update:modelValue', {
          x: x + this.dragging.dx,
          y: y + this.dragging.dy,
        });
      },
    }    
  }
</script>

<style scoped>
  g { cursor: grab; }
  g.dragging { user-select: none; cursor: grabbing; }
</style>


```

and here’s the same thing in the Composition API:

```
<template>
  <g
    :transform="`translate(${modelValue.x},${modelValue.y})`"
    @pointerdown.left="start"
    @pointerup="end" @pointercancel="end"
    @pointermove="dragging ? move($event) : null"
    @touchstart.prevent="" @dragstart.prevent=""
    :class="{dragging}">
    <slot />
  </g>
</template>

<script setup>  
  import {ref} from 'vue';
  import {convertPixelToSvgCoord} from './svgcoords.js';
  
  const props = defineProps({
    modelValue: Object // should be {x, y}
  });
  
  const emit = defineEmits(['update:modelValue'])

  const dragging = ref(false);

  function start(event) {
    if (event.ctrlKey) return;
    let {x, y} = convertPixelToSvgCoord(event);
    dragging.value = {dx: props.modelValue.x - x, dy: props.modelValue.y - y};
    event.currentTarget.setPointerCapture(event.pointerId);
  }

  function end(event) {
    dragging.value = null;
  }

  function move(event) {
    let {x, y} = convertPixelToSvgCoord(event);
    emit('update:modelValue', {
      x: x + dragging.value.dx,
      y: y + dragging.value.dy,
    });
  }
</script>

<style scoped>
  g { cursor: grab; }
  g.dragging { user-select: none; cursor: grabbing; }
</style>


```

This component only handles SVG elements, as that’s my primary need. You’ll have to modify it if you’re dragging HTML elements. You can use _computed setters_ on the model value if you want to apply constraints like bounds checking or grid snapping. If you’re using Vue v2 (whether Options API or Composition API), you’ll need to change the v-model prop from `modelValue` to `value` and the event from `update:modelValue` to `input`.

To use this component, create position variables (`data` in Options API or `ref` in Composition API) of type `{x, y}`, and then draw the desired shape as the child slot of the draggable component. In this example, I have `redCircle` and `blueSquare` positions:

```
<svg viewBox="-100 -100 200 200">
  <Draggable v-model="redCircle">
    <circle r="10" fill="hsl(0 50% 50%)" />
  </Draggable>
  <Draggable v-model="blueSquare">
    <rect x="-10" y="-10" width="20" height="20" fill="hsl(220 50% 50%)" />
  </Draggable>
</svg>


```

I put a full example of this [on the Vue Playground](https://play.vuejs.org/#eNrtV2tv2zYU/SsXXofImS2lAfbFSfbyimEYhgZIUGyYh1WWrmQ2kqiRlB8z/N93SD0sO26WtRj2pQZsi+S5l/d5RG4H35alv6x4MBkE58TrMC8zJplQrMKUwiLGgywpkYr0MiXOOOfC6KtZQURRWNCcKZexSATHZCStpHqglTALWpg86/BYkrWMZsaSKfUkCFarla84nmdynoY5az+SeZCHD6JIxzIJrAlpOM84oPNgVsyKax0pURroMFX5lVUn8lIqQ1vFyY4SJXM6gzNnbqsgaJe/bxU1ED/oZl6XRshC2xDUUv8sMpWAaGHl9mL2G0GRITg0FSqC5A2eE2+7ntCXFyPaTGh8ebEbOngNnWcV3/1ZhaqPHTfgBnsd1E7DXQwMIz+hYef8tc3IUvDqO7m+mQ3GLy8urDQkLwjPswFps8kYS/MwekiVrIp4Qp8x82zgFEDF3s/lGHnkDOjOgw4GYFT7pLD+0qpORGaxC515F/Dvc/sdYj5oNe8D9tRe+xD0N1McGWp8gs5N97QSsVlgdGkHCxbpwrSjnkGXl88yCaFdpni6DnphHYwG78s0euQoAWmteGJUWGj0SA4D3rqBxXgvts7NNyGc9Ne70cF4sxu+nQ1qBd+UUhSGVSxXhZ9xYp3SJlTmGFCVWOEihlftFJowcqF004fwXC5t+l0joanoa/TqEnbxEk05pAkVVZZ1QkZW0cJt65fKQSBrd7Lyj+cb36Ms1BoT23aX3b66dCZNE/7r4FGojxu67qKnWrpdQ/ssWZlbsebsXt4t06mUKm7hvs1rZGe0/04f92cJPtPot5gTUfCtHXnb2t59dib0ev7OFiFIRC9klcWW6LZrdObOYps23mvlXJhO6SsMtPfbWVXG8HSyV3v2+9C63Qp1eam7PwkzzVaxRSRVEdnCQw8j8F6TscZQkVA940dGZT/xZggFplJFzbGUMYLkjIXq08FqNDYCrSX+0poJoW0MKnKh8vs1TGOC2hj09Ghtg7XNrtHXGFcphf/7UKVsfCT5ti7LaVjCWG5caGr1x7g2BvE98B9lfeT9I2NtFZ+UddV+KPyvQ2Mz651I5ajVSIRQremLI7v8eD1qAYjX5gRg0wCaaoL1h2zv6Jt0JEuOXROltCUEVUs1oVSF8ysrg2m/q6QtVZrVWOPFGxn0tyz46kBkDpQTw1ZW/RHl9d6Hn+juP6K7g5PLh1EZr5006CasMmhx88b15IR6YW95bDd6H5OhnUITeuiPhkLw0Lo2IUdJtEPBoFRnBhTJZiFju0lb26f46Tkc9UHNaD9mIfS+4BuqcpOnmep46YCoPpKsXPd2bW6p6o/HkTg2uKOrQ+kTZPVxMXrxDOrq0deBmX326jHYEaYlsB6J9X3a2Z//mdn6/eMuOec0rYNYt5gLHeIKiCgQJ9JlGDF5eHuYBVMZpjy0F5u7Nz/0ULDtvEXa246o0dhsRAtZSGUtVqxLcKlY4tjMWvyFuVF9gg3qo6sN07m7ZDWn+KG76DTN3b3Fnkr6CFesmxMF3JVQfdKwN4UbQH0wLiv48qq+mPXOCyUAgPmRYjgIiKt/r81qiZaChnqnTODvl25lc7Tya7uC+dLPQ6PE+r59Z3h2E1h4h424mN7/7A19YR3U7A3b3RoqKjFEQge7vwFtIua8)[20], with both Options API and Composition API components.