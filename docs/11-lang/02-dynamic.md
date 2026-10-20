---
title: 1.2. Frameworks and Dynamic Types
---

In the previous section, a GUI application demonstrated the basic class-based paradigm. Now, consider adding user interaction to that demo.

### User Interaction

```cpp title="src/window.cpp"
...
void Win::Run() const {
  ...
  while (!WindowShouldClose()) {
    // Event
    if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
      Vector2 pos = GetMousePosition();
      for (ComNode *p = this->nodes; p; p = p->next) {
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

In programming, mechanisms are typically code and constants, which are fixed during design, whereas policies are traditionally variables, which can vary at runtime. In C/C++, types act as mechanisms, which at compile time statically determine data memory layouts and valid operations. At runtime, different instances with the same memory layout can serve as dynamic policies. Also, function addresses are constants determined at compile time. For the same calling code to run different functions, those functions must be called through variables, known as callbacks. In C/C++, callbacks are function pointer variables, implemented via register jump instructions.

### Callbacks

```cpp title="src/component.h"
...
class Com {
  ...
  void (*onClicked)();
  ...
};
...
```

```cpp title="src/component.cpp"
...
Com::Com(char const *text, int font_size, int x, int y, int width, int height, Win *win) {
  ...
  this->onClicked = NULL;
}
...
```

```cpp title="src/window.cpp"
...
void Win::Run() const {
  ...
  while (!WindowShouldClose()) {
    // Event
    if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
      Vector2 pos = GetMousePosition();
      for (ComNode *p = this->nodes; p; p = p->next) {
        if (CheckCollisionPointRec(pos, p->com->rec) && p->com->onClicked) {
          p->com->onClicked();
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
void onCom1Clicked() { TraceLog(LOG_INFO, "Business triggered by com1 clicked"); }

void onCom2Clicked() { TraceLog(LOG_INFO, "Business triggered by com2 clicked"); }

int main(int argc, char *argv[]) {
  ...
  com1.onClicked = onCom1Clicked;
  com2.onClicked = onCom2Clicked;

  win.Run();
  ...
}
```

In essence, callbacks transform target addresses for calling from constants determined at compile time into variables flexible at runtime, making the same calling code reusable across different callees. The former can be referred to as static binding, while the latter can be referred to as dynamic binding. Generally, transformation from constants or code into variables can convert fixed mechanisms into flexible policies to improve reusability.

Dynamic binding can be pushed further. Note that the offset of that callback itself within that object is still a constant, and if it is transformed into a variable, binding can be even more flexible. This actually means that the callback would be dynamically allocated on the heap at runtime as its address is not determined at compile time. Callbacks allocated on the heap are commonly referred to as signals, and callees for binding are known as slots. Signals can have more than just one function pointer to support more for binding. For example, one signal can be bound to multiple slots, and calling (signal emission) can be asynchronous and even across threads or processes. With signals and slots, framework customisation becomes more of wiring-up than binding.

Note that the callback is defined in that class and each instance has its own callee. Similarly, signal names and signatures are still specified at the class level and shared across all instances, while each instance has its own callback instances (signal connection table). Actually, in GObject and Qt, signals specifically refer to their names and signatures belonging to the class (type). As a result, types no longer only exist at compile time, since their signal names and signatures are also kept at runtime as metadata. Metadata enables types to serve as dynamic policies, allowing frameworks to be reusable across different types.

Dynamic types can be extended even further. In Objective-C, metadata includes all method names and addresses, and binding is based on messages (method names) by default. On the other hand, dynamic binding implies runtime cost, including additional memory for callbacks, signals, connections, and metadata, as well as time overhead for looking up signal names or messages.

In contrast, C++ prioritises runtime performance above all else, providing virtual functions for dynamic binding. All callbacks for a class are aggregated into a virtual function table, shared across all instances, eliminating per-instance connections. Calling callbacks uses their indexes in that table, which are determined at compile time, eliminating lookups. In fact, the entire table is determined at compile time, and customisation relies on inheritance. Derived classes have the same indexes in their own tables as their base classes for those inherited callbacks, only with the callee addresses replaced with their own overridden ones. The same virtual function calling therefore can be bound to different methods across derived classes based on the target instance.

Virtual functions offer near-optimal performance, where metadata and wiring-up are all determined at compile time, with only one table for each class and one pointer to that table for each instance as runtime overhead. However, relying on inheritance introduces rigid type contracts and compromises ergonomics. Moreover, inheritance forces classes to expose their memory layouts to ensure that base class operations work across derived classes, resulting in implementation leaking into interfaces, which compromises updatability. Even if abstract interfaces remain unchanged, modifying base class memory layouts still breaks ABI compatibility and requires all derived downstream code to be recompiled.

GObject supports both signals and virtual functions for distinct use cases. Virtual functions and inheritance are still useful for the is-a relationship. For example, a widget is an object, and a button is a widget. They share common interfaces and type contracts are determined during design. Signals, by contrast, have no knowledge of any customisers at all. For updatability, GObject supports private members hidden within implementation.

### Signals and Virtual Functions

```bash title="GObject with raylib"
sudo port install gtk4 +quartz
gcc `pkg-config --cflags gobject-2.0 raylib` -o build/demo src/*.c `pkg-config --libs gobject-2.0 raylib`
```

#### Component

```c title="src/component.h"
#ifndef COMPONENT_H
#define COMPONENT_H

#include <glib-object.h>
#include <raylib.h>

/* Properties, only necessary for derivable classes
#define P_TEXT "text"
#define P_FONT_SIZE "font_size"
#define P_REC "rec"
// Only for init
#define P_WIN "win"
*/

// Signals
#define ON_CLICKED "onClicked"

typedef struct _Win Win;

// The 4th param is usually a bare name for parent objects.
// G_DECLARE_FINAL_TYPE(MyClass, my_class, MY_CLASS, OBJ, Parent):
// typedef struct _MyClass MyClass; // Class
// This class metadata is mainly a virtual function table.
// typedef struct { ParentClass parent_class; } MyClassClass;
// GType my_class_get_type(); // Get the class ID (also register the class for the 1st call)
// MY_CLASS_OBJ(obj) // Castor for instances
// MY_CLASS_IS_OBJ(obj) // Checker for instances
// G_DECLARE_DERIVABLE_TYPE:
// Differences:
// Definition instead of declaration for layout compatibility across derived classes. Real data
// members should therefore be defined in implementations.
// typedef struct _MyClass { Parent parent_instance; } MyClass;
// Declaration instead of definition since derivable classes typically have virtual functions to
// define in metadata
// typedef struct _MyClassClass MyClassClass;
// Additions: For virtual functions
// MY_CLASS_OBJ_CLASS(class) // Castor for metadata
// MY_CLASS_IS_OBJ_CLASS(class) // Checker for metadata
// MY_CLASS_OBJ_GET_CLASS(obj) // Get metadata from instances
// GInitiallyUnowned instead of GObject for floating objects. All objects derived from GObject are
// managed via reference counting, created via g_object_new with 1 initial ref, while floating
// objects must sink before their ref counts work. Floating implies no owners, but the initial ref
// count is still 1. Therefore g_object_ref increasing refs can't be used for floating objects,
// while g_object_ref_sink not increasing floating refs but eliminating floating marks has to be
// used instead to assume their ownership. Note that g_object_ref_sink is the same as g_object_ref
// for normal objects without floating marks. Typically, UI components are initially floating to be
// managed by their containers. GObject also supports weak references, which are basically pointers
// without g_object_ref to assume target ownership, but will automatically clear the pointer to NULL
// when the target is finalised.
G_DECLARE_FINAL_TYPE(Com, com, COM, FLOATING_OBJ, GInitiallyUnowned)
#define COM_TYPE (com_get_type())

Com *com_new(char const *text, int font_size, Rectangle const *rec, Win *win);
Rectangle const *com_rec(Com const *self);
void com_render(Com const *self);
void com_clicked(Com *self);

#endif // COMPONENT_H
```

```c title="src/component.c"
#include <glib-object.h>
#include <raylib.h>

#include "component.h"
#include "window.h"

#define PADDING 4

// Class
struct _Com {
  GObject parent_instance;

  /* For derivable classes
  struct _ComPrivate {
  */
  char *text;
  int font_size;
  Rectangle rec;
  Win *win;
};

// G_DEFINE_TYPE(MyClass, my_class, PARENT_TYPE):
// static gpointer my_class_parent_class = NULL; // Parent metadata
// GType my_class_get_type() {
//  // Register the class for the 1st call, and return its ID
//  // Registration process:
//  // Allocate the metadata with its parent_class cloned from parent metadata, register it for the
//  // class into the system, including necessary wiring-up like my_class_init, also wire up
//  // my_class_parent_class
//  // Call my_class_class_init with the metadata
// }
// G_DEFINE_TYPE_WITH_PRIVATE also implements private members defined by `struct _MyClassPrivate`
// and provides my_class_get_instance_private to access them.
G_DEFINE_TYPE(Com, com, G_TYPE_INITIALLY_UNOWNED)

static void com_finalize(GObject *obj) {
  Com *self = COM_FLOATING_OBJ(obj);
  g_free(self->text);
  if (self->win) {
    g_object_remove_weak_pointer(G_OBJECT(self->win), (gpointer *)&self->win);
  }
  G_OBJECT_CLASS(com_parent_class)->finalize(obj);
}

/* Properties, only necessary for derivable classes
// Note that all GObject objects are created via g_object_new, which calls my_class_init in a
// universal way, and properties can be used to init objects with custom args, because g_object_new
// can set properties right after my_class_init. This is the only way to init parent_instance of
// objects with args, but only necessary for derivable classes, because my_class_new with those args
// described in the signature should typically be provided, which is enough for finals.
enum { PROP_0, TEXT, FONT_SIZE, REC, WIN, PROP_N };
enum { FONT_SIZE_MIN = 8, FONT_SIZE_MAX = 64, FONT_SIZE_DEF = 12 };
static GParamSpec *props[PROP_N] = {NULL};
static void com_set_property(GObject *obj, guint id, GValue const *value, GParamSpec *spec) {
  Com *self = COM_FLOATING_OBJ(obj);
  gpointer p;
  switch (id) {
  case TEXT:
    g_free(self->text);
    self->text = g_value_dup_string(value);
    g_return_if_fail(self->text);
    break;
  case FONT_SIZE:
    self->font_size = g_value_get_int(value);
    break;
  case REC:
    p = g_value_get_pointer(value);
    g_return_if_fail(p);
    self->rec = *(Rectangle *)p;
    break;
  case WIN:
    p = g_value_get_object(value);
    g_return_if_fail(p);
    self->win = (Win *)p;
    g_object_add_weak_pointer(G_OBJECT(self->win), (gpointer *)&self->win);
    win_add_com(self->win, self);
    break;
  default:
    G_OBJECT_WARN_INVALID_PROPERTY_ID(obj, id, spec);
    break;
  }
}
*/

enum { CLICKED, SIGNAL_N };
static guint signals[SIGNAL_N] = {0};

static void com_class_init(ComClass *klass) {
  GObjectClass *gobj_class = G_OBJECT_CLASS(klass);
  gobj_class->finalize = com_finalize;

  /* Properties, only necessary for derivable classes
  gobj_class->set_property = com_set_property;
  props[TEXT] = g_param_spec_string(P_TEXT, NULL, NULL, "", G_PARAM_CONSTRUCT | G_PARAM_WRITABLE);
  props[FONT_SIZE] = g_param_spec_int(P_FONT_SIZE,
                                      NULL,
                                      NULL,
                                      FONT_SIZE_MIN,
                                      FONT_SIZE_MAX,
                                      FONT_SIZE_DEF,
                                      G_PARAM_CONSTRUCT | G_PARAM_WRITABLE);
  props[REC] = g_param_spec_pointer(P_REC, NULL, NULL, G_PARAM_CONSTRUCT | G_PARAM_WRITABLE);
  props[WIN] =
      g_param_spec_object(P_WIN, NULL, NULL, WIN_TYPE, G_PARAM_CONSTRUCT_ONLY | G_PARAM_WRITABLE);
  g_object_class_install_properties(gobj_class, PROP_N, props);
  */

  signals[CLICKED] = g_signal_new(
      ON_CLICKED,
      COM_TYPE,
      G_SIGNAL_RUN_LAST,
      0,
      g_signal_accumulator_true_handled, // Stop emission on the 1st TRUE return from multiple slots
      NULL,
      // NULL for g_cclosure_marshal_generic handling arbitrary signatures dynamically. There are
      // also built-in static marshallers like g_cclosure_marshal_VOID__VOID
      NULL,
      G_TYPE_BOOLEAN,
      1,
      G_TYPE_STRING);
}

static void com_init(Com *self) {}

Com *com_new(char const *text, int font_size, Rectangle const *rec, Win *win) {
  /* Create with properties, only necessary for derivable classes
  return g_object_new(COM_TYPE, P_TEXT, text, P_FONT_SIZE, font_size, P_REC, rec, P_WIN, win, NULL);
  */

  // Disabled for release with G_DISABLE_CHECKS defined
  g_return_val_if_fail(text && rec && win, NULL);
  Com *com = COM_FLOATING_OBJ(g_object_new(COM_TYPE, NULL));
  com->text = g_strdup(text);
  com->font_size = font_size;
  com->rec = *rec;
  win_add_com(win, com);
  com->win = win;
  g_object_add_weak_pointer(G_OBJECT(com->win), (gpointer *)&com->win);
  return com;
}

Rectangle const *com_rec(Com const *self) {
  g_return_val_if_fail(self, NULL);
  return &self->rec;
}

void com_render(Com const *self) {
  g_return_if_fail(self);
  DrawRectangleRec(self->rec, GREEN);
  DrawText(self->text, self->rec.x + PADDING, self->rec.y + PADDING, self->font_size, BLACK);
}

void com_clicked(Com *self) {
  g_return_if_fail(self);
  gboolean ret;
  g_signal_emit(self, signals[CLICKED], 0, "emission", &ret);
  TraceLog(LOG_INFO, "Last return: %d", ret);
}
```

#### Window

```c title="src/window.h"
#ifndef WINDOW_H
#define WINDOW_H

#include <glib-object.h>
#include <raylib.h>

typedef struct _Com Com;

G_DECLARE_FINAL_TYPE(Win, win, WIN, OBJ, GObject)
#define WIN_TYPE (win_get_type())

Win *win_new(char const *title, Vector2 const *vec2);
void win_add_com(Win *self, Com *com);
void win_run(Win const *self);

#endif // WINDOW_H
```

```c title="src/window.c"
#include <glib-object.h>
#include <raylib.h>

#include "component.h"
#include "window.h"

typedef struct _ComNode {
  Com *com;
  struct _ComNode *next;
} ComNode;

struct _Win {
  GObject parent_instance;

  ComNode *nodes;
};

G_DEFINE_TYPE(Win, win, G_TYPE_OBJECT)

static void win_finalize(GObject *obj) {
  CloseWindow();
  Win *self = WIN_OBJ(obj);
  for (ComNode *p = self->nodes; p;) {
    ComNode *node = p;
    p = p->next;
    g_object_unref(node->com);
    g_free(node);
  }
  G_OBJECT_CLASS(win_parent_class)->finalize(obj);
}

static void win_class_init(WinClass *klass) {
  GObjectClass *gobj_class = G_OBJECT_CLASS(klass);
  gobj_class->finalize = win_finalize;
}

static void win_init(Win *self) { self->nodes = NULL; }

Win *win_new(char const *title, Vector2 const *vec2) {
  g_return_val_if_fail(title && vec2, NULL);
  Win *win = WIN_OBJ(g_object_new(WIN_TYPE, NULL));
  InitWindow(vec2->x, vec2->y, title);
  return win;
}

void win_add_com(Win *self, Com *com) {
  g_return_if_fail(self && com);
  ComNode *node = g_malloc(sizeof(ComNode));
  node->com = g_object_ref_sink(com);
  node->next = NULL;
  if (!self->nodes) {
    self->nodes = node;
    return;
  }
  ComNode *p = self->nodes;
  for (; p->next; p = p->next)
    ;
  p->next = node;
}

void win_run(Win const *self) {
  g_return_if_fail(self);
  EnableEventWaiting();
  while (!WindowShouldClose()) {
    if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
      Vector2 pos = GetMousePosition();
      for (ComNode *p = self->nodes; p; p = p->next) {
        if (CheckCollisionPointRec(pos, *com_rec(p->com))) {
          com_clicked(p->com);
        }
      }
    }
    BeginDrawing();
    ClearBackground(WHITE);
    for (ComNode *p = self->nodes; p; p = p->next) {
      com_render(p->com);
    }
    EndDrawing();
  }
}
```

#### Use and Customisation

```c title="src/main.c"
#include <raylib.h>

#include "component.h"
#include "window.h"

#define FONT_SIZE 20
#define WIDTH 200
#define HEIGHT 60
#define MARGIN 8

static gboolean on_com1_clicked_0(Com *com, gchar const *emission, gpointer user_data) {
  g_return_val_if_fail(com && emission && user_data, FALSE);
  TraceLog(LOG_INFO,
           "Business 0 triggered by com1 clicked at %f with %s via %s",
           com_rec(com)->y,
           emission,
           (char *)user_data);
  return FALSE;
}

static gboolean on_com1_clicked_1(Com *com, gchar const *emission, gpointer user_data) {
  g_return_val_if_fail(com && emission && user_data, FALSE);
  TraceLog(LOG_INFO,
           "Business 1 triggered by com1 clicked at %f with %s via %s",
           com_rec(com)->y,
           emission,
           (char *)user_data);
  return FALSE;
}

static gboolean on_com2_clicked(Com *com, gchar const *emission, gpointer user_data) {
  g_return_val_if_fail(com && emission, FALSE);
  TraceLog(LOG_INFO, "Business triggered by com2 clicked at %f with %s", com_rec(com)->y, emission);
  return FALSE;
}

int main(int argc, char *argv[]) {
  Win *win = win_new("Components", &(Vector2){WIDTH + MARGIN * 2, HEIGHT * 2 + MARGIN * 3});
  Com *com1 = com_new("Component 1", FONT_SIZE, &(Rectangle){MARGIN, MARGIN, WIDTH, HEIGHT}, win);
  Com *com2 = com_new(
      "Component 2", FONT_SIZE, &(Rectangle){MARGIN, HEIGHT + MARGIN * 2, WIDTH, HEIGHT}, win);

  g_signal_connect(com1, ON_CLICKED, G_CALLBACK(on_com1_clicked_0), (gpointer) "connection a");
  g_signal_connect(com1, ON_CLICKED, G_CALLBACK(on_com1_clicked_1), (gpointer) "connection b");
  g_signal_connect(com2, ON_CLICKED, G_CALLBACK(on_com2_clicked), NULL);

  win_run(win);

  g_object_unref(win);
  return 0;
}
```
