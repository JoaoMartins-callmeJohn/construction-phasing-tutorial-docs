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
...
update(model, dbids) {
  if (phasing_config.tasks.length === 0) {
    this.inputCSV();
  }
  //Start of the new content
  model.getBulkProperties(dbids, { propFilter: phasing_config.propFilter }, (results) => {
    results.map((result => {
      this.updateObjects(result);
    }))
  }, (err) => {
    console.error(err);
  });
  //End of the new content
  if (phasing_config.tasks.length > 0) {
    this.gantt = this.createGanttChart();
  }
}
...
```

we also need to add `updateObjects` function inside `PhasingPanel` class, as defined below:

```js
...
validateCSV(line) {
  let parameters = line.split(',');
  return Object.values(phasing_config.requiredProps).every((currentProp) => !!parameters.find(p => p === currentProp));
}
//Start of the new content
updateObjects(result) {
  let currentTaskId = phasing_config.mapTaksNProps[result.properties[0].displayValue];
  if (!!currentTaskId) {
    if (!phasing_config.objects[currentTaskId])
      phasing_config.objects[currentTaskId] = [];

    phasing_config.objects[currentTaskId].push(result.dbId);
  }
}
//End of the new content
```

And athat covers the elements, now we need to implement `mapTaksNProps` field with the values from our input.
To do that, we'll take advantage of `addPropToMap` function called every time we read the csv.

Add `addPropToMap` function inside `PhasingPanel` class:

```js
...
updateObjects(result) {
  let currentTaskId = phasing_config.mapTaksNProps[result.properties[0].displayValue];
  if (!!currentTaskId) {
    if (!phasing_config.objects[currentTaskId])
      phasing_config.objects[currentTaskId] = [];

    phasing_config.objects[currentTaskId].push(result.dbId);
  }
}
//Start of the new content
addPropToMap(filterValue, taskId) {
  phasing_config.mapTaksNProps[filterValue] = taskId;
}
//End of the new content
```

And reference it inside `lineToObject` function:

```js
...
//This function converts a line from imported csv into an object to generate the GANTT chart
lineToObject(line, inputHeadersLine) {
  let parameters = line.split(',');
  let inputHeaders = inputHeadersLine.split(',');
  let newObject = {};
  // Object.keys(newObject) = PHASING_CONFIG.requiredProps;
  Object.values(phasing_config.requiredProps).forEach(requiredProp => {
    newObject.id = parameters[inputHeaders.findIndex(h => h === phasing_config.requiredProps.id)];
    newObject.name = parameters[inputHeaders.findIndex(h => h === phasing_config.requiredProps.taskName)];
    newObject.start = parameters[inputHeaders.findIndex(h => h === phasing_config.requiredProps.startDate)];
    newObject.end = parameters[inputHeaders.findIndex(h => h === phasing_config.requiredProps.endDate)];
    newObject.progress = parameters[inputHeaders.findIndex(h => h === phasing_config.requiredProps.taskProgress)];
    newObject.dependencies = parameters[inputHeaders.findIndex(h => h === phasing_config.requiredProps.dependencies)];
    newObject.dependencies.replaceAll('-', ',');
  });
  //Start of the new content
  this.addPropToMap(parameters[inputHeaders.findIndex(h => h === phasing_config.propFilter)], newObject.id);
  //End of the new content
  return newObject;
  }
```

Now that we have our schedule with tasks and model elements properly mapped, we can take advantage of the events triggered by [Frappe-Gantt](https://frappe.io/gantt).
The first one we can use is `on_click` in a way to isolate proper elements when a task gets clicked.

To achieve that, we need to change `PhasingPanel.js` by adding the contents bellow.

First, we add the `on_click` listener on the Gantt chart instantiation inside `createGanttChart` function:

```js
...
createGanttChart() {
  document.getElementById('phasing-container').innerHTML = `<svg id="phasing-container"></svg>`;

  let newGantt = new Gantt("#phasing-container", phasing_config.tasks, 
  //Start of the new content
  {
    on_click: this.barCLickEvent.bind(this)
  }
  );
  //End of the new content
  return newGantt;
}
//Start of the new content
barCLickEvent(task) {
  this.extension.viewer.isolate(phasing_config.objects[task.id]);
}
//End of the new content
...
```

And now we're able to isolate linked elements by double-clicking in a task, just like in the gif below:

![First Step Result](../../assets/images/doubleclick.gif)

In the next step we'll impletent methods to handle elements coloring based on their progress.

[Next step - Handling elements and bars colors]({{ site.baseurl }}/connecting/handlingcolors/){: .btn}