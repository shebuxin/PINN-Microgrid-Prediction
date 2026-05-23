# Foundations Lecture Series

A seven-lecture sequence on the prerequisites needed to follow the project-driven
microgrid security course. Each lecture is a self-contained Beamer deck plus
(optionally) a hands-on notebook.

## Lecture order

| # | Lecture | What you will learn |
|---|---|---|
| L01 | Power Flow Fundamentals | Bus types, Y-matrix, NR, the general AC PF problem |
| L02 | Distribution Network Power Flow | Radial topology, BFS, R/X ratio, DER impact |
| L03 | Balanced Power Flow | Single-line equivalent, positive sequence, when balance holds |
| L04 | Unbalanced Power Flow | Symmetrical components, three-phase PF, VUF |
| L05 | Multilayer Perceptron (MLP) | Layers, activations, backprop, training discipline |
| L06 | Graph Neural Networks (GNN) | Message passing, GCN/SAGE/GAT, graph-level readouts |
| L07 | Physics-informed Neural Networks (PINN) | Soft physics constraints, limit-aware loss design |

## Audience

Undergraduate EE/CS students with circuit basics and Python literacy, but no
prior exposure to power systems analysis or machine learning. Lectures are
self-contained but build on each other.

## Status

| Lecture | Slides | Notebook |
|---|---|---|
| L01 | full | --- |
| L02--L07 | skeleton (compileable, TODO bullets) | --- |

The skeleton decks list the planned frames and section structure. They compile to
short PDFs that preview the lecture outline.

## How to compile

```bash
cd L01_power_flow_fundamentals/slides
pdflatex L01_power_flow_fundamentals.tex
pdflatex L01_power_flow_fundamentals.tex   # second pass for outlines
```
