### Related
- [[CMake]]
- [[Makefile]]
- [[RTTI]]
- [[Namespace]]

#### 1. The Problem: C++ is Static

Standard C++ is designed for raw performance. Once you compile your code, class names (like `MyClass`) are essentially erased and turned into memory addresses.

- **Standard C++:** "I call the function at memory address `0x1234`."
- **Qt Needs:** "I want to connect the 'Button Click' event to the function named 'onOpenFiles'."
    
Standard C++ **cannot** look up a function by its string name at runtime. It doesn't know what "onOpenFiles" is anymore.

#### 2. The Solution: The MOC (Code Generator)

Qt solves this by adding a step _before_ the actual C++ compiler runs.
1. **Scanning:** The MOC tool scans your header files.
2. **Detection:** It looks for the **`Q_OBJECT`** macro.
3. **Generation:** If it finds that macro, it generates a brand new C++ file (usually named `moc_filename.cpp`).    

**What is inside `moc_filename.cpp`?** This generated file contains a "lookup table" or metadata for your class:

- The string name of your class ("MainWindow").
- A list of all signals and slots as strings.
- Pointers to the actual functions.

Basically, it writes the C++ code you would have had to write manually to make reflection work.

---
### `main.cpp`

```cpp
#include <QApplication>
#include "ui/mainwindow.hpp"

int main(int argc, char *argv[]) {
	QApplication app(argc, argv);
	app.setApplicationName("SVG Editor");
	MainWindow window;
	window.show();
	return app.exec();
}
```

#### 1.  Standard C++ Entry Point

```cpp
int main(int argc, char *argv[]) {
```

- Like any C++ program, execution starts here. `argc` (argument count) and `argv` (argument vector) capture any command-line arguments passed when launching the app (e.g., `./SVGEditor --debug`).

#### 2. The Application Object 

```cpp
	QApplication app(argc, argv);
```

- This creates the core control object for your GUI. It initialises the screen, fonts, cursors, and handles OS-level integration.
- You **must** create one (and only one) `QApplication` object before you create any UI elements (like your `MainWindow`).

#### 3. Setting Metadata

```cpp
    app.setApplicationName("SVG Editor");
```

- This sets a global, readable name for your app.
- Qt uses this behind the scenes. For example, if you use `QSettings` to save user preferences, Qt will automatically create a folder or registry key named "SVG Editor" to store them. It also helps OS window managers identify your app.

#### 4. Creating and Showing the Window

```cpp
    MainWindow window;
    window.show();
```

- `MainWindow window;`: This instantiates your actual user interface on the stack. At this exact moment, the window exists in memory, but it is invisible.
- `window.show();`: By default, Qt widgets are created hidden (so you can build complex UIs without them flickering on the screen piece by piece). This line asks the OS to finally draw the window on your monitor.

### 5. The Event Loop 

```cpp
    return app.exec();
}
```

- `app.exec()` starts the **Qt Event Loop**.
- The program enters an infinite loop waiting for things to happen (mouse clicks, keyboard presses, timers, or network messages).
- When you click a button, the OS tells the Event Loop, the Event Loop finds the button, and the button emits its `clicked()` signal.
- The program stays trapped inside `app.exec()` for the entire life of your app.
- *When you close the last window (or call `QApplication::quit()`), the Event Loop breaks. `exec()` then returns an integer (usually `0` for success), which gets returned to the operating system, and the program terminates safely.

---
### SVG

#### 1. The Headers (The Metadata)

The first two lines tell the computer how to interpret the file:

- **`<?xml version="1.0" encoding="UTF-8"?>`**:

    - This is the **XML Declaration**. Since SVG is a dialect of XML, this header tells the browser/editor that the file follows XML version 1.0 rules and uses UTF-8 character encoding (which supports almost all global characters).
        
- **`<svg ... >`**:
    - **`xmlns="http://www.w3.org/2000/svg"`**: This is the **XML Namespace**. It tells the software that every tag inside (like `<rect>`) belongs to the official W3C SVG standard. Without this, a browser might just treat it as a generic text file.
    - **`width="800" height="600"`**: This defines the "physical" size of the drawing area on your screen in pixels.
    - **`viewBox="0 0 800 600"`**: This is the **coordinate system**. It says: "The top-left corner is (0,0) and the bottom-right is (800, 600)." This allows the image to scale perfectly; if you change the width/height, the `viewBox` ensures the internal shapes stay in the same relative positions.