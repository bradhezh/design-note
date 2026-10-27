---
title: Outline
slug: /
---

Looking into language mechanics and framework ergonomics, alongside typical and practical programming paradigms, design patterns, and use cases.

- Chapter 1. [Language and Frameworks](/lang/intro)
  - 1.1. [The Starting Point](/lang/intro)  
    From general design ideas to the type system  
    Cases: C/C++ classes
  - 1.2. [Frameworks and Dynamic Types](/lang/dynamic)  
    Why and how types become dynamic for framework binding  
    Cases: Callbacks, signals, and virtual functions in C/C++ and GObject
  - 1.3. [Decorators, Reflection, and Metaprogramming](/lang/decorator)  
    How decorators, reflection, and DI introduce a declarative style to web backend design  
    How metaprogramming evolves with source generation  
    Declarative vs. imperative  
    Declarative GUI  
    GUI architectures: MVC and MVVM  
    Cases: ASP.NET Controllers and Minimal APIs, .NET XAML and Avalonia, as well as GTK
  - 1.4. [Rust — Pure Compilation](/lang/rust)  
    Metaprogramming: Templates vs. decorators  
    Rust's improvements over traditional templates, and how it achieves decorator-like declarative elegance in pure compilation via procedural macros and traits  
    Cases: C++ templates, Rust procedural macros and traits, as well as Rust declarative frameworks such as Axum and Actix Web
  - 1.5. [JavaScript — Value-Based Typing](/lang/javascript)  
    How dynamic typing evolves into being value-based  
    How value-based typing keeps the data-and-process paradigm effective  
    Two parallel styles: object-and-function and decorator-based DI
  - 1.6. [Convergence](/lang/converge)  
    How modern languages borrow from and converge with each other for type safety, performance, and ergonomics  
    Cases: Type inference and type checking, as well as JIT and AOT
- Chapter 2. [Event-Driven Architectures](/event/intro)
  - 2.1. [Synchronous and Asynchronous](/event/intro)
  - 2.2. [The Event Loop](/event/loop)
