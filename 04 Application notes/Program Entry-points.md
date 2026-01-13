
# Program entry
Except for some very small applications, most Python-programs consist of multiple Python-files.

If so, you should implement a proper program-entry.

The entry-point is the part of code you need to start manually to open the program.

This application note shows a couple of useful things you can to do using entry-points.

# Introduction
Probably the best known program-entry looks like this:

`main.py`
```py
# Imports go here
import SwiftGUI as sg

def main():
    # Functional code starts here
    ...

if __name__ == "__main__":
    main()
```
Running `main.py` directly executes `main()`.\
However, `import main` does not, even though it runs through the script.

Note that the file does not need to be named `main.py` to work.

## Why use a main-function
Couldn't you just write the code like this?
```py
# Imports go here
import SwiftGUI as sg

if __name__ == "__main__":
    # Functional code starts here
    ...
```
You can do whatever you want.\
Using a main function has two major advantages though.

1. You can run it from different files:
`other_file.py`
```py
import main

if __name__ == "__main__":
    main.main()
```
This is great if your program contains multiple modules (parts).
Every module could have their own entry to only open that program-part.

2. You need to keep definition-order in mind.

This works:
```py
def main():
    fct()

def fct():
    ...

if __name__ == "__main__":
    main()
```
This does not (`fct` isn't defined yet):
```py
fct()

def fct():
    ...
```

A minor advantage is that you can declare non-global variables in functions.
All variables outside of functions are global.

# Multiple program-entries
`if __name__ == "__main__"` basically tests if you started that script directly or through a different source.

So you can use it to create multiple entry-points for different purposes:

`main.py`
```py
import SwiftGUI as sg

def main():
    # Functional code starts here
    ...

if __name__ == "__main__":  # Normal entry
    sg.Files.set_root("SwiftGUI Applicationnote")
    main()
```

`debug.py`
```py
import SwiftGUI as sg
import main

if __name__ == "__main__":
    sg.Files.set_root("SwiftGUI Applicationnote", ignore_parent=True)
    main.main()
```

Starting `main.py` defines the program-directory inside the home-folder.
Starting `debug.py` on the other hand puts it inside the script-folder.
That's useful for debugging.

# Entry-point with error-log
I often convert my program from Python-files to exe using auto-py-to-exe, so users with no Python-knowledge can use them too.

For these users, I usually create another entry-point with a crude error-log:\
`user_entry.py`
```py
import SwiftGUI as sg
import main

if __name__ == "__main__":
    try:
        main.main()
    except Exception as ex:
        with open("Errorlog.txt", "w") as f:
            f.write(f"{ex.__class__.__name__} \n\n{ex}")
```
There are probably many better ways to report errors, but it's enough for me.





