---
layout: post
title: Visualising hypercube graphs through animation
description: I watched a talk that described a neat recursive algorithm that constructs graph hypercubes using adjacency matrices. I'll explain how I made a quick visualisation of this algorithm. 
---
**Note:** This was written in November 2021 on a different blog and has since been updated.

Recently, I wanted to try making prettier mathematical animations. Through [3Blue1Brown](https://www.youtube.com/c/3blue1brown) I found the Python animation engine `manim`. As I am already familiar with Python, I decided to give it a shot. `manim` is a programmatic animation engine with support for mathematics-specific animation. For my first animation, I wanted to choose a topic that was visually interesting that would benefit from visualisation.

I watched a [talk](https://www.youtube.com/watch?v=GDNkDMOxC-I) by the late Gilles Castel, Eline Degryse, and Pieterjan Thijs that briefly described a tidy recursive algorithm for describing hypercubes in graphs which seemed to fit my criteria. Taking some cues from the existing presentation, I made this short clip that animates the full algorithm with added colour.

<iframe width="100%" height="400" src="https://www.youtube.com/embed/4Qt7UC-Bds0?vq=hd1080&showinfo=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; modestbranding" allowfullscreen></iframe>

<br>
This is a neat little algorithm. Unfortunately, I haven't found many resources talking about it. I did find this [Observable article](https://observablehq.com/@toja/hypercube-graphs) by Torben Jansen very helpful. It goes into more detail than my explanation below.

## Graphs & Adjacency Matrices

A graph $G$ can be defined by two properties. $V$ is the set of vertices or nodes in the graph, and $E$ is the set of edges. Edges are defined as pairs of vertices, and in an undirected graph the edges are bidirectional (e.g. $\\{x,y\\}$ is equivalent to $\\{y,x\\}$).

Using this, we can describe the graph below with $V = \\{v,x,y,z\\}$ and $E=\\{ \\{x,y\\},\\{v,y\\},\\{v,z\\},\\{y,z\\} \\}$.

![A graph with four nodes v,x,y,z. v is connected to y and z, and y is additionally connected to x.](/assets/graph_basic.png){: height="170" }


This algorithm uses undirected graphs, resulting in a symmetric matrix about the principal diagonal.

