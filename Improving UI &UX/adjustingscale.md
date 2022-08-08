---
layout: page
title: Adjusting the scale
nav_order: 1
has_children: false
parent: Improving UI & UX
permalink: /improving/scale/
---

# Adjusting the scale

We can start this part by adding a [dropdown](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/select) to our panel, specifically inside the `initialize` function of `PhasingPanel` class:

```js
...
initialize() {
  this.title = this.createTitleBar(this.titleLabel || this.container.id);
  this.title.style.overflow = 'auto';
  this.initializeMoveHandlers(this.title);
  this.container.appendChild(this.title);

  this.div = document.createElement('div');
  this.container.appendChild(this.div);

  ========START OF THE  ADDITIONAL CONTENT========

  //Here we create a dropdown to control vision of the GANTT
  this.dropdown = document.createElement('select');
  this.dropdown.style.width = (this.options.dropdownWidth || 100) + 'px';
  this.dropdown.style.height = (this.options.dropdownHeight || 28) + 'px';
  this.dropdown.style.margin = (this.options.margin || 5) + 'px';
  this.dropdown.style.verticalAlign = (this.options.verticalAlign || 'middle');
  this.dropdown.style.backgroundColor = (this.options.backgroundColor || 'white');
  this.dropdown.style.borderRadius = (this.options.borderRadius || 8) + 'px';
  this.dropdown.style.borderStyle = (this.options.borderStyle || 'groove');
  for (const viewMode of phasing_config.viewModes) {
    let currentOption = document.createElement('option');
    currentOption.value = viewMode;
    currentOption.innerHTML = viewMode;
    this.dropdown.appendChild(currentOption);
  }

  this.dropdown.onchange = this.changeViewMode.bind(this);
  this.div.appendChild(this.dropdown);

  ========END OF THE  ADDITIONAL CONTENT========

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

  //Here we add the svg for the GANTT chart
  this.content = document.createElement('div');
  this.content.style.backgroundColor = (this.options.backgroundColor || 'white');
  this.content.innerHTML = `<svg id="phasing-container"></svg>`;
  this.container.appendChild(this.content);

  this.updateTasks();
}

========START OF THE  ADDITIONAL CONTENT========
changeViewMode(event) {
  const viewMode = event ? event.target.value : this.currentViewMode;
  this.gantt.change_view_mode(viewMode);
  this.handleColors.call(this);
  this.currentViewMode = viewMode;
}
========END OF THE  ADDITIONAL CONTENT========
...
```

And to keep the viewmode persistent, we need to add a call to `changeViewMode` inside the `update` function of `PhasingPanel` class:

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
    this.handleColors.call(this);
    ========START OF THE  ADDITIONAL CONTENT========
    this.changeViewMode.call(this);
    ========END OF THE  ADDITIONAL CONTENT========
  }
}
...
```

It also takes advantage of `phasing_config.viewModes` from our configuration file. Let's add the available options according to our library inside `config.js`:

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
  "statusColors": {
    "finished": "31,246,14",
    "inProgress": "235,246,14",
    "late": "246,55,14",
    "notYetStarted": "",
    "advanced": "14,28,246"
  },
  ========START OF THE  ADDITIONAL CONTENT========
  "viewModes": [
    // "Quarter Day",
    // "Half Day",
    "Day",
    "Week",
    "Month"
  ========END OF THE  ADDITIONAL CONTENT========
]
}

```

And that's it, our Gantt chart can now be rescaled to show the tasks in days, weeks or months, just like in the gif below!

![Adjusting the scale](../../assets/images/viewmodes.gif)

[Next step - Handling Panel orientation]({{ site.baseurl }}/improving/orientation/){: .btn}