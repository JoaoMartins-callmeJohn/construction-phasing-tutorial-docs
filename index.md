---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

# Construction Phasing Tutorial

## Introduction

In this tutorial we'll be focusing on building a sample for construction phasing inside Viewer through an extension.

In details, we'll basically render a model (managed by you Forge app or Autodesk SaaS applications, refer [here](https://forge.autodesk.com/en/docs/data/v2/developers_guide/basics/)) on the browser with Viewer, and then adding a [Gantt chart](https://en.wikipedia.org/wiki/Gantt_chart) connected with the model's, in a way that we can check the objects related to each task, having a graphical visual of our schedule.

Cool, right, but for that we'll need some pre-requisited in order to take the most of our efforts:

1. First of all, if you're taking your first steps with Forge, please go through [Getting Started](https://forge-tutorials.autodesk.io) and [Environment Setup](https://forge-tutorials.autodesk.io/setup/).

2. This tutorial will focus on building a [Viewer Extension](https://forge.autodesk.com/en/docs/viewer/v7/developers_guide/viewer_basics/extensions/), not the whole app. If you don't have an app, you can take advantage of [Simple Viewer](https://forge-tutorials.autodesk.io/tutorials/simple-viewer/) or [Hubs Browser](https://forge-tutorials.autodesk.io/tutorials/hubs-browser/) in either Node.js or .NET.

3. To build this extension, we'll take advantage of the [Basic Extension](https://forge-tutorials.autodesk.io/tutorials/dashboard/basic) from Forge tutorial.

4. If you don't have a model for testing, feel free to grab one of our [sample files](https://knowledge.autodesk.com/support/revit/getting-started/caas/CloudHelp/cloudhelp/2022/ENU/Revit-GetStarted/files/GUID-7B9C7A69-1083-406D-A01F-53D405C167F3-htm.html) (we strongly recomend a Revit sample).

Once you have everything steps 1, 2 & 3 covered, you can go through this tutorial.

It's divided in the steps below:

1. [Input and Libraries](/Input.md)

2. Building the Gantt chart

3. Connecting the chart with the model

4. Improving UI & UX
