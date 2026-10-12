---
title: 1.1. The Starting Point
---

No matter which language is used, programming begins with functional design in logic terms. Decomposition and abstraction are general principles of design. Functionality can be decomposed into logically independent parts, where each part can be used via an abstract interface. Decomposition not only reduces complexity of functionality, but also isolates it from external changes, making it reusable across different tasks. For reusability, functionality should consist of fixed mechanisms with variable policies. On the other hand, abstraction hides both implementation complexity and differences between specific implementations, making tasks portable across different implementations. In other words, abstraction makes functionality updatable with the same interface. Through decomposition and abstraction, tasks are designed as layered functionalities.

Languages provide various forms for functionality, resulting in different programming paradigms. Taking C as the starting baseline, the basic form is the function, which can be called via its interface and reused with different arguments. Note that in C, operations depend on memory layouts of data, and the language provides type semantics to determine data storage structures and valid operations. Functions are therefore reusable across different data instances of parameter types. Types with functions operating on them are actually the fundamental unit of design, which is provided as classes in C++, known as the object-oriented or class-based paradigm.

A GUI application can demonstrate design ideas discussed in this chapter. It is built on top of a simple graphics library — raylib.

```bash title="raylib"
# macOS
sudo port install raylib pkgconfig
# C
gcc `pkg-config --cflags raylib` -o build/demo src/*.c `pkg-config --libs raylib`
# C++
g++ -std=c++11 `pkg-config --cflags raylib` -o build/demo src/*.cpp `pkg-config --libs raylib`
```

:::note
Types with Functions in C

```c title="src/component.h"
#ifndef COMPONENT_H
#define COMPONENT_H

void *ComNew(char const *text, int fontSize, int x, int y, int width, int height);
void ComDelete(void *com);

void ComRender(void *com);

#endif // COMPONENT_H
```

```c title="src/component.c"
#include <raylib.h>
#include <stdlib.h>
#include <string.h>

#include "component.h"

#define PADDING 4

typedef struct {
  char *text;
  int fontSize;
  Rectangle rec;
} Component;

void *ComNew(char const *text, int fontSize, int x, int y, int width, int height) {
  Component *com = malloc(sizeof(Component));

  com->text = malloc(sizeof(char) * (strlen(text) + 1));
  strcpy(com->text, text);
  com->fontSize = fontSize;
  com->rec.x = x;
  com->rec.y = y;
  com->rec.width = width;
  com->rec.height = height;

  return com;
}

void ComDelete(void *com) {
  free(((Component *)com)->text);
  free(com);
}

void ComRender(void *com) {
  Component *com0 = com;
  DrawRectangleRec(com0->rec, GREEN);
  DrawText(com0->text, com0->rec.x + PADDING, com0->rec.y + PADDING, com0->fontSize, BLACK);
}
```

:::

:::note
Classes in C++

```cpp title="src/component.h"
#ifndef COMPONENT_H
#define COMPONENT_H

#include <raylib.h>

class Component {
public:
  char *text;
  int fontSize;
  Rectangle rec;

  Component(char const *text, int fontSize, int x, int y, int width, int height);
  ~Component();

  void Render();
};

#endif // COMPONENT_H
```

```cpp title="src/component.cpp"
#include <raylib.h>
#include <stdlib.h>
#include <string.h>

#include "component.h"

#define PADDING 4

Component::Component(char const *text, int fontSize, int x, int y, int width, int height) {
  this->text = (char *)malloc(sizeof(char) * (strlen(text) + 1));
  strcpy(this->text, text);
  this->fontSize = fontSize;
  this->rec.x = x;
  this->rec.y = y;
  this->rec.width = width;
  this->rec.height = height;
}

Component::~Component() { free(this->text); }

void Component::Render() {
  DrawRectangleRec(this->rec, GREEN);
  DrawText(this->text, this->rec.x + PADDING, this->rec.y + PADDING, this->fontSize, BLACK);
}
```

:::

```cpp title="src/main.cpp or src/main.c" showLineNumbers {12-19,25-32,36-40}
#include <raylib.h>

#include "component.h"

#define FONT_SIZE 20
#define WIDTH 200
#define HEIGHT 60
#define MARGIN 8

int main(void) {
  InitWindow(WIDTH + MARGIN * 2, HEIGHT * 2 + MARGIN * 3, "Components");
  // Reusable across different data of the type
  /* Class in C
  void *com1 = ComNew("Component 1", FONT_SIZE, MARGIN, MARGIN, WIDTH, HEIGHT);
  void *com2 = ComNew("Component 2", FONT_SIZE, MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT);
  */
  // C++
  Component com1("Component 1", FONT_SIZE, MARGIN, MARGIN, WIDTH, HEIGHT);
  Component com2("Component 2", FONT_SIZE, MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT);

  EnableEventWaiting();
  while (!WindowShouldClose()) {
    BeginDrawing();
    ClearBackground(WHITE);
    // Reusable across different data of the type
    /* Class in C
    ComRender(com1);
    ComRender(com2);
    */
    // C++
    com1.Render();
    com2.Render();
    EndDrawing();
  }

  // Reusable across different data of the type
  /* Class in C
  ComDelete(com1);
  ComDelete(com2);
  */
  CloseWindow();
  return 0;
}
```

Note that the "window" logic in the `main` function can also be decomposed into an independent part with abstract interfaces to hide the implementation.

```cpp title="src/window.h"
#ifndef WINDOW_H
#define WINDOW_H

class Component;

typedef struct _Coms {
  Component *com;
  _Coms *next;
} Coms;

class Window {
public:
  Coms *coms;

  Window(int width, int height, char const *title);
  ~Window();

  void AddCom(Component *com);
  void Run();
};

#endif // WINDOW_H
```

```cpp title="src/window.cpp"
#include <raylib.h>
#include <stdlib.h>

#include "component.h"
#include "window.h"

Window::Window(int width, int height, char const *title) {
  this->coms = NULL;
  InitWindow(width, height, title);
}

Window::~Window() {
  for (Coms *p = coms; p;) {
    Coms *p0 = p;
    p = p0->next;
    free(p0);
  }
  CloseWindow();
}

void Window::AddCom(Component *com) {
  Coms *p0 = (Coms *)malloc(sizeof(Coms));
  p0->com = com;
  p0->next = NULL;
  if (!coms) {
    coms = p0;
    return;
  }
  Coms *p = coms;
  for (; p->next; p = p->next)
    ;
  p->next = p0;
}

void Window::Run() {
  EnableEventWaiting();
  while (!WindowShouldClose()) {
    BeginDrawing();
    ClearBackground(WHITE);
    for (Coms *p = coms; p; p = p->next) {
      p->com->Render();
    }
    EndDrawing();
  }
}
```

```cpp title="src/component.h"
...
class Window;

class Component {
  ...
  Component(char const *text, int fontSize, int x, int y, int width, int height, Window *win);
  ...
};
...
```

```cpp title="src/component.cpp"
...
#include "window.h"
...
Component::Component(
    char const *text, int fontSize, int x, int y, int width, int height, Window *win) {
  ...
  win->AddCom(this);
}
...
```

```cpp title="src/main.cpp"
#include "component.h"
#include "window.h"

#define FONT_SIZE 20
...
int main(void) {
  Window win(WIDTH + MARGIN * 2, HEIGHT * 2 + MARGIN * 3, "Components");
  Component com1("Component 1", FONT_SIZE, MARGIN, MARGIN, WIDTH, HEIGHT, &win);
  Component com2("Component 2", FONT_SIZE, MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT, &win);

  win.Run();

  return 0;
}
```
