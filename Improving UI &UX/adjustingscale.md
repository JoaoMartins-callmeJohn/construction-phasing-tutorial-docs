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
```

If you pay attention, this dropdown calls the `changeViewMode` function, so let's add it inside `PhasingPanel` class:

```js
changeViewMode(event) {
  const viewMode = event ? event.target.value : this.currentViewMode;
  this.gantt.change_view_mode(viewMode);
  this.handleColors.call(this);
  this.currentViewMode = viewMode;
}
```

And to keep the viewmode persistent, we need to add a call to `changeViewMode` inside the `update` function of `PhasingPanel` class:

```js
if (phasing_config.tasks.length > 0) {
  this.gantt = this.createGanttChart();
  this.handleColors.call(this); 
  this.changeViewMode.call(this);<< ADD THIS LINE
}
```

It also takes advantage of `phasing_config.viewModes` from our configuration file. Let's add the available options according to our library inside `config.js`:

```js
"viewModes": [
  // "Quarter Day",
  // "Half Day",
  "Day",
  "Week",
  "Month"
]
```

And that's it, our Gantt chart can now be rescaled to show the tasks in days, weeks or months, just like in the gif below!

![Adjusting the scale](../../assets/images/viewmodes.gif)

[Next step - Handling Panel orientation]({{ site.baseurl }}/improving/orientation/){: .btn}