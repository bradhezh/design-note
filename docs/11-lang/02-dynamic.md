---
title: 1.2. Frameworks and Dynamic Types
---

In the previous section, a GUI application demonstrated the basic class-based paradigm. Now, consider adding user interaction to that demo.

```cpp title="src/window.cpp"
...
void Window::Run() {
  ...
  while (!WindowShouldClose()) {
    // Event
    if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
      Vector2 pos = GetMousePosition();
      for (Coms *p = coms; p; p = p->next) {
        if (CheckCollisionPointRec(pos, p->com->rec)) {
          TraceLog(LOG_INFO, "%s clicked", p->com->text);
        }
      }
    }
    // Render
    ...
  }
}
```

Furthermore, the interactive behaviour should be customised by the application rather than hardcoded in the `Window` class. When the behaviour (logic) of functionality can be customised, we refer to it as a framework in this book. Essentially, framework customisation requires functions to be reusable across different functions, not just across different data.

In programming, mechanisms are typically code and constants, which are fixed during design, whereas policies are traditionally data, which can vary at runtime. In C/C++, types act as mechanisms, which at compile time statically determine data storage structures and valid operations. At runtime, different instances with the same storage structure can act as dynamic policies. Also, function addresses are constants determined at compile time. For the same calling code to run different functions, those functions must be called through variables, known as callbacks. In C/C++, callbacks are function pointer variables, implemented via register jump instructions.

```cpp title="src/component.h"
...
class Component {
  ...
  void (*onClick)();
  ...
};
...
```

```cpp title="src/component.cpp"
...
Component::Component(
    char const *text, int fontSize, int x, int y, int width, int height, Window *win) {
  ...
  this->onClick = NULL;
}
...
```

```cpp title="src/window.cpp"
...
void Window::Run() {
  ...
  while (!WindowShouldClose()) {
    // Event
    if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
      Vector2 pos = GetMousePosition();
      for (Coms *p = coms; p; p = p->next) {
        if (CheckCollisionPointRec(pos, p->com->rec) && p->com->onClick) {
          p->com->onClick();
        }
      }
    }
    // Render
    ...
  }
}
```

```cpp title="src/main.cpp"
...
void Business1() { TraceLog(LOG_INFO, "Business 1"); }

void Business2() { TraceLog(LOG_INFO, "Business 2"); }

int main(void) {
  ...
  com1.onClick = Business1;
  com2.onClick = Business2;

  win.Run();
  ...
}
```
