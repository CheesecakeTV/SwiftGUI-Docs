
# SwiftGUI
### ...Documentation

Find the main repository here: https://github.com/CheesecakeTV/SwiftGUI

# Structure of this documentation
1. Basic tutorials: Every user of SwiftGUI should have read all of them. They give a good overview over the most important features provided.
2. Element tutorials: Seperate tutorials for each element.
3. Advanced tutorials: Not necessarely harder, just more in-depth explanations for advanced users.
4. Application notes: "Practical tutorials". These provides tricks to improve your programs.
5. Examples: Small applications, sorted by their difficulty. Theory is good, but examples can clear up concepts better. Each file contains a list of concepts shown in the example.
6. References: Short explanations and links to applications developed using SwiftGUI. (Mostly mine ones at the moment...).

# Does your GUI look bad?
`import SwiftGUI as sg`

Call `sg.Themes.FourColors.Emerald()` before creating the layout.

This applies the `Emerald`-theme, my personal favorite.

See which themes are available by calling `sg.Examples.preview_all_themes()`.

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
And don't forget PySimpleGUIWeb and the other two packages.

Not to underestimate, its community is quite big nowadays, so it's easy to find helpful articles online.

I really do like PySimpleGUI, don't get me wrong.\
However, it seems like its creator develops the package, but doesn't apply it to actual applications.

Somehow, it misses prette obvious and "simple-to-implement" features.

I for my part used PySimpleGUI in actual applications for years, so I know what's important and what's not.

Let me use this part to rant a little about PySimpleGUI.

## Bugs in PySimpleGUI
The codebase is an utter mess, to say the least.
Its creator basically "brute-forced" all of the features, at least for updates prior to version 5.0.0.
"If it works, it works".\
One of the files has 16596 lines of code (not making this up).

Due to this short-minded thinking, PySimpleGUI has a ton of tiny bugs that can't be fixed easily.

Here are just a few I remember from the top of my hat:
- `sg.Input` throws an event when `Ctrl`, `Alt`, or `Shift` is pressed, even if the text doesn't change
- `sg.Table` sometimes throws multiple events instead of a single one
- `sg.Combo` doesn't throw an event when the text inside is changed, only when selecting an element
- Buttons for filebrowser, colorpicker, datepicker, etc. often return wrong/old/None values
- Holding down `Ctrl`, you can mess with the underline of `sg.Listbox` in strange ways (Never understood why there even is an underline by default...)

There are many more bugs I stumbled upon over time.
None of these are a big deal, only annoying.

SwiftGUI's elements have a clear, yet flexible structure.
Such a structure produces less bugs, simply because you need to write less code.
Also, bugs usually affect multiple elements, so they'll be more noticeable and don't slip through testing.

## Missing "obvious" features
What's even more strange, is the lack of obvious, basic features.

`Tkinter`, the package behind PySimpleGUI already offer so many methods to handle widgets/elements.
Adapting these features is almost as easy as simply calling the underlying method.

Example: You can't modify/append rows in `sg.Table`.
If you want to update **A SINGLE ROW**, you have to **REPLACE THE WHOLE TABLE**.\
Just why.\
Tkinter offers a method to overwrite row-contents, which makes it even more infuriating to me.

SwiftGUI offers many of those obvious methods, but also a lot of higher functionality, especially for `sg.Table`.
I "waste" my time so you don't have to.

You're welcome.

## Missing expandability
PySimpleGUI is a nightmare to expand.
Methods are too long and soooo much code is copied instead of being put in its own method.

You can't modify the behavior of elements without copying a good chunk of code (Something you shouldn't do.).

SwiftGUI is built to be easily expandable.
Using "combined elements", you can easily create your own elements.

SwiftGUI's class `BasePopup` lets you create your own, custom popups super conveniently.

## An event-loop isn't always a good thing
For smaller layouts, an event-loop is great.
Less functions and good readability.

For larger layouts, a single, big event-loop does the exact opposite.
Even the tiniest additional element makes the loop bigger and less tidy.

You should not need to "spend" a whole key for a button that does nothing but to clear an input-element.

In SwiftGUI, you can create sub-layouts that divide one big event-loop into multiple smaller ones.
You can also handle events/actions without going through the event-loop at all.

These features are one of the bigger strengths of SwiftGUI.
