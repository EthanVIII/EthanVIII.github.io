---
layout: post
title: Visualising hypercube graphs through animation
description: I watched a talk that described the neat recursive algorithm to construct hypercube graphs using adjacency matrices. I'll explain how I made a quick visualisation of this algorithm. 
---
**Note:** This was written in November 2021 on a different blog and has since been updated.

Recently, I wanted to try making prettier mathematical animations. Through [3Blue1Brown](https://www.youtube.com/c/3blue1brown) I found the Python animation engine `manim`. As I am already familiar with Python, I decided to give it a shot. `manim` is a programmatic animation engine with support for mathematics-specific animation. For my first go, I wanted to choose a topic that was visually interesting that would benefit from visualisation.

I watched a [talk](https://www.youtube.com/watch?v=GDNkDMOxC-I) by the late Gilles Castel, Eline Degryse, and Pieterjan Thijs that briefly described the tidy recursive algorithm for describing hypercubes in graphs which seemed to fit my criteria. Taking some cues from the existing presentation, I made this short clip that animates the full algorithm with added colour.

<iframe width="100%" height="400" src="https://www.youtube.com/embed/4Qt7UC-Bds0?vq=hd1080&showinfo=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; modestbranding" allowfullscreen></iframe>

<br>
To understand this neat little algorithm, we first need to describe graphs and adjacency matrices.

## Graphs & adjacency matrices
A graph $G$ can be defined by two properties. $V$ is the set of vertices or nodes in the graph, and $E$ is the set of edges. Edges are defined as pairs of vertices, and in an undirected graph the edges are bidirectional (e.g. $\\{x,y\\}$ is equivalent to $\\{y,x\\}$).

Using this, we can describe the graph below with $V = \\{v,x,y,z\\}$ and $E=\\{ \\{x,y\\},\\{v,y\\},\\{v,z\\},\\{y,z\\} \\}$.

![A graph with four nodes v,x,y,z. v is connected to y and z, and y is additionally connected to x.](/assets/graph_basic.png){: height="170" }

We can express the edges of a graph with an adjacency matrix, where the value of row $i$ and column $j$ indicate the presence of the edge $\\{i,j\\}$. We obtain the following matrix from our example graph.

![A populated 4 by 4 adjacency matrix with row and column names (v,x,y,z).](/assets/matrix_1.png){: height="170" }

As this is an undirected graph, this results in a symmetric matrix about the principal diagonal.

## The hypercube graph algorithm
Let $A_n$ be the adjacency matrix for a hypercube with dimension $n$. The base case is $A_0=[0]$, where a 0 dimension hypercube has no edges. For $A_{n+1}$, we start with a $2^n \times 2^n$ matrix. We can populate this matrix with this:

$${A_{n+1}= \begin{bmatrix}
A_n & I_{2^{n-1}} \newline
I_{2^{n-1}} & A_n
\end{bmatrix}
}$$

Where $I_{2^{n-1}}$ is an identity matrix with side length $2^{n-1}$. for $n=1$ to $n=4$, this algorithm yields these adjacency matrices.

![Adjacency Matrices from n=1 to n=4](/assets/hypercube_matrix.png){: height="170" }

We then get the following graphs from the adjacency matrices.

![graphs from n=1 to n=4](/assets/graph_shapes.png){: height="170" }

## The animation
With this, I used `manim` to render this short animation, using matching colours between the adjacency matrix cells and the edges generated to better show the relationship betwen the two. 

Because this short animation only went to $n=4$, I simply hardcoded the bulk of the animation. This was tedious and made the code difficult to work with. If I had the chance to redo this project or extend it to higher dimensional cases, it would be easier overall to abstract the generation of the adjacency matrix and graph visualisation to functions.

