---
layout: page
title: Handling Panel orientation
nav_order: 2
has_children: false
parent: Improving UI & UX
permalink: /improving/orientation/
---

# Handling Panel orientation

Now is time to address the UI & UX of our chart.
For that, we'll add two buttons that will dock our panel to the left and to the bottom of our screen.
You just need to add the snippet below inside `initialize` function of the `PhasingPanel` class:

```js
this.dockleft = document.createElement('img');
//https://icons8.com/icon/105589/left-docking
this.dockleft.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADAAAAAwCAYAAABXAvmHAAAABmJLR0QA/wD/AP+gvaeTAAABGklEQVRoge2ZMQ6CQBBFn8bGxgNI7Rltbb2JFvbeiZgIjSUWsIYQdiMws8sm85IpDOuf/7MTNgAYhmGsmRPwAGqgWVg1cAeKmOZfAsaHVcYK8VAw7+oWI4DE2PjqLWVyE7jWTFj7D9J6AGwlRFJiAVKzm7B2OMOrIPsdyCXAZc6ftM4AV1PMzxrfuQ0l9Zz5LAP0zWcXYGg+qwBj5rMJ4DOfRYCQeW//NZ0DH2nBFCN0HlmXzQg5fCHUGmrojYVQbaihNwyh3lBDrx8iSkMNPRdilCkPNKm4AnvfRXsrkRoLkBoLkJrQbbQGDr3fEmeBo5ISCu3AU6pJZO0fBe3HiNBDxpwqgWOMAHSNbrRbvtR41WlFM28YhqHPF0NRPAWhEg4IAAAAAElFTkSuQmCC';
this.dockleft.style.width = (this.options.imageWidth || 30) + 'px';
this.dockleft.style.height = (this.options.imageHeight || 28) + 'px';
this.dockleft.style.margin = '0 0 0 ' + (this.options.margin || 10) + 'px';
this.dockleft.style.verticalAlign = (this.options.verticalAlign || 'middle');
this.dockleft.onclick = this.toggleOrientation.bind(this, false);
this.div.appendChild(this.dockleft);

this.dockbottom = document.createElement('img');
//https://icons8.com/icon/105606/bottom-docking
this.dockbottom.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADAAAAAwCAYAAABXAvmHAAAABmJLR0QA/wD/AP+gvaeTAAAA+0lEQVRoge2XsQ6CMBCGP9RV4yyzz+hofBsc2F0dfBriAC+AAzQhBBp6plxJ7ktuIC30+3OQUjAMw9g6N6AV1l3BdxJJiGTkHSEhkpN3LAmRrLzDFyJ5ecdUiM3IO4YhosnvYz0Y+AAZ8AYeEdcxDMPDFSiBBvn/jrQa4Ank/8h/FcTHVUlDlAnIuyrmJDNPgAY4BoaORQ2cpwZ8AdqAuTFYtP5uBZGoWABtDgFzx+9kEmy+AxZAm5BvQHsfmGTzHbAA2lgAbSyANr59oAFOg2vNf6F6bsDXgVcEESkil5zuQK19Hq6AiyQA/Y0FXQvXFq/7tcXyhmEY8fkBDhLw1fWJwhgAAAAASUVORK5CYII=';
this.dockbottom.style.width = (this.options.imageWidth || 30) + 'px';
this.dockbottom.style.height = (this.options.imageHeight || 28) + 'px';
this.dockbottom.style.margin = '0 0 0 ' + (this.options.margin || 10) + 'px';
this.dockbottom.style.verticalAlign = (this.options.verticalAlign || 'middle');
this.dockbottom.onclick = this.toggleOrientation.bind(this, true);
this.div.appendChild(this.dockbottom);
```

And also the `toggleOrientation` function inside the `PhasingPanel` class:

```js
toggleOrientation(isVertical) {
  const { left: startX, top: startY, right: endX, bottom: endY } = this.extension.viewer.impl.getCanvasBoundingClientRect();

  let defaultPanelHeigth = (this.options.height || 400);
  let defaultPanelWidth = (this.options.width || 500);

  if (isVertical) {
    this.container.style.width = (endX - startX - 20) + 'px';
    this.container.style.height = defaultPanelHeigth + 'px';
    this.container.style.left = (this.options.x || 0) + 'px';
    this.container.style.top = (endY - startY - defaultPanelHeigth - this.options.y) + 'px';
  }
  else {
    this.container.style.width = defaultPanelWidth + 'px';
    this.container.style.height = (endY - startY - 20) + 'px';
    this.container.style.left = (this.options.x || 0) + 'px';
    this.container.style.top = (this.options.y || 0) + 'px';
  }
}
```

![Handling Panel orientation](../../assets/images/orientation.gif)

[Next step - Update CSV]({{ site.baseurl }}/improving/csv/){: .btn}