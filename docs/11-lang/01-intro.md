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

void *ComNew(char const *text);
void ComDelete(void *com);

void ComRender(void *com, int x, int y);

#endif // COMPONENT_H
```

```c title="src/component.c"
#include <raylib.h>
#include <stdlib.h>
#include <string.h>

#include "component.h"

#define FONT_SIZE 20

typedef struct {
  char *text;
} Component;

void *ComNew(char const *text) {
  Component *com = malloc(sizeof(Component));

  com->text = malloc(sizeof(char) * (strlen(text) + 1));
  strcpy(com->text, text);

  return com;
}

void ComDelete(void *com) {
  free(((Component *)com)->text);
  free(com);
}

void ComRender(void *com, int x, int y) {
  DrawText(((Component *)com)->text, x, y, FONT_SIZE, BLACK);
}
```

:::

:::note
Classes in C++

```cpp title="src/component.h"
#ifndef COMPONENT_H
#define COMPONENT_H

class Component {
public:
  char *text;

  Component(char const *text);
  ~Component();

  void Render(int x, int y);
};

#endif // COMPONENT_H
```

```cpp title="src/component.cpp"
#include <raylib.h>
#include <stdlib.h>
#include <string.h>

#include "component.h"

#define FONT_SIZE 20

Component::Component(char const *text) {
  this->text = (char *)malloc(sizeof(char) * (strlen(text) + 1));
  strcpy(this->text, text);
}

Component::~Component() { free(this->text); }

void Component::Render(int x, int y) {
  DrawText(this->text, x, y, FONT_SIZE, BLACK);
}
```

:::

```cpp title="src/main.cpp or src/main.c" showLineNumbers {10-14,20-24,28-30}
#include <raylib.h>

#include "component.h"

#define WIDTH 200
#define HEIGHT 60

int main(void) {
  InitWindow(WIDTH, HEIGHT, "Component");
  /* Class in C
  void *com = ComNew("Component");
  */
  // C++
  Component com("Component");

  EnableEventWaiting();
  while (!WindowShouldClose()) {
    BeginDrawing();
    ClearBackground(WHITE);
    /* Class in C
    ComRender(com, 0, 0);
    */
    // C++
    com.Render(0, 0);
    EndDrawing();
  }

  /* Class in C
  ComDelete(com);
  */
  CloseWindow();
  return 0;
}
```
