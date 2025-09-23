
# SwiftGUI
### ...Documentation

Find the main repository here: https://github.com/CheesecakeTV/SwiftGUI

# Does your GUI look shitty?
`import SwiftGUI as sg`

Call `sg.Themes.FourColors.Emerald()` before creating the layout.

This applies the `Emerald`-theme.

See which themes are available by calling `sg.Examples.preview_all_themes()`.

# 28 different elements
(Version 0.8.0)

`import SwiftGUI as sg`

Call `sg.Examples.preview_all_elements()` for an overview.

# installation

Install using pip:
```bash
pip install SwiftGUI
```

Update to use the newest features and elements:
```bash
pip install SwiftGUI -U
```

# Documentations and tutorials are very important to me
If SwiftGUI should have any chance on competing with other gui-packages, it has to have a solid documentation.

My goal is to make developers use tools they don't know yet, so I better teach them properly.

# Why not just use PySimpleGUI?
I know how popular PySimpleGUI became over time, which it definetly deserved.
A lot of time and effort went into creating PySimpleGUI.

It has a ton of features and is straight-forward to use.
It is also quite compatible with older Python-versions.

Not to forget, its community is quite big nowadays, so it's easy to find helpful articles online.

Unfortunately, that's about all the nice things I can say about PySimpleGUI.

Let me use this part to rant a little about PySimpleGUI.

## Bugs in PySimpleGUI
The codebase is an utter mess, to say the least.
Its creator basically just "brute-forced" all of the features.
"If it works, it works".\
One of the files has 16596 lines of code (not making this up).

Due to this short-minded thinking, PySimpleGUI has a ton of tiny bugs that can't be fixed easily.

Here are just a few I remember from the top of my hat:
- `sg.Input` throws an event, when `Ctrl`, `Alt`, or `Shift` is pressed, even though the text didn't change.
- `sg.Table` sometimes throws multiple events for a single one
- `sg.Combo` doesn't throw an event when the text inside is changed, only when selecting an element
- Buttons for filebrowser, colorpicker, datepicker, etc. often return wrong values
- Holding down `Ctrl`, you can mess with the underline of `sg.Listbox` in strange ways (Never understood why there even is an underline by default...)

There are many more I stumbled upon over time.
None of these bugs is a big deal, they are just super annoying.

SwiftGUI's elements have a clear, yet flexible structure.
Such a structure produces less bugs, simply because you need to write less code.
Also, bugs usually affect multiple elements, so they'll be more noticeable and fixed sooner.

## Missing "obvious" features
What's even worse (in my opinion), is the lack of basic features.

`Tkinter`, the package behind PySimpleGUI already offer so many methods to handle widgets/elements.
Adapting these features is almost as easy as calling the underlying method.

Example: You can't modify/append rows in `sg.Table`.
If you want to update **A SINGLE ROW**, you have to **REPLACE THE WHOLE TABLE**.\
Just why.\
Tkinter offers a method to overwrite row-contents, which makes it even more infuriating to me.

As someone who has used PySimpleGUI for years, I know most of the things you regularely have to implement.
One of SwiftGUI's goals is to make regular tasks quick and simple.

## Missing expandability
PySimpleGUI is a nightmare to expand.
Methods are too long and soooo much code is copied instead of being put in its own method.

You can't modify the behavior of elements without copying a good chunk of code.

SwiftGUI is built to be easily expandable.
Using "combined elements", you can create your own elements very easy.

## An event-loop isn't always a good thing
For smaller layouts, an event-loop is great.
Less functions and good readability.

For larger layouts, a single, big event-loop does the exact opposite.
Even the tiniest additional element makes the loop bigger and less tidy.

You should not need to "spend" a whole key for a button that does nothing but to clear an input-element.

In SwiftGUI, you can create sub-layouts that divide one big event-loop into multiple smaller ones.
You can also handle events/actions without going through the event-loop at all.

These features are one of the bigger strengths of SwiftGUI.
