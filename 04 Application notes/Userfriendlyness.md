
# Userfriendlyness
I know, SwiftGUI's GUIs can look pretty outdated.
But for me personally, that's not an issue.

As long as it doesn't look terrible, my sole focus is on the userfriendlyness.

Userfriendlyness transforms a good program into a great one.
On the other side, bad userfriendlyness makes even the best programs feel very bad.

A good example for this is LTSpice, a circuit simulator.
The simulations are amazing, but I hate to use it.
It has very bad user-friendlyness.

# Use your program and be petty about it
Your program shouldn't annoy the user, ideally at all.

Sure, going through 2 popups to reach your desired functionality might make for a cleaner code, but the user doesn't see the code.

Use your own program a lot.\
Improve everything that doesn't feel natural to you.\
Improve everything that is slowing you down unnecessarely.

Yes, something very small is only a tiny bit annoying, but trust me, the user is gonna notice.

Remove every tiny annoyance you find.

# Working with the keyboard
A general rule of thumb:\
Moving the hand from the mouse to the keyboard and vice-versa is annoying.

So try to not mix too many "keyboard-actions" with "mouse-actions".

## Replacing button-presses

## Focus
The focus is one of the most important parts of any GUI, yet noone seems to know it even needs special considerations.

First of all, what is the focus?\
When an element "has the focus", it can be modified through the keyboard.

E.g.: Clicking on an input-element gives that element the focus, so that typing will add text to the input.
Or when a button has the focus (visible as a dotted line around it), pressing enter presses the button.

Your task is to move the focus where it needs to be as naturally as possible.

Run this example:
```py
import SwiftGUI as sg

sg.Themes.FourColors.Emerald()

texts = [
    sg.T(text, width= 10, tk_kwargs= {"takefocus":True}) for text in ["Name", "Age", "Birthday", "Town"]
]
num_rows = len(texts)

inputs = [
    sg.In(key= n) for n in range(num_rows)
]

clear_buttons = [
    sg.Button("x", width= 2) for _ in inputs
]

layout = zip(texts, inputs, clear_buttons)

w = sg.Window(layout, title= "Enter your information please")

for e,v in w:
    ...
```
Enter your information.

Does this feel natural/convenient?\
In my opinion, definetly not.

Now, try this variant:
```py
import SwiftGUI as sg

sg.Themes.FourColors.Emerald()

texts = [
    sg.T(text, width= 10, tk_kwargs= {"takefocus":True}) for text in ["Name", "Age", "Birthday", "Town"]
]
num_rows = len(texts)

inputs = [
    sg.In(
        key= n
    ).bind_event(
        sg.Event.KeyEnter, key_function= lambda w, e: w[min(num_rows - 1, e + 1)].set_focus()
    ) for n in range(num_rows)
]

clear_buttons = [
    sg.Button("x", width= 2) for _ in inputs
]

layout = zip(texts, inputs, clear_buttons)

w = sg.Window(layout, title= "Enter your information please")

for e,v in w:
    ...
```
I know it looks dificult, but all the addition does is to jump the focus to the next input-element when the user presses enter.

You have two main ways to control the focus:
1. Activate/deactivate the `takefocus`-option for certain elements.
If `takefocus = False`, that element will be skipped when the user presses `tab`.
2. Set the focus manually like I did in the example.

Don't underestimate the blessing a good focus-routine is.
It makes or breaks a lot of your user-friendlyness.

# Layout considerations
## Mouse-movement-distances and hotkeys

## Accessability vs tidyness

## Visual separation of functionalities 

## Descriptive texts

# Getting feedback from other people
