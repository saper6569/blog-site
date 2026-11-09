---
layout: post
author: "Sanija Perera"
github: "https://github.com/saper6569"
title: "Plan to Create an Open Source PCB Resistance Dataset" 
date: 2026-11-08
tags: [research] 
---

# Introduction
The main pcb editor I currently use is Kicad, an open source, free editor. One of the main features I quickly found myself searching for was a method of finding how much current a trace could carry safely. Kicad has a built in trace calculator for selecting trace widths that safely allow specified currents while maintaining safe temperatures. While this is sufficient for traces many pcbs require copper pours, which aren't as simple when it comes to calculating safe current limits. 

To allow this functionality I decided to try developing my own plugin that could complete this exact task. With the use of claude for assitance I developed a python based plugin that works out the DC resistance and current capacity of a net, with a focus on electrical behavior. This plugin is aimed aiding the open source power systems design workflow. 

# Plugin 
- How you use it: You pick a net and tick the pads where current enters and leaves. It then computes the resistance along that path, the voltage drop, and how much current the copper can carry for a given temperature rise. Capacity comes from IPC curves, with an IPC-2152 option (ipc_calculations.py).
- Copper pours: Pours are solved as real 2-D conductors from KiCad’s filled zone geometry using a mesh solver (mesh_solver.py), not approximated as rectangles. Vias and multilayer paths are included too.
- Output: The results window has tabs for Results, Layers, Vias, Pours, Bottlenecks, Pads, a Heatmap and Notes. You can export a full .txt report. Each result is graded High, Estimated or Unsupported, and the Notes tab lists what was assumed.
- Dependencies: Nothing beyond what ships with KiCad (numpy, wx, pcbnew). tools/install.py copies it into KiCad’s plugin folder.
- Tests: There’s a test suite with 14 generated test boards, covering straight and arc traces, pours with cutouts, thermal reliefs and hatching, multiple vias, parallel paths and multiple nets. The repo also includes a mutation-testing script and validation docs.

## Future plans
I recognize that there are similar plugins to the one that I developed, so I decided I could validate my plugin while also using physical data to calibrate my solver. I planned on using any dataset that was publicly available to collect data on resistance of pours and planes, however I ran into the issue of not being able to find any. I decided that creating such a a dataset would be more valuable than the software contribution so I decided to pivot from here and instead focus on creating a public data set of measured pour and via resistance, with raw data, board files and measured stackup. 

This will likely take a while before any substantial progress will be made.