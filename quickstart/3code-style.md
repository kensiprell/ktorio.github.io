---
title: Code Style
caption: Code Style
category: quickstart
permalink: /quickstart/code-style.html
toc: false
---

## Official Code Convention

Ktor as well as other official Kotlin libraries use the [official Coding Conventions](https://kotlinlang.org/docs/reference/coding-conventions.html).

You can use the official coding standard by adding `kotlin.code.style=official` to your `gradle.properties` file.

## With Star Imports

The Official Coding Conventions don't define a recommended way of using imports.
The IntelliJ default is to include star (`*`) imports after importing at least 5 symbols from a package.
However, in Ktor we always use them internally and recommend that you do as well.

The rationale behind it is that usually when you include a class, you will probably want to include all the extension methods and properties declared for that class.
That's especially convenient for operator extension methods.

You can change the import configuration in `Preferences... -> Editor -> Code Style -> Kotlin -> Imports`:

![](/quickstart/code-style/code-style-imports.png)
