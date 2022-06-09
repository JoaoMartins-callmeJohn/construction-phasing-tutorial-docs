---
layout: page
title: Reacting to Gantt Chart
nav_order: 1
has_children: false
parent: Connecting the chart with the model
permalink: /connecting/reacting/
---

# Reacting to Gantt chart events

With our tasks defined, we need to map each one of them with the proper element rendered in Viewer.
To do that, we need to retrieve the elements properties right after the input if the csv file.

That's our first step, through the [getbulkProperties](https://forge.autodesk.com/en/docs/viewer/v7/reference/Viewing/Model/#getbulkproperties-dbids-options-onsuccesscallback-onerrorcallback) function.
For this sample, we'll map tasks and elements based on the value of a property defined, and this property will be defined by the `config.js`file.
Let's add three fields in the configuration file, one to store the objects, one to store tasks and properties and last one to control wich property we'll use for mapping, just like the snippet below:

```json
"objects": {},
"propFilter": "Type Name",
"mapTaksNProps": {}
```

For this demo we'll use `Type Name`property for mapping.
With that, we can define the method to map tasks and elements.
Add the content below inside `update` function of `PhasingPanel` class:

```js
model.getBulkProperties(dbids, { propFilter: phasing_config.propFilter }, (results) => {
  results.map((result => {
    this.updateObjects(result);
  }))
}, (err) => {
  console.error(err);
});
```

we also need to add `updateObjects` function inside `PhasingPanel` class, as defined below:

```js
updateObjects(result) {
  let currentTaskId = phasing_config.mapTaksNProps[result.properties[0].displayValue];
  if (!!currentTaskId) {
    if (!phasing_config.objects[currentTaskId])
      phasing_config.objects[currentTaskId] = [];

    phasing_config.objects[currentTaskId].push(result.dbId);
  }
}
```

And athat covers the elements, now we need to implement `mapTaksNProps` field with the values from our input.
To do that, we'll take advantage of `addPropToMap` function called every time we read the csv.

Add `addPropToMap` function inside `PhasingPanel` class:

```js
addPropToMap(filterValue, taskId) {
  phasing_config.mapTaksNProps[filterValue] = taskId;
}
```

And reference it inside `lineToObject` function:

```js
this.addPropToMap(parameters[inputHeaders.findIndex(h => h === phasing_config.propFilter)], newObject.id);
```

Now that we have our schedule with tasks and model elements properly mapped, we can take advantage of the events triggered by [Frappe-Gantt](https://frappe.io/gantt).
The first one we can use is `on_click` in a way to isolate proper elements when a task gets clicked.

To achieve that, we need to change `PhasingPanel.js` by adding the contents bellow.

First, we add the listener on the Gantt chart instantiation:

```js
let newGantt = new Gantt("#phasing-container", phasing_config.tasks, {
  on_click: this.barCLickEvent.bind(this)
});
```

Now, we add the referred `barCLickEvent` function:

```js
barCLickEvent(task) {
  this.extension.viewer.isolate(phasing_config.objects[task.id]);
}
```

And now we're able to isolate linked elements by double-clicking in a task, just like in the gif below:

![First Step Result](../../assets/images/doubleclick.gif)

In the next step we'll impletent methods to handle elements coloring based on their progress.

[Next step - Handling elements and bars colors](/connecting/handlingcolors/){: .btn}