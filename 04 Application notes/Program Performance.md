
# Runtime performance
This application note is about reducing the runtime-cost to increase the code performance.

This is only relevant for "bigger" scripts, like when using a lot of elements.

But there are some good practices to reduce the runtime-cost with low effort.
So why not.

# Use sg.Notebook
Many elements at once cause the layout to be laggy, especially when moving around the window.

`sg.Notebook` is a double-edged sword.
It hides some of the elements, making the window less laggy.
However, it takes some time when changing tabs, since the hidden elements have to be reloaded.

So only use a notebook when you have a lot of elements and don't need to access all of them at once.

# .update
Don't call `.update` multiple times if you can call it once instead.

Don't:
```py
my_elem.update(background_color= "red")
my_elem.update(text_color= "green")
my_elem.update(fontsize= 14)
my_elem.update(font_bold= True)
```

Do:
```py
my_elem.update(
    background_color= "red",
    text_color= "green",
    fontsize= 14,
    font_bold= True,
)
```

# Background color propagation
One of SwiftGUI's most costly features is background-color-propagation.

You might recall that elements can't have a truely transparent background.
It looks transparent, because the background-color of "transparent" elements is kept the same as its container's background-color.

When the background-color of the container changes, all of those elements inside also change background-color.

As you can imagine, that costs.

So, don't often change the background-color of frames with a lot of elements.

# Applying global options
Since SwiftGUI version 0.10.7, global options got a lot more efficient.

It's still more efficient to make less "single" calls and instead use `apply`.

Don't:
```py
go_class.single("background_color", ...)
go_class.single("text_color", ...)
go_class.single("fonttype", ...)
go_class.single("fontsize", ...)
``` 

Do:
```py
go_class.apply(
    {
        "background_color": ...,
        "text_color": ...,
        "fonttype": ...,
        "fontsize": ...
    }
)
```


