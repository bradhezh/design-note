---
title: Outline
slug: /
---

Looking into language mechanics and framework ergonomics, alongside typical and practical programming paradigms, design patterns, and use cases.

- Chapter 1. [Language and Frameworks](/lang/intro)
  - 1.1. [The Starting Point](/lang/intro)  
    From general design ideas to the type system  
    Cases: C as the baseline
  - 1.2. [Frameworks and Dynamic Types](/lang/dynamic)  
    Why and how types become dynamic for framework binding  
    Cases: C++ virtual functions, Qt/GObject signal-slot mechanics, and Objective-C messaging
  - 1.3. [Decorators — Declarative Elegance](/lang/decorator)  
    How metaprogramming (metadata and reflection) emerges  
    How decorators make framework binding declarative  
    Cases: DI frameworks such as Java Spring Boot and C# ASP.NET Core
  - 1.4. [Rust — Pure Compilation](/lang/rust)  
    Metaprogramming: Templates vs. decorators  
    Rust's improvements over traditional templates, and how it achieves decorator-like declarative elegance in pure compilation via procedural macros and traits  
    Cases: C++ templates, Rust procedural macros and traits, as well as Rust declarative frameworks such as Axum or Actix Web
  - 1.5. [JavaScript — Value-Based Typing](/lang/javascript)  
    As the other end of the spectrum, how value-based typing keeps the data-and-process paradigm effective  
    Two parallel paradigms: Global objects + functions and decorator-based DI  
    Cases: React, Express, Angular, and NestJS
  - 1.6. [Convergence](/lang/converge)  
    How modern languages borrow from and converge with each other for type safety, performance, and ergonomics  
    Cases: Type inference and type checking, as well as JIT and AOT
- Chapter 2. [Event-Driven Architectures](/event/intro)
  - 2.1. [Synchronous and Asynchronous](/event/intro)
  - 2.2. [The Event Loop](/event/loop)
