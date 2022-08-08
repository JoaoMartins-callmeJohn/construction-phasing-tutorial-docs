---
layout: page
title: Input, Libraries and Configuration file
permalink: /input/index
nav_order: 2
---

# Input, Libraries and Configuration file

To generate the Gantt chart connected with our models, we need a library to generate the chart and a way to implement additional information related to the tasks that each element will be part of.

For the library, we choose [Frappe/Gantt](https://frappe.io/gantt) for its simplicity and flexibility. <br /> This library requires the following object to define a task:

```
{
  id: 'Task id',
  name: 'Task name',
  start: 'Task start',
  end: 'Task end',
  progress: Task_progress,
  dependencies: 'Tasks dependencies'
}
```

We need all of these information to generate each task, and we also need a way to map tasks and the elements of our model. In this sample, we'll use a property to do this mapping (controlled by the configuration file `config.js`). <br /> In the end, we'll have the following configuration for our csv:

| ID  | NAME          | START      | END        | PROGRESS | Type Name              | DEPENDENCIES |
| --- | ------------- | ---------- | ---------- | -------- | ---------------------- | ------------ |
| 123 | Timber Floors | 2022-03-10 | 2022-04-04 | 100      | Timber Suspended Floor | 121-122      |

With this csv we have all the information required by the library and the property used to connect with the model (**Type Name**).

We also need the configuration file (`config.js`). The config contain information that we'll use across different sections of this tutorial, and we'll cover each part here. For now let's focus on the property used for mapping tasks and elements. This is controlled by the `propFilter` field:

For now, you can create a `config.js` file under `wwwroot/extensions` folder with the content below:

```js
export const phasing_config = {
  propFilter: "Type Name",
};
```

We can have an idea on how it's gonna work through the tutorial, as we'll keep improving `config.js` and our extension.

As said in home page, you'll also need the `BaseExtension.js` for this sample to work.
Add the file under the `extensions` folder of your project with the content below.

```js
export class BaseExtension extends Autodesk.Viewing.Extension {
  constructor(viewer, options) {
    super(viewer, options);
    this._onObjectTreeCreated = (ev) => this.onModelLoaded(ev.model);
    this._onSelectionChanged = (ev) => this.onSelectionChanged(ev.model, ev.dbIdArray);
    this._onIsolationChanged = (ev) => this.onIsolationChanged(ev.model, ev.nodeIdArray);
  }

  load() {
    this.viewer.addEventListener(Autodesk.Viewing.OBJECT_TREE_CREATED_EVENT, this._onObjectTreeCreated);
    this.viewer.addEventListener(Autodesk.Viewing.SELECTION_CHANGED_EVENT, this._onSelectionChanged);
    this.viewer.addEventListener(Autodesk.Viewing.ISOLATE_EVENT, this._onIsolationChanged);
    return true;
  }

  unload() {
    this.viewer.removeEventListener(Autodesk.Viewing.OBJECT_TREE_CREATED_EVENT, this._onObjectTreeCreated);
    this.viewer.removeEventListener(Autodesk.Viewing.SELECTION_CHANGED_EVENT, this._onSelectionChanged);
    this.viewer.removeEventListener(Autodesk.Viewing.ISOLATE_EVENT, this._onIsolationChanged);
    return true;
  }

  onModelLoaded(model) {}

  onSelectionChanged(model, dbids) {}

  onIsolationChanged(model, dbids) {}

  findLeafNodes(model) {
    return new Promise(function (resolve, reject) {
      model.getObjectTree(function (tree) {
        let leaves = [];
        tree.enumNodeChildren(tree.getRootId(), function (dbid) {
          if (tree.getChildCount(dbid) === 0) {
            leaves.push(dbid);
          }
        }, true);
        resolve(leaves);
      }, reject);
    });
  }

  async findPropertyNames(model) {
    const dbids = await this.findLeafNodes(model);
    return new Promise(function (resolve, reject) {
      model.getBulkProperties(dbids, {}, function (results) {
        let propNames = new Set();
        for (const result of results) {
          for (const prop of result.properties) {
            propNames.add(prop.displayName);
          }
        }
        resolve(Array.from(propNames.values()));
      }, reject);
    });
  }

  createToolbarButton(buttonId, buttonIconUrl, buttonTooltip) {
    let group = this.viewer.toolbar.getControl('dashboard-toolbar-group');
    if (!group) {
      group = new Autodesk.Viewing.UI.ControlGroup('dashboard-toolbar-group');
      this.viewer.toolbar.addControl(group);
    }
    const button = new Autodesk.Viewing.UI.Button(buttonId);
    button.setToolTip(buttonTooltip);
    group.addControl(button);
    const icon = button.container.querySelector('.adsk-button-icon');
    if (icon) {
      icon.style.backgroundImage = `url(${buttonIconUrl})`; 
      icon.style.backgroundSize = `24px`; 
      icon.style.backgroundRepeat = `no-repeat`; 
      icon.style.backgroundPosition = `center`; 
    }
    return button;
  }

  removeToolbarButton(button) {
    const group = this.viewer.toolbar.getControl('dashboard-toolbar-group');
    group.removeControl(button);
  }

  loadScript(url, namespace) {
    if (window[namespace] !== undefined) {
      return Promise.resolve();
    }
    return new Promise(function (resolve, reject) {
      const el = document.createElement('script');
      el.src = url;
      el.onload = resolve;
      el.onerror = reject;
      document.head.appendChild(el);
    });
  }

  loadStylesheet(url) {
    return new Promise(function (resolve, reject) {
      const el = document.createElement('link');
      el.rel = 'stylesheet';
      el.href = url;
      el.onload = resolve;
      el.onerror = reject;
      document.head.appendChild(el);
    });
  }
}
```

Now that we know how it's gonna work and the project dependencies, we can start building our sample.

[Next step - Building the Gantt chart]({{ site.baseurl }}/building/home){: .btn}
