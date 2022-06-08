---
layout: page
title: Input, Libraries and Configuration file
permalink: /input/index
nav_order: 1
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

Now that we know how it's gonna work and the project dependencies, we can start building our sample.

[Next step](/building/home){: .btn}
