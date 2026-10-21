---
title: 1.1. The Starting Point
---

No matter which language is used, programming begins with functional design in logic terms. Decomposition and abstraction are general principles of design. Functionality can be decomposed into logically independent parts, where each part can be used via an abstract interface. Decomposition not only reduces the complexity of functionality, but also isolates it from external changes, making it reusable across different tasks. For reusability, functionality should consist of fixed mechanisms with variable policies. On the other hand, abstraction hides both implementation complexity and the differences between specific implementations, making tasks portable across different implementations. In other words, abstraction makes functionality updatable with the same interface. Through decomposition and abstraction, tasks are designed as layered functionalities.

Languages provide various forms for functionality, resulting in different programming paradigms. Taking C as the starting baseline, the basic form is the function, which can be called via its interface and reused with different arguments. Note that in C, operations depend on data memory layouts, and the language provides the type semantics to determine data memory layouts and valid operations. Functions therefore operate on their parameter types and then are reusable across different data instances of those types. Types paired with the functions operating on them actually serve as the fundamental unit of design, provided as classes in C++, forming the object-oriented or class-based paradigm.

A GUI application can demonstrate general design ideas that this chapter tries to explain. It is built on top of a simple graphics library — raylib.

### raylib

```bash title="raylib"
# macOS
sudo port install raylib pkgconfig
# C
gcc `pkg-config --cflags raylib` -o build/demo src/*.c `pkg-config --libs raylib`
# C++
g++ -std=c++11 `pkg-config --cflags raylib` -o build/demo src/*.cpp `pkg-config --libs raylib`
```

### Component in C

```c title="src/component.h"
#ifndef COMPONENT_H
#define COMPONENT_H

#include <raylib.h>

typedef struct _Com Com;

Com *com_new(char const *text, int font_size, Rectangle const *rec);
void com_del(Com *self);
void com_render(Com const *self);

#endif // COMPONENT_H
```

```c title="src/component.c"
#include <raylib.h>
#include <stdlib.h>
#include <string.h>

#include "component.h"

#define PADDING 4

struct _Com {
  char *text;
  int font_size;
  Rectangle rec;
};

Com *com_new(char const *text, int font_size, Rectangle const *rec) {
  Com *com = malloc(sizeof(Com));
  com->text = malloc(sizeof(char) * (strlen(text) + 1));
  strcpy(com->text, text);
  com->font_size = font_size;
  com->rec = *rec;
  return com;
}

void com_del(Com *self) {
  free(self->text);
  free(self);
}

void com_render(Com const *self) {
  DrawRectangleRec(self->rec, GREEN);
  DrawText(self->text, self->rec.x + PADDING, self->rec.y + PADDING, self->font_size, BLACK);
}
```

### Component in C++

```cpp title="src/component.h"
#ifndef COMPONENT_H
#define COMPONENT_H

#include <raylib.h>

class Com {
public:
  char *text;
  int font_size;
  Rectangle rec;

  Com(char const *text, int font_size, int x, int y, int width, int height);
  ~Com();
  void Render() const;
};

#endif // COMPONENT_H
```

```cpp title="src/component.cpp"
#include <raylib.h>
#include <stdlib.h>
#include <string.h>

#include "component.h"

#define PADDING 4

Com::Com(char const *text, int font_size, int x, int y, int width, int height) {
  this->text = (char *)malloc(sizeof(char) * (strlen(text) + 1));
  strcpy(this->text, text);
  this->font_size = font_size;
  this->rec.x = x;
  this->rec.y = y;
  this->rec.width = width;
  this->rec.height = height;
}

Com::~Com() { free(this->text); }

void Com::Render() const {
  DrawRectangleRec(this->rec, GREEN);
  DrawText(this->text, this->rec.x + PADDING, this->rec.y + PADDING, this->font_size, BLACK);
}
```

### Use of Component

```cpp title="src/main.cpp or src/main.c" showLineNumbers {11-18,25-32,37-41}
#include <raylib.h>

#include "component.h"

#define FONT_SIZE 20
#define WIDTH 200
#define HEIGHT 60
#define MARGIN 8

int main(int argc, char *argv[]) {
  /* Class in C
  void *com1 = com_new("Component 1", FONT_SIZE, &(Rectangle){MARGIN, MARGIN, WIDTH, HEIGHT});
  void *com2 =
      com_new("Component 2", FONT_SIZE, &(Rectangle){MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT});
  */
  // C++
  Com com1("Component 1", FONT_SIZE, MARGIN, MARGIN, WIDTH, HEIGHT);
  Com com2("Component 2", FONT_SIZE, MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT);

  InitWindow(WIDTH + MARGIN * 2, HEIGHT * 2 + MARGIN * 3, "Components");
  EnableEventWaiting();
  while (!WindowShouldClose()) {
    BeginDrawing();
    ClearBackground(WHITE);
    // Reusable across different instances
    /* Class in C
    com_render(com1);
    com_render(com2);
    */
    // C++
    com1.Render();
    com2.Render();
    EndDrawing();
  }
  CloseWindow();

  // Reusable across different instances
  /* Class in C
  com_del(com1);
  com_del(com2);
  */
  return 0;
}
```

Note that the "window" logic in the `main` function can also be decomposed into an independent part with abstract interfaces to hide the implementation.

### Window

```cpp title="src/window.h"
#ifndef WINDOW_H
#define WINDOW_H

class Com;
typedef struct _ComNode ComNode;

class Win {
public:
  ComNode *nodes;

  Win(char const *title, int width, int height);
  ~Win();
  void AddCom(Com *com);
  void Run() const;
};

#endif // WINDOW_H
```

```cpp title="src/window.cpp"
#include <raylib.h>
#include <stdlib.h>

#include "component.h"
#include "window.h"

struct _ComNode {
  Com *com;
  struct _ComNode *next;
};

Win::Win(char const *title, int width, int height) {
  this->nodes = NULL;
  InitWindow(width, height, title);
}

Win::~Win() {
  CloseWindow();
  for (ComNode *p = this->nodes; p;) {
    ComNode *node = p;
    p = p->next;
    free(node);
  }
}

void Win::AddCom(Com *com) {
  ComNode *node = (ComNode *)malloc(sizeof(ComNode));
  node->com = com;
  node->next = NULL;
  if (!this->nodes) {
    this->nodes = node;
    return;
  }
  ComNode *p = this->nodes;
  for (; p->next; p = p->next)
    ;
  p->next = node;
}

void Win::Run() const {
  EnableEventWaiting();
  while (!WindowShouldClose()) {
    BeginDrawing();
    ClearBackground(WHITE);
    for (ComNode *p = this->nodes; p; p = p->next) {
      p->com->Render();
    }
    EndDrawing();
  }
}
```

### Use of Window

```cpp title="src/component.h"
...
class Win;

class Com {
  ...
  Com(char const *text, int font_size, int x, int y, int width, int height, Win *win);
  ...
};
...
```

```cpp title="src/component.cpp"
...
#include "window.h"
...
Com::Com(char const *text, int font_size, int x, int y, int width, int height, Win *win) {
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
int main(int argc, char *argv[]) {
  Win win("Components", WIDTH + MARGIN * 2, HEIGHT * 2 + MARGIN * 3);
  Com com1("Component 1", FONT_SIZE, MARGIN, MARGIN, WIDTH, HEIGHT, &win);
  Com com2("Component 2", FONT_SIZE, MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT, &win);

  win.Run();

  return 0;
}
```
