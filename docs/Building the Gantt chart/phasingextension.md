---
layout: page
title: Phasing Extension
nav_order: 1
has_children: false
parent: Building the Gantt chart
permalink: /building/phasingextension/
---

# Phasing Extension

Here we'll add the `PhasingExtension.js` to our app. This extension will extend the `BaseExtension.js`.
Through it, we'll handle the creation of the button, panel and loading the required library (frappe-gantt).
Let's start by creating our panel `PhasingExtension.js` under `wwwroot/extensions` with the content below:

```js
import { BaseExtension } from './BaseExtension.js';
import { PhasingPanel } from './PhasingPanel.js';

class PhasingExtension extends BaseExtension {
    constructor(viewer, options) {
        super(viewer, options);
        this._button = null;
        this._panel = null;
    }

    async load() {
        super.load();
        await Promise.all([
            this.loadScript('https://cdn.jsdelivr.net/npm/frappe-gantt@0.6.1/dist/frappe-gantt.js', 'FrappeGantt'),
            this.loadStylesheet('https://cdn.jsdelivr.net/npm/frappe-gantt@0.6.1/dist/frappe-gantt.css')
        ]);
        console.log('PhasingExtension loaded.');
        return true;
    }

    unload() {
        super.unload();
        if (this._button) {
            this.removeToolbarButton(this._button);
            this._button = null;
        }
        if (this._panel) {
            this._panel.setVisible(false);
            this._panel.uninitialize();
            this._panel = null;
        }
        this.viewer.clearThemingColors();
        this.viewer.showAll();
        console.log('PhasingExtension unloaded.');
        return true;
    }

    onToolbarCreated() {
        this._panel = new PhasingPanel(this, 'dashboard-phases-panel', 'Schedule', { x: 10, y: 10 });//
        this._button = this.createToolbarButton('dashboard-phases-button', 'https://img.icons8.com/small/32/000000/gantt-chart.png', 'Show Gantt Chart');//
        this._button.onClick = () => {
            this._panel.setVisible(!this._panel.isVisible());
            this._button.setState(this._panel.isVisible() ? Autodesk.Viewing.UI.Button.State.ACTIVE : Autodesk.Viewing.UI.Button.State.INACTIVE);
            if (this._panel.isVisible() && this.viewer.model) {
                this.update();
            }
        };
    }

    onModelLoaded(model) {
        super.onModelLoaded(model);
        if (this._panel && this._panel.isVisible()) {
            this.update();
        }
    }

    async update() {
        const dbids = await this.findLeafNodes(this.viewer.model);
        this._panel.update(this.viewer.model, dbids);
    }
}

Autodesk.Viewing.theExtensionManager.registerExtension('PhasingExtension', PhasingExtension);
```

Now we need to add our extension to Viewer.

Add the line below to your js file that's loading the Viewer ():

```
import './extensions/PhasingExtension.js';
```

We also need to add our extension 

```
const config = {
    extensions: ['PhasingExtension']
};
const viewer = new Autodesk.Viewing.GuiViewer3D(container, config);
```

You might have noticed that this extension imports the `PhasingPanel` class. That'll be our next step.

[Next step - Creating the Phasing Panel](/building/phasingpanel/){: .btn}
