---
layout: page
title: Handling elements and bars colors
nav_order: 2
has_children: false
parent: Connecting the chart with the model
permalink: /connecting/handlingcolors/
---

# Handling elements and bars colors

In order to override the model and bars colors based on the Gantt chart configuration, we need a method to calculate the status of a task, a field in the configuration file to define the colors for each status and a functions reacting to the events triggered when we change the chart.
Let's go do it!
First we need to increment `config.js` adding the fields below:

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
  },
  //Start of the new content
  "statusColors": {
    "finished": "31,246,14",
    "inProgress": "235,246,14",
    "late": "246,55,14",
    "notYetStarted": "",
    "advanced": "14,28,246"
  }
  //End of the new content
}
```

Note that this will define the color of each status, and the numbers define the value for the red, green and blue components of the color to be used.
Now we need the function to obtain the status for each task. Let's add the `checkStatus` function inside `PhasingPanel` class:

```js
...
addPropToMap(filterValue, taskId) {
  phasing_config.mapTaksNProps[filterValue] = taskId;
}
//Start of the new content
checkTaskStatus(task) {
  let currentDate = new Date();

  let taskStart = new Date(task._start);
  let taskEnd = new Date(task._end);

  let shouldHaveStarted = currentDate > taskStart;
  let shouldHaveEnded = currentDate > taskEnd;

  let taskProgress = parseInt(task.progress, 10);

  //We need to map finished, in progress, late, not yet started or advanced
  //finished should have started and ended and actually ended (progress 100%)
  //in progress should have started, not ended and progress should be greater than 0
  //late should have started and ended but progress is less than 100, or should have started not ended and progress is 0
  //not yet started should not have started nor ended and progress is 0
  //advanced should not have started and have progress greater than 0 or should not have ended and progress is 100

  if (shouldHaveStarted && shouldHaveEnded && taskProgress === 100)
    return 'finished';

  else if (shouldHaveStarted && !shouldHaveEnded) {
    switch (taskProgress) {
      case 100:
        return 'advanced';
      case 0:
        return 'late';
      default:
        return 'inProgress';
    }
  }

  else if (shouldHaveStarted && shouldHaveEnded && taskProgress < 100)
    return 'late';

  else if (!shouldHaveStarted && !shouldHaveEnded && taskProgress === 0)
    return 'notYetStarted';

  else if (!shouldHaveStarted && taskProgress > 0)
    return 'advanced';
}
//End of the new content
```

Now we just need to call this method whenever the gantt chart gets changed. For that, we can take advantage of `on_progress_change` and `on_date_change` events from our library, by modifying the Gantt chart instantiation once more:

```js
...
createGanttChart() {
  document.getElementById('phasing-container').innerHTML = `<svg id="phasing-container"></svg>`;

  let newGantt = new Gantt("#phasing-container", phasing_config.tasks, {
    on_click: this.barCLickEvent.bind(this),
    //Start of the new content
    on_progress_change: this.handleColors.bind(this),
    on_date_change: this.handleColors.bind(this)
    //End of the new content
  });

  return newGantt;
}

//Start of the new content
handleColors() {
  this.handleElementsColor.call(this);
  this.handleBarsColor.call(this);
}

handleElementsColor() {
  const overrideCheckbox = document.getElementById('colormodel');
  if (overrideCheckbox.checked) {
    let tasksNStatusArray = this.gantt.tasks.map(this.checkTaskStatus);
    let mappeddbIds = [];
    for (let index = 0; index < this.gantt.tasks.length; index++) {
      const currentTaskId = this.gantt.tasks[index].id;
      const currentdbIds = phasing_config.objects[currentTaskId];
      const colorVector4 = this.fromRGB2Color(phasing_config.statusColors[tasksNStatusArray[index]]);
      currentdbIds.forEach(dbId => {
        if (colorVector4) {
          this.extension.viewer.setThemingColor(dbId, colorVector4)
        }
        else {
          this.extension.viewer.hide(dbId);
        }
      });
      mappeddbIds.push(...currentdbIds);
    }
    this.extension.viewer.isolate(mappeddbIds);
  }
  else {
    this.extension.viewer.clearThemingColors();
    this.extension.viewer.showAll();
  }
}

handleBarsColor() {
  this.gantt.bars.map(bar => {
    let tasksStatus = this.checkTaskStatus(bar.task);
    let barColor = phasing_config.statusColors[tasksStatus];
    bar.$bar.style = `fill: rgb(${barColor})`;
  })
}

fromRGB2Color(rgbString) {
  if (rgbString) {
    let colorsInt = rgbString.replaceAll(' ', '').split(',').map(colorString => parseInt(colorString, 10));
    return new THREE.Vector4(colorsInt[0] / 255, colorsInt[1] / 255, colorsInt[2] / 255, 0.5);
  }
  else {
    return null;
  }
}
//End of the new content
...
```

To manage the bars colors as soon as we have the input loaded, we need to increment the `update` function from `PhasingPanel` class:

```js
...
update(model, dbids) {
  if (phasing_config.tasks.length === 0) {
    this.inputCSV();
  }
  model.getBulkProperties(dbids, { propFilter: phasing_config.propFilter }, (results) => {
    results.map((result => {
      this.updateObjects(result);
    }))
  }, (err) => {
    console.error(err);
  });
  if (phasing_config.tasks.length > 0) {
    this.gantt = this.createGanttChart();
    //Start of the new content
    this.handleColors.call(this);
    //End of the new content
  }
}
...
```

Now there's only one part missing. How we'll control the color of the elements based on the status?

For that, we can add a checkbox in our panel.
Add the content below inside the `initialize` function of the `PhasingPanel` class:

```js
...
initialize() {
  this.title = this.createTitleBar(this.titleLabel || this.container.id);
  this.title.style.overflow = 'auto';
  this.initializeMoveHandlers(this.title);
  this.container.appendChild(this.title);

  this.div = document.createElement('div');
  this.container.appendChild(this.div);

  //Start of the new content

  //Here we create a switch to control vision of the schedule based on the GANTT chart
  this.checkbox = document.createElement('input');
  this.checkbox.type = 'checkbox';
  this.checkbox.id = 'colormodel';
  this.checkbox.style.width = (this.options.checkboxWidth || 30) + 'px';
  this.checkbox.style.height = (this.options.checkboxHeight || 28) + 'px';
  this.checkbox.style.margin = '0 0 0 ' + (this.options.margin || 5) + 'px';
  this.checkbox.style.verticalAlign = (this.options.verticalAlign || 'middle');
  this.checkbox.style.backgroundColor = (this.options.backgroundColor || 'white');
  this.checkbox.style.borderRadius = (this.options.borderRadius || 8) + 'px';
  this.checkbox.style.borderStyle = (this.options.borderStyle || 'groove');

  this.checkbox.onchange = this.handleColors.bind(this);
  this.div.appendChild(this.checkbox);

  //End of the new content

  //Here we add the svg for the GANTT chart
  this.content = document.createElement('div');
  this.content.style.backgroundColor = (this.options.backgroundColor || 'white');
  this.content.innerHTML = `<svg id="phasing-container"></svg>`;
  this.container.appendChild(this.content);

  this.updateTasks();
}
...
```

With all of that defined, you should be able to see the model just like in the gif below:

![Second Step Result](../../assets/images/steptwo.gif)

[Next step - Improving UI & UX]({{ site.baseurl }}/improving/home/){: .btn}