> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [likegeeks.com](https://likegeeks.com/python-gui-examples-tkinter-tutorial/)

> Learn how to develop GUI applications using Python Tkinter package, In this tutorial, you'll learn ho......

In this tutorial, we will learn how to develop graphical user interfaces by writing some Python GUI examples using the Tkinter package.

Tkinter package is shipped with Python as a standard package, so we don’t need to install anything to use it.

Tkinter package is a very powerful package. If you already have installed Python, you may use IDLE which is the integrated IDE that is shipped with Python, this IDE is written using Tkinter. Sounds Cool!!

We will use Python 3.6, so if you are using Python 2.x, it’s strongly recommended to switch to Python 3.x unless you know the language changes so you can adjust the code to run without errors.

I assume that you have a little background about [Python basics](https://likegeeks.com/python-programming-basics/) to help you understand what we are doing.

We will start by creating a window then we will learn how to add widgets such as buttons, combo boxes, etc, then we will play with their properties, so let’s get started.

Create your first GUI application
---------------------------------

First, we will import Tkinter package and create a window and set its title:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.mainloop()

```

The result will be like this:

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAANYAAADvBAMAAAB8uDd4AAAAKlBMVEXw8PAAeNff398Yg9j////Hx8jk5OTV1dXQ0NDa2tpbsOkHJDyp6veJjNgfpNqNAAACVUlEQVR42u3dMWvbQABAYUGN6Hook6JJAYNLh9BQx22WBkSGQIYECh0duxxkzmBrzBCsdnSgRqNLEkq34G72UujmehBd+2N6ukSNXRMwde9C23dYktHy+KSTRp3jOK6UMjQ6pKw7TinKU2GYGh6hiuWt5oaFUdUttyksjCfSXutZuZ631v611uY7qVrSSqvW1a0zS63XkdO4bQ0NtuLZ1nBsrvU0Lk+3Vr9uWWuNPv9eK8rH/OlHant/23qjW8dFa6Dvl78vJqKq/gTZYq1tta3Mnz7vC393qhWq1tFxcQkHV/nRO/Qeig/Lt1RI5X62Ot0p1+p447rV9A+UbOmWOP+4K35pFa7ReEu3xCT4nk280t6yLf9FX9zhGg1upsZF8PzbYZDV5lor+STYXrwV9e90DW4e5WrVf3uyHu0YuIaF69P11FCtC+9xtp6ZmBuFa1i8oYI98SULTlRron5/as7PzsOi5T/wLoVX2lm0tdizPOMSVybf87MuG617cQmLLoEL1//iciyMwmWvhQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cOHChQsXLly4cP2NriPuF677delvVjfO7LTK9lqxvVZNt6Sdlv7+vLtmY82FTf1dfbfRa5+evjQ7kjjULVlJW4nR2Ksk7pbzliNlmPZa7U6StBMTu06nFaehXksij1UqvbSXprGJnQql3VBKV6/94Uqzq39085RTL0U/AJ7H6WeUwmQhAAAAAElFTkSuQmCC)

Awesome!! Our application just works.

The last line which calls the mainloop function, this function calls the endless loop of the window, so the window will wait for any user interaction till we close it.

If you forget to call the mainloop function, nothing will appear to the user.

Create a label widget
---------------------

To add a label to our previous example, we will create a label using the label class like this:

```
lbl = Label(window, text="Hello")

```

Then we will set its position on the form using the grid function and give it the location like this:

```
lbl.grid(column=0, row=0)

```

So the complete code will be like this:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

lbl = Label(window, text="Hello")

lbl.grid(column=0, row=0)

window.mainloop()

```

And this is the result:

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAKIAAABFCAMAAAD+bc0TAAAAQlBMVEUAeNfw8PD////a2toYg9fS0tLExMTLy8vh4eLc2MTp6emNzfFYoeNfADHw8KsKAwrMyLURKVLwy4QIXaWr8PCrYADn1o73AAABuklEQVRo3u3a0Y6CMBCFYcrQZSzVUnDf/1W3s0PANSjiJnCI/TVS774MFk2kMM5xirBids6kylQiVpCJcSA6rgrAKkrGTPw3UYxKZMIkWiE6dCLDE+kZsY+x2LNlYozN/sRazrQST6d7Ymwavz+xPUkzRBX27xLLseKF2OvR8xyxfUaMPr5LnBYv5Es/HuaJPEeUIQYfii2IilPhiin2IQmruAVRjSJcRdQhxk2IakzCtUQZYli9A94nmmdEniXqEBFOdPtoimXwSYi8XWIMoU9HjIvOgx0dohwxLt18T9RCgdAwxcMS+wKhv0TMn7T6WQQntoeYohLJYhK/rCXmTMzE3bsjGsAyMRNROiTxcm7GV13IekWZeDRiF0K8ClHXwZuNWiaG31RWCVGe3nRbD3N5ip1AeyV28WrM97ZjXCYqy0ATL2d5O51oYW7e8nbREz2sG7Omz/p2qTPxM4gH+CwegUgjkTCJ9aGIjEtkdthEgife/GXuuILMEk9EIrK2Tn2BlCjW0g3RsSJTNULWClCFSjRiZJIsWYgopUIzEFMYt97Z9NB4EA5Eo0icOwRZcm4ilvBlYiai9APwHUsQriZQkQAAAABJRU5ErkJggg==)

Without calling the grid function for the label, it won’t show up.

### Set label font size

You can set the label font so you can make it bigger and maybe bold. You can also change the font style.

To do so, you can pass the font parameter like this:

```
lbl = Label(window, text="Hello", font=("Arial Bold", 50))

```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALsAAAB/CAMAAACAN0P6AAAAb1BMVEXw8PAAeNcAAAD///8Yg9fd3d3IyMjW1tbR0NHm4s3Z1L/f28fh5eXw8KtgAABfrO6q8fEBYKsAAGALHDqrYADw78+Vy/GANgU/itPz6tkIOHA2BADnq2eUZkXwzofMhzY5XnMAb8eMntUAmu+Nn6uJmsxuAAAD0klEQVR42u3agXqaMBQFYAQR0XTIUiimOLX1/Z9xAdRre+yXUKOQ9p5vWzeWJf8ul4BswbpJkRapL1mn6zZRpO1n9nL86aDFuujsZ/XCh3T+Y92LdOpbNL6re5pOJ35l2lTeV/tC44vWvvTSnhYe21NP7Q3e157R9rbuywXac6km4422L76y76Us5WS8aezLL+yyLCs/7HEyBfquknvXK0bn2IwWWfc1E1fss3kSC3HFvpelqhTVPZvpH1Zvt9vpJxbJooy+XLPHYO/KHl6W/X2tvwfzB9v3Wg10k30vNf2y3d9FNcnK+MH21g10g101ZaeWaftFf1vd346VD4huZaeyXzT8qsoMl58DOxY++oZdKfp10/C65TMfekZGYVd2sgtd+fdH279zrcpdGOYfx64ivUMOskfu++2RuQzV53mCanz3JrTr7MPJyAN2is/2MT/+gt27zx5stwnb2T542P7woD3wK2y3CtvZPnjYbhW2s33wsN0qbGf74Plp9qc/4TF5AHk+/d7fVWAVmDS7cojtbGc729nOdrazfRT2ot68hjpSHZLb7fVGdrPtDuLe9sUmvIjMbrIvXkKY7W724iinqLKvHeUU9XY3e0GrQbF627fhteTiPvanho6prO14BjFK3MOOVUeT2U4LGPAu7OblZGltx4bB5HZ2c8he08Hd7OOllve1/3v9NFm6uThSubYXL1DmGo6Y7Nh9qsQLQJau7AjFM5/3s9dEX13ro9yx/RnOKNWPRtnY8RTiYUd2Go3z1sSytVO3Ux3wuCM7zUtbGLiqPvYtTQ67Gy3ixI6tjadZCXt78QJz4TXl0I7tjg1vtn85Boc6uzcR0nR7Mtup/ai6eLVWbDfYyWVrr+FSxWVzd3Y92F+7z3U3233o9w6AGa8d9nfIePd3uK9CnNxXaVWX9hoeNSAPfJ4x283Pkc9S7naHwzwJhnmONNvxROMUTp7faVFHdupFKMo27P/8Tn8OPzfRCk7tNJymrokgTHY8hJ9XaU1ndnwtod4EvZm0h2awocB7Ahrnym66tSpha6f5McRxbKdtgIJjzHZzIXLh3o543CzMdsQj3b0d8Ug32wm/NbyYdWyn65OyK12+fy8D93ZcEF/229upFDCZyX5bCvgnoj5BPk32O/7fki9hu1XYzvbBw3arsJ3tg4ftVmH7T7ILX+1JfLTH3tnnJ7vw0i5Odt/Slj1o7CKOk2Q+n3mSua76yd7gtV7ztX/cfwPt0/KWfraf9No/6rTEuKt60NpPes2Pk3EnPsq1XZzsnV7EHkS0cp3GHvkbn+3/AQKW0gcOaWRsAAAAAElFTkSuQmCC)

Note that the font parameter can be passed to any widget to change its font not labels only.

Great, but the window is so small, we can even see the title, what about setting the window size?

### Setting window size

We can set the default window size using geometry function like this:

```
window.geometry('350x200')

```

The above line sets the window width to 350 pixels and the height to 200 pixels.

Let’s try adding more GUI widgets like buttons and see how to handle the button click event.

Adding a button widget
----------------------

Let’s start by adding the button to the window, the button is created and added to the window the same as the label:

```
btn = Button(window, text="Click Me")

btn.grid(column=1, row=0)

```

So our window will be like this:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

lbl = Label(window, text="Hello")

lbl.grid(column=0, row=0)

btn = Button(window, text="Click Me")

btn.grid(column=1, row=0)

window.mainloop()

```

The result looks like this:

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAWsAAADyCAMAAAC8hhR1AAAAWlBMVEXw8PDc3NzFxcTPz8/g4N/k5OTr6+sAeNfX19cYg9f///4AhOne2saYlti1/vmH0vRpaGifoaBfADHw8KtYresMBQza/fnsyOEaKl9dfNjwyYGXSQ0FXabA3PM4gr3oAAAGB0lEQVR42u2ci3aaQBRFIeEhUJzEAILR///N3nnxUBONBZem+8SFCDK2m50zkKw2yIM88EmChMwcgWqT+0dgKcdJTGaMBmqAD7wNabMzIjNHE7ewbYNo0FEU6qzIrNFMo3gwO0gysmic2uJ1EmcpWTBZ2JsN68VZ+xpJkgjWy7Jeha5FkjiE9eKsjdgJrBdn/epKRCoE1kuzXoXfsN63LYxmZR1r1nG0OmHdtqqG0ZyspUR0XZ+ybpXq8HpG1q8ruVk3U+Mxa0G9h3Xe55p3Hyr7XB3Osw6/YF2rthv1tSrTtFjrxdZuMC9+FHtE0ZTTQ4smz4Ps9I2Pwfpk5btUeTV6OmL9YkrETI1HrEXruhvVtUZcNLKofsD66D3+1WSzsJcz2aXn3vhsrC3ls6hHrE+8rgX1uEIKuYevDqEV/DbWxTnWhSovnaInYq05n0c9sI6PWRutx5d8mokq112x7sx3fWeAqDwvzctSA2ryssrzra2FTjPTr+zuoUPWpXls3TBNN1TJMLTer9xxyoyih++fHpa1wD6P+huvW6315IpPlcUhk4X5+0ql6KXaOoSmhIOs0DTsaSj6XXb3EWtV+mH094tAHQY1S1X2E4M/RoZv/NPdprsFWMfnWE/vZATBVsq6sj7mGoh10ixV2XPV23Ojt+Pc+eIZWDdOfhmm6dLhGD9042cH88m5OxfuU1JVPnOHnHqd11OtpbC1a6Fy9NJvWXfDJDewLo5Zd37jdKzU7td/8NJtU0/C+pq58cTruq3r/ZS1MqWsO3XtnR11SDdmvR5dq/RuTztk64aRP1k3HNMPrew+88qVx9Z+yjYdTuVzXvMds97X9fF9TKUvgi3dxnaImfxGc+OIre0QmdYmc6PvHn2aNEE7keb9fLodhlaBNF/e2ZMcHOSYg/2Ugxvt6e5lvvRazL765vM+f+t1+WCXgz+9R//6+jp9sB87Fb+G9Rmv9w/JOv2VXpP7eU3wGq8JXuM1XsMDr3+j18HF5DfmLSBTr69gfdu/YniH9Yj1tV7HN30MrG/yGtZ4jdewxmu8/s+8/tiofmlX+vUR6+izrttG9uxGb/Qre/P8WStYf+/1Vax3dSWLcrxndERrR4D1HF5Hn9WU7/SINxkn2bUbWF/t9U5qYu1Yy3pd9ax37Xp8ImIplMqs783G981atr37IWD9tde1iUWcWZwfmyqwzexYNyPW8ac7R9Z2u7Jr/VnKYH3Ra61yvbfEjMgG5Rmv7Ss5R5nfKOfis+qHgPXFvvYIz7Ae9fXoje3eb5QCsRcp7pzA+oLXH5usx2k6xIA7vg6ZdEgW9FWfDUPA+vJ1yG7oELOuhmu+RL826uqduuIrA133uJkt39bDELBe+L4x4b6Rn4fw8xC8hjVe4zWs8fp5vX6/LbD+uddvtwbUP/aa3M9rgtd4TfAar/Ga4DVeE7zGa7wmeI3XBK/xGtZ4jdd4TfAarwle4zVeE7zGa4LXeI3XBK/xmuA1XuM1rPEarwle4zXBa7zGa4LXeE3wGq/xmuA1XhO8xmu8hgde4zXBa7wmeI3XeE3wGq8JXuM1XhO8xmuC13iN1wSv8ZrgNV7DGq/xGq8JXuM1wWu8xmuC13hN8Bqv8ZrgNV4TvMZrWOM1XuM1wWu8JniN13hN8BqvCV7jNV4TvMZrgtePyTqC9Z1Yx38ksF6Y9YumHESwvhvrPzGs78Baw6ZD7ul1BOt7scZr+pq+JvQ1fU3oa/qavib0NX1N6Gv6+n9jHfA7sDuw5ne7d2eN13iN1wSv8ZrgNV7jNcFrvCZ4jdd4TfAarwle4zVewxqv8ZrgNV4TvMZrvCZ4jdcEr/Earwle4zXBa7zGa4LXeE3wGq9hjdd4jdcEr/Ga4DWsfzvrV2EdG9YRrBdnLVrHgflP9GG9MGtdIUlgSgTWi7OOHesoI4vGaC2s7ey4etV5IbNHsK5GrMXsUHA73nzN9mUipA1qzVqLHTvaq1cLnMwTjTTUqD1rY7ahHVrgq5DHPz88xzCypBM3UVrYcRRZ3mS2RJp0nNi2DnIP2+M2CXn88yOyoHur/wKY3vkwg11HxAAAAABJRU5ErkJggg==)

Note that we place the button on the second column of the window which is 1. If you forget and place the button on the same column which is 0, it will show the button only, since the button will be on the top of the label.

### Change button foreground and background colors

You can change the foreground for a button or any other widget using **fg** property.

Also, you can change the background color for any widget using **bg** property.

```
btn = Button(window, text="Click Me", bg="orange", fg="red")

```

![](https://likegeeks.com/wp-content/uploads/2018/01/03-Python-GUI-examples-button-colors.png)

Now, if you tried to click on the button, nothing happens because the click event of the button isn’t written yet.

### Handle button click event

First, we will write the function that we need to execute when the button is clicked:

```
def clicked():

    lbl.configure(text="Button was clicked !!")

```

Then we will wire it with the button by specifying the function like this:

btn = Button(window, text=**“Click Me”**, command=clicked)

**Note that**, we typed clicked only not clicked() with parentheses.

Now the full code will be like this:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

lbl = Label(window, text="Hello")

lbl.grid(column=0, row=0)

def clicked():

    lbl.configure(text="Button was clicked !!")

btn = Button(window, text="Click Me", command=clicked)

btn.grid(column=1, row=0)

window.mainloop()

```

And when we click the button, the result as expected:

![](https://likegeeks.com/wp-content/uploads/2018/01/04-Python-GUI-examples-button-click-event.png)

Cool!!

Get input using Entry class (Tkinter textbox)
---------------------------------------------

In the previous Python GUI examples, we saw how to add simple widgets, now let’s try getting the user input using the Tkinter Entry class (Tkinter textbox).

You can create a textbox using Tkinter Entry class like this:

```
txt = Entry(window,width=10)

```

Then you can add it to the window using grid function as usual

So our window will be like this:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

lbl = Label(window, text="Hello")

lbl.grid(column=0, row=0)

txt = Entry(window,width=10)

txt.grid(column=1, row=0)

def clicked():

    lbl.configure(text="Button was clicked !!")

btn = Button(window, text="Click Me", command=clicked)

btn.grid(column=2, row=0)

window.mainloop()

```

And the result will be like this:

![](https://likegeeks.com/wp-content/uploads/2018/01/04-Python-GUI-examples-add-entry-widget.png)

Now, if you click the button, it will show the same old message, what about showing the entered text on the Entry widget?

First, you can get entry text using get function. So we can write this code to our clicked function like this:

```
def clicked():

    res = "Welcome to " + txt.get()

    lbl.configure(text= res)

```

If you click the button and there is a text on the entry widget, it will show “Welcome to” concatenated with the entered text.

And this is the complete code:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

lbl = Label(window, text="Hello")

lbl.grid(column=0, row=0)

txt = Entry(window,width=10)

txt.grid(column=1, row=0)

def clicked():

    res = "Welcome to " + txt.get()

    lbl.configure(text= res)

btn = Button(window, text="Click Me", command=clicked)

btn.grid(column=2, row=0)

window.mainloop()

```

Run the above code and check the result:

![](https://likegeeks.com/wp-content/uploads/2018/01/05-Python-GUI-examples-entry-widget-event.png)

Awesome!!

Every time we run the code, we need to click on the entry widget to set focus to write the text, what about setting the focus automatically?

### Set focus to the entry widget

That’s super easy, all we need to do is to call focus function like this:

```
txt.focus()

```

And when you run your code, you will notice that the entry widget has the focus so you can write your text right away.

### Disable entry widget

To disable the entry widget, you can set the state property to disabled:

```
txt = Entry(window,width=10, state='disabled')

```

![](https://likegeeks.com/wp-content/uploads/2018/01/05-Python-GUI-examples-disable-entry-widget.png)

Now, you won’t be able to enter any text.

Add a combobox widget
---------------------

To add a combobox widget, you can use the Combobox class from ttk library like this:

```
from tkinter.ttk import *

combo = Combobox(window)

```

Then you can add your values to the combobox.

```
from tkinter import *

from tkinter.ttk import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

combo = Combobox(window)

combo['values']= (1, 2, 3, 4, 5, "Text")

combo.current(1) #set the selected item

combo.grid(column=0, row=0)

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/05-Python-GUI-examples-combobox.png)

As you can see, we add the combobox items using the [tuple](https://likegeeks.com/lists-vs-tuples/).

To set the selected item, you can pass the index of the desired item to the current function.

To get the select item, you can use the get function like this:

```
combo.get()

```

Add a Checkbutton widget (Tkinter checkbox)
-------------------------------------------

To create a checkbutton widget, you can use the Checkbutton class like this:

```
chk = Checkbutton(window, text='Choose')

```

Also, you can set the checked state by passing the check value to the Checkbutton like this:

```
from tkinter import *

from tkinter.ttk import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

chk_state = BooleanVar()

chk_state.set(True) #set check state

chk = Checkbutton(window, text='Choose', var=chk_state)

chk.grid(column=0, row=0)

window.mainloop()

```

Check the result:

![](https://likegeeks.com/wp-content/uploads/2018/01/05-Python-GUI-examples-add-checkbutton.png)

### Set check state of a Checkbutton

Here we create a variable of type BooleanVar which is not a standard Python variable, it’s a Tkinter variable, and then we pass it to the Checkbutton class to set the check state as the highlighted line in the above example.

You can set the Boolean value to false to make it unchecked.

Also, you can use IntVar instead of BooleanVar and set the value to 0 or 1.

```
chk_state = IntVar()

chk_state.set(0) #uncheck

chk_state.set(1) #check

```

These examples give the same result as BooleanVar.

Add radio buttons widgets
-------------------------

To add radio buttons, simply you can use RadioButton class like this:

```
rad1 = Radiobutton(window,text='First', value=1)

```

Note that you should set the value for every radio button with a different value, otherwise, they won’t work.

```
from tkinter import *

from tkinter.ttk import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

rad1 = Radiobutton(window,text='First', value=1)

rad2 = Radiobutton(window,text='Second', value=2)

rad3 = Radiobutton(window,text='Third', value=3)

rad1.grid(column=0, row=0)

rad2.grid(column=1, row=0)

rad3.grid(column=2, row=0)

window.mainloop()

```

The result of the above code looks like this:

![](https://likegeeks.com/wp-content/uploads/2018/01/06-Python-GUI-examples-add-radio-button.png)

Also, you can set the command of any of these radio buttons to a specific function, so if the user clicks on any one of them, it runs the function code.

This is an example:

```
rad1 = Radiobutton(window,text='First', value=1, command=clicked)

def clicked():

# Do what you need

```

Pretty simple!!

### Get radio button value (selected radio button)

To get the currently selected radio button or the radio button value, you can pass the variable parameter to the radio buttons, and later you can get its value.

```
from tkinter import *

from tkinter.ttk import *

window = Tk()

window.title("Welcome to LikeGeeks app")

selected = IntVar()

rad1 = Radiobutton(window,text='First', value=1, variable=selected)

rad2 = Radiobutton(window,text='Second', value=2, variable=selected)

rad3 = Radiobutton(window,text='Third', value=3, variable=selected)

def clicked():

   print(selected.get())

btn = Button(window, text="Click Me", command=clicked)

rad1.grid(column=0, row=0)

rad2.grid(column=1, row=0)

rad3.grid(column=2, row=0)

btn.grid(column=3, row=0)

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/06-Python-GUI-examples-get-radio-button-value.png)

Every time you select a radio button, the value of the variable will be changed to the value of the selected radio button.

Add a ScrolledText widget (Tkinter textarea)
--------------------------------------------

To add a ScrolledText widget, you can use the ScrolledText class like this:

```
from tkinter import scrolledtext

txt = scrolledtext.ScrolledText(window,width=40,height=10)

```

Here we specify the width and the height of the ScrolledText widget, otherwise, it will fill the entire window.

```
from tkinter import *

from tkinter import scrolledtext

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

txt = scrolledtext.ScrolledText(window,width=40,height=10)

txt.grid(column=0,row=0)

window.mainloop()

```

The result as you can see:

![](https://likegeeks.com/wp-content/uploads/2018/01/07-Python-GUI-examples-add-scrolledtext.png)

### Set scrolledtext content

To set scrolledtext content, you can use the insert method like this:

```
txt.insert(INSERT,'You text goes here')

```

### Delete/Clear scrolledtext content

To clear the contents of a scrolledtext widget, you can use delete method like this:

```
txt.delete(1.0,END)

```

Great!!

Create a MessageBox
-------------------

To show a message box using Tkinter, you can use the messagebox library like this:

```
from tkinter import messagebox

messagebox.showinfo('Message title','Message content')

```

Pretty easy!!

Let’s show a message box when the user clicks a button.

```
from tkinter import *

from tkinter import messagebox

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

def clicked():

    messagebox.showinfo('Message title', 'Message content')

btn = Button(window,text='Click here', command=clicked)

btn.grid(column=0,row=0)

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/08-Python-GUI-examples-show-messagebox.png)

When you click the button, an info messagebox will appear.

### Show warning and error messages

You can show a warning message or error message in the same way. The only thing that needs to be changed is the message function

```
messagebox.showwarning('Message title', 'Message content')  #shows warning message

messagebox.showerror('Message title', 'Message content')    #shows error message

```

### Show askquestion dialogs

To show a yes no message box to the user, you can use one of the following messagebox functions:

```
from tkinter import messagebox

res = messagebox.askquestion('Message title','Message content')

res = messagebox.askyesno('Message title','Message content')

res = messagebox.askyesnocancel('Message title','Message content')

res = messagebox.askokcancel('Message title','Message content')

res = messagebox.askretrycancel('Message title','Message content')

```

You can choose the appropriate message style according to your needs. Just replace the showinfo function line from the previous line and run it.

Also, you can check what button was clicked using the result variable

If you click **OK** or **yes** or **retry**, it will return **True** value, but if you choose **no** or **cancel**, it will return **False**.

The only function that returns one of three values is **askyesnocancel** function, it returns **True or False or None**.

Add a SpinBox (numbers widget)
------------------------------

To create a Spinbox widget, you can use Spinbox class like this:

```
spin = Spinbox(window, from_=0, to=100)

```

Here we create a Spinbox widget and we pass the from_ and to parameters to specify the numbers range for the Spinbox.

Also, you can specify the width of the widget using the width parameter:

```
spin = Spinbox(window, from_=0, to=100, width=5)

```

Check the complete example:

```
from tkinter import *

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

spin = Spinbox(window, from_=0, to=100, width=5)

spin.grid(column=0,row=0)

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/08-Python-GUI-examples-spinbox.png)

You can specify the numbers for the Spinbox instead of using the whole range like this:

```
spin = Spinbox(window, values=(3, 8, 11), width=5)

```

Here the Spinbox widget only shows these 3 numbers only 3, 8, and 11.

### Set default value for Spinbox

To set the Spinbox default value, you can pass the value to the textvariable parameter like this:

```
var =IntVar()

var.set(36)

spin = Spinbox(window, from_=0, to=100, width=5, textvariable=var)

```

Now, if you run the program, it will show 36 as a default value for the Spinbox.

Add a Progressbar widget
------------------------

To create a progress bar, you can use the progressbar class like this:

```
from tkinter.ttk import Progressbar

bar = Progressbar(window, length=200)

```

You can set the progress bar value like this:

```
bar['value'] = 70

```

You can set this value based on any process you want like [downloading a file](https://likegeeks.com/downloading-files-using-python/) or completing a task.

### Change Progressbar color

Changing Progressbar color is a bit tricky, but super easy.

First, we will create a style and set the background color and finally set the created style to the Progressbar.

Check the following example:

```
from tkinter import *

from tkinter.ttk import Progressbar

from tkinter import ttk

window = Tk()

window.title("Welcome to LikeGeeks app")

window.geometry('350x200')

style = ttk.Style()

style.theme_use('default')

style.configure("black.Horizontal.TProgressbar", background='black')

bar = Progressbar(window, length=200, style='black.Horizontal.TProgressbar')

bar['value'] = 70

bar.grid(column=0, row=0)

window.mainloop()

```

And the result will be like this:

![](https://likegeeks.com/wp-content/uploads/2018/01/12-Python-GUI-examples-change-progressbar-color.png)

Add a filedialog (file & directory chooser)
-------------------------------------------

To create a file dialog (file chooser), you can use the filedialog class like this:

```
from tkinter import filedialog

file = filedialog.askopenfilename()

```

After you choose a file and click open, the file variable will hold that file path.

Also, you can ask for multiple files like this:

```
files = filedialog.askopenfilenames()

```

### Specify file types (filter file extensions)

You can specify the file types for a file dialog using the filetypes parameter, just specify the extensions in tuples.

```
file = filedialog.askopenfilename(filetypes = (("Text files","*.txt"),("all files","*.*")))

```

You can ask for a directory using the askdirectory method:

```
dir = filedialog.askdirectory()

```

You can specify the initial directory for the file dialog by specifying the initialdir like this:

```
from os import path

file = filedialog.askopenfilename(initialdir= path.dirname(__file__))

```

Easy!!

Add a Menu bar
--------------

To add a menu bar, you can use menu class like this:

```
from tkinter import Menu

menu = Menu(window)

menu.add_command(label='File')

window.config(menu=menu)

```

First, we create a menu, then we add our first label, and finally, we assign the menu to our window.

You can add menu items under any menu by using add_cascade() function like this:

```
menu.add_cascade(label='File', menu=new_item)

```

So our code will be like this:

```
from tkinter import *

from tkinter import Menu

window = Tk()

window.title("Welcome to LikeGeeks app")

menu = Menu(window)

new_item = Menu(menu)

new_item.add_command(label='New')

menu.add_cascade(label='File', menu=new_item)

window.config(menu=menu)

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/09-Python-GUI-examples-add-menu-bar.png)

Using this way, you can add many menu items as you want.

```
from tkinter import *

from tkinter import Menu

window = Tk()

window.title("Welcome to LikeGeeks app")

menu = Menu(window)

new_item = Menu(menu)

new_item.add_command(label='New')

new_item.add_separator()

new_item.add_command(label='Edit')

menu.add_cascade(label='File', menu=new_item)

window.config(menu=menu)

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/10-Python-GUI-examples-menu-add-cascade.png)

Here we add another menu item called Edit with a menu separator.

You may notice a dashed line at the beginning, well, if you click that line, it will show the menu items in a small separate window.

You can disable this feature by disabling the tearoff feature like this:

```
new_item = Menu(menu, tearoff=0)

```

Just replace the new_item in the above example with this one and it won’t show the dashed line anymore.

I don’t need to remind you that you can type any code that works when the user clicks on any menu item by specifying the command property.

```
new_item.add_command(label='New', command=clicked)

```

Add a Notebook widget (tab control)
-----------------------------------

To create a tab control, there are 3 steps to do so.

*   First, we create a tab control using Notebook class
*   Create a tab using Frame class.
*   Add that tab to the tab control.
*   Pack the tab control so it becomes visible in the window.

```
from tkinter import *

from tkinter import ttk

window = Tk()

window.title("Welcome to LikeGeeks app")

tab_control = ttk.Notebook(window)

tab1 = ttk.Frame(tab_control)

tab_control.add(tab1, text='First')

tab_control.pack(expand=1, fill='both')

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/11-Python-GUI-examples-notebook-widget.png)

You can add many tabs as you want the same way.

### Add widgets to Notebooks

After creating tabs, you can put widgets inside these tabs by assigning the parent property to the desired tab.

```
from tkinter import *

from tkinter import ttk

window = Tk()

window.title("Welcome to LikeGeeks app")

tab_control = ttk.Notebook(window)

tab1 = ttk.Frame(tab_control)

tab2 = ttk.Frame(tab_control)

tab_control.add(tab1, text='First')

tab_control.add(tab2, text='Second')

lbl1 = Label(tab1, text= 'label1')

lbl1.grid(column=0, row=0)

lbl2 = Label(tab2, text= 'label2')

lbl2.grid(column=0, row=0)

tab_control.pack(expand=1, fill='both')

window.mainloop()

```

![](https://likegeeks.com/wp-content/uploads/2018/01/12-Python-GUI-examples-add-widgets-to-notebooks.png)

Add spacing for widgets (padding)
---------------------------------

You can add padding for your controls to make it looks well organized using padx and pady properties.

Just pass padx and pady to any widget and give them a value.

```
lbl1 = Label(tab1, text= 'label1', padx=5, pady=5)

```

Just that simple!!

In this tutorial, we saw many Python GUI examples using Tkinter library and we saw how easy it’s to develop graphical interfaces using it.

This tutorial covers the main aspects of Python GUI development not all of them. There is no tutorial or a book can cover everything.

I hope you find these examples useful. Keep coming back.

Thank you.

![](https://secure.gravatar.com/avatar/34ca0a894f5a054e8d2bda344dd48c49?s=100&d=mm&r=g)

Mokhtar is the founder of LikeGeeks.com. He works as a Linux system administrator since 2010. He is responsible for maintaining, securing, and troubleshooting Linux servers for multiple clients around the world. He loves writing shell and Python scripts to automate his work.