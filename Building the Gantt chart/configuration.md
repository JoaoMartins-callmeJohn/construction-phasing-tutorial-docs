---
layout: page
title: Configuration File
nav_order: 3
has_children: false
parent: Building the Gantt chart
permalink: /building/configuration/
---

# Configuration File

The configuration file defines how our extension will read the csv file, arrange and connect data with our model.
In this first step we need to define the property we'll use for filtering, the required properties expected from a csv input and the variables to store tasks, objects maped to tasks and a dictionary for mapping.
Let's improve our configuration file `config.js` under `wwwroot/extensions` with the content below:

```js
export const phasing_config = {
  "propFilter": "Type Name",
  "tasks": [],
  "requiredProps": {
    "id": "ID",
    "taskName": "NAME",
    "startDate": "START",
    "endDate": "END",
    "taskProgress": "PROGRESS",
    "dependencies": "DEPENDENCIES"
  }
}
```

With that, we're able to check our brand new Gantt chart generated from our csv input.

![First Step Result](../../assets/images/stepone.gif)

[Next step - Connecting the chart with the model]({{ site.baseurl }}/connecting/home/){: .btn}