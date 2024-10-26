---
title: "Lights Out!"
excerpt: "Two versions of the children's game"
collection: coding-portfolio
date: 2022-08-19
---


# Intro, Java version

Lights Out is a puzzle game originally released by Tiger Games as a physical
toy in 1995. As a senior in highschool I built a version of the game in Java
for a course project in AP Computer Science.

Lights Out has a few interesting variants and properties. Lights Out features
a 5x5 grid of lights. Clicking on any individual light toggles it, as well as the
four adjacent lights.
"Lights Out Delux" follows the same rules but on a 6x6 board.

Randomly generating Lights Out boards has some interesting properties. For a 6x6
board, any given board is always solveable. For a 5x5 board, only some boards
are solveable. Solveable boards are all orthogonal to two known vectors
[(Anderson and Feil)](https://doi.org/10.1080%2F0025570X.1998.11996658).

![Java Lights Out](/images/code/java_lightsout.png)

[View on Github](https://github.com/WardBrian/LightsOut/tree/java/)

# Python Version

A few years later, I also implemented the game in Python
to try out the [py_cui](https://github.com/jwlodek/py_cui)
TUI library.

![Python Lights Out](/images/code/python_lightsout.png)

[View on Github](https://github.com/WardBrian/LightsOut/tree/python/)
