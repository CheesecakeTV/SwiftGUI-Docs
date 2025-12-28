
# Userfriendliness
I know, SwiftGUI's GUIs can look pretty outdated.\
But for me personally, that's not an issue.

As long as it doesn't look terrible, my sole focus is on the userfriendliness.

Userfriendliness transforms a good program into a great one.
On the other side, bad userfriendliness makes even the best programs feel very bad.

A good example for this is LTSpice, a circuit simulator.
The simulations are amazing, but I hate to use it.
It has very bad user-friendliness.

**Keep in mind that this note only contains my personal experience and opinions.**
I have no formal education on any subjects presented here.

I also have strong opinions on a lot of "modern" gui-styles.
Many modern user-interfaces focus solely on looks and make using it unnecessarely complicated on the way.

# The most important rule
**Use your program and be petty about it.**

Your program shouldn't annoy the user, ideally at all.

Sure, going through 2 popups to reach your desired functionality might make for a cleaner code, but the user only cares about frontend (the interface).

Use your own program a lot.\
Improve everything that doesn't feel natural to you.\
Improve everything that is slowing you down unnecessarely.

Yes, something very small is only a tiny bit annoying, but trust me, the user is gonna notice.

Remove every tiny annoyance you find.

# Working with the keyboard
A general rule:\
Moving the hand from the mouse to the keyboard and vice-versa is annoying.

So try to not mix too many "keyboard-actions" with "mouse-actions".

## Don't overuse the keyboard
The most obvious way to make the user work quicker is by adding a ton of hotkeys and making the mouse obsolete.

While that is definetly true, efficiency isn't the same as user-friendliness.\
There is a reason the text-editor vim isn't as popular as it deserves to be.

Also, keep in mind that most pc-users probably can't type as fast as you programmer.

They like to use the mouse more, especially when thinking about what to do next.

So, I like to add "quick-action"-buttons to input-elements.\
Examples:
- A button to clear the input
- A button to copy the input-text to clipboard (Same as `ctrl+c`)
- A button to paste the text from clipboard (Same as `ctrl+v`)
- For numeric inputs, buttons to add/substract 1, multiply/divide by 10, multiply by -1, etc.

This can easily be done for multiple elements using teplate functions, as described in the advanced tutorial on custom combined elements.

## Replacing button-presses
A lot of times, an input-element only serves a very specific purpose.

So, try to "predict" what the user wants to do next and automate it.

Examples:
- (Very common) An entry of some kind. Instead of clicking on "submit", the user could simply press "enter", like most of us are used to.
- An input that acts as a searchbar. When the user inputs something, the  What do you think the user wants to do if his search only yields a single result? So open that result for him.
- An input that adds texts to a list. What do you think is the first thing the user does after adding a text? He clears the input. So clear it for him.

## Focus
The focus is one of the most important parts of any GUI, yet noone seems to know it even needs special considerations.

First of all, what is the focus?\
When an element "has the focus", it can be modified using the keyboard.

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
Enter your information as quickly as possible.

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
I know it looks dificult, but all the addition does is to jump the focus to the next input-element when the user presses enter.\
Keep that in mind and try to enter your information again.
Much smoother.

You have two main ways to control the focus:
1. Activate/deactivate the `takefocus`-option for certain elements.
If `takefocus = False`, that element will be skipped when the user presses `tab`.
2. Set the focus manually like I did in the example.

Don't underestimate the blessing of a good focus-handling.

# Layout considerations
## Efficient mouse-movements
Keeping mouse-movements to a minimum is the "mouse-equivalent" of proper focus handling.

If specific buttons are often clicked in sequence, move these buttons closer together.

Example: Shutting down your windows PC.\
First, you'll press the windows-icon, then the power-actions and finally "shutdown".
In Windows 10, all of these buttons are very close together, so the user can quickly press them one after another.\
In Windows 11 (by default), the windows-icon is a little to the left of the start-menu, while the power options are on the right.
Just why.

### Protecting "dangerous" actions
Buttons that do something hard/impossible to undo should have an extra layer of protection to them.
Especially when they are placed next to regularely used buttons.

Otherwise, the user needs to move the mouse slower/careful to not accidently cause such actions.

Example: I think we all clicked on "reboot" instead of "shutdown" too many times.

The straight-forward way to prevent this is a popup asking if the action should really be executed.
In SwiftGUI, this is quite easy to integrate into the event-loop:
```py
for e, v in w:
    if e == "Prank_Parents" and sg.Popups.yes_no("Do you really want to make your parents angry?"):
        prank_parents()
```

Another thing I like to do is to make the button require a double-click to cause an event:
```py
layout = [
    [
        sg.Button(
            "Prank parents",
            key= "Prank_Parents",
            default_event= False,   # Disable single-click-event
        ).bind_event(   # Enable double-click-event
            sg.Event.ClickDoubleLeft
        )
    ]
]
```
Alternative implementation without setting a key for the button:
```py
layout = [
    [
        sg.Button(
            "Prank parents",
        ).bind_event(
            sg.Event.ClickDoubleLeft,
            key = "Prank_Parents",
        )
    ]
]
```

## Visual separation of functionalities
Believe it or not, but separators, notebooks and label-frames are your friend.

If certain elements have little/nothing to do with other elements, separate them from each other.

Label-frames are especially useful.
Try to get used to them.

# Beginner-friendliness
Keep in mind that nobody knows your own program better than you.\
That's not always a good thing.

It makes it hard to empathize with new users.

In Germany, we have a very fitting word for this: "Betriebsblindheit".\
It means something like "blinded by working on it".

This is the true danger to your program.

Users want to use your program, not learn it.
Make it as obvious as possible.

## Descriptive texts
There is an annoying trend where text-buttons are replaced by image-buttons in modern layouts.

Even though you yourself know what an image is supposed to represent, not everyone will.

Take a look at PyCharm's sidebar:\
![](../assets/images/2025-12-28-02-42-50.png)

Could you tell what these buttons do without knowing beforehand?

Long story short, only use pure image-buttons if the image is actually obvious.

A good solution is to combine the image with text, as described in the basic tutorial about images:
![](../assets/images/2025-09-21-18-25-52.png)

By the way, this doesn't just apply to buttons.\
E.g. input-elements don't have any description/image on their own.
Make sure the user knows what each field represents.

## External feedback
To counter the effect of "Betriebsblindheit", make sure to get feedback from actual new users.\
(Do you know the German word, or did you skip that part, proving my point?)

The reason I'm even including this is that it can be hard to take their criticism seriously.
The feedback will feel petty and stupid, but that's their honest impression.

If it bothers them, it will bother others too.

My advice: 
- Ask multiple people
- Don't tell them anything on how to use the program, let them figure it out completely on their own
- If they use the program longer, ask for feedback after 1-2 hours again. New programs are never instantly understood

Little annecdote: My father tested one of my programs.
He complained about a button that said "Apply", because he didn't know the English word "apply".
The word is pretty common in my opinion, so I would have never guessed this could be an issue.
I changed the text to "Apply/Use".

## Users are stupid
It's sad, but it's true.

Treat your users like toddlers:
- Foolproof everything
  - Especially make sure the program doesn't crash on wrong inputs
- Don't expect them to read descriptive texts
- Don't expect them to have intuition
- They'll randomly click on things they don't know/understand
- They won't remember most hotkeys

Force them to user your software correctly.
But don't make it too hard, or they'll stop using the software completely.


