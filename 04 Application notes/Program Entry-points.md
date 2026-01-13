
# Program entry
Except for some very small applications, most Python-programs consist of multiple Python-files.

If so, you should implement a proper program-entry.

The entry-point is the part of code you need to start manually to open the program.

This application note shows a couple of useful things you can to do using entry-points.

# Most basic program-entry
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
However, `import main` does not.

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



