*From this [page](https://www.jspsych.org/overview/plugins/) of the jsPsych Documentation*

# Creating a new plugin
You can add new kinds of tasks to jsPsych by creating new plugins, or modifying existing plugins. A task can be virtually any kind of activity. If it can be implemented in JavaScript, then it almost certainly can be turned into a jsPsych plugin.

What's in a plugin file?
Plugin files follow a specific template. Adherence to the template is what allows jsPsych to run a plugin without knowing anything about what the plugin is doing. What makes plugins so flexible is that the template imposes very few requirements on the code. Here's what an empty plugin template looks like:


```javascript
jsPsych.plugins['plugin-name'] = (function(){

  var plugin = {};

  plugin.info = {
    name: 'plugin-name',
    parameters: {
    }
  }

  plugin.trial = function(display_element, trial){
    jsPsych.finishTrial();
  }

  return plugin;

})();
```

This plugin will work! It defines a plugin called 'plugin-name', and it does absolutely nothing. However, it won't break the experiment, and jsPsych will understand that this is a valid plugin.

Let's examine it in more detail.

The overall structure of the plugin is defined using a module JavaScript design pattern. This pattern uses a technique called an anonymous closure. This is why the first line has (function(){ and the last line is })();. The details aren't important, but if you want to learn more about it, this is a nice overview. The reason this pattern is useful is because it allows for persistent state and private scope. In other words, the plugin is isolated and can't be altered by other plugins.

The module, created by the (function(){ })(); expressions, contains an object called plugin. The plugin object has two properties: info and trial. The plugin object is returned at the end of the module, which is what assigns the defined properties of plugin to jsPsych['plugin-name'].

# plugin.info
The plugin's info property is an object that contains all of the available parameters for the plugin. Each parameter name is a property, and the value is an object that includes a description of the parameter, the value's type (string, integer, etc.), and the default value. See some of the plugin files in the jsPsych plugins folder for examples.

jsPsych allows most plugin parameters to be dynamic, which means that the parameter value can be a function that will be evaluated right before the trial starts. However, if you want your plugin to have a parameter that is a function that shouldn't be evaluated before the trial starts, then you should make sure that the parameter type is 'FUNCTION'. This tells jsPsych not to evaluate the function as it normally does for dynamic parameters. See the canvas-* plugins for examples.

# plugin.trial
The plugin's trial property is a function that runs a single trial. There are two parameters that are passed into the trial method. The first, display_element, is the DOM element where jsPsych content is being rendered. This parameter will be an HTMLElement. Generally, you don't need to worry about this parameter being in the correct format, and can assume that it is an HMTLElement and use methods of that class. The second, trial, is an object containing all of the parameters specified in the corresponding TimelineNode. If you have specified all of your parameters in plugin.info, along with default values for each one, then the trial object will contain the default values for any parameters that were not specified in the trial's definition.

The only requirement for the trial method is that it calls jsPsych.finishTrial() when it is done. This is how jsPsych knows to advance to the next trial in the experiment (or end the experiment if it is the last trial). The plugin can do whatever it needs to do before that point.

Of course, there are other things that you will probably want the plugin to do inside the plugin.trial function, besides just end. Here are some examples:

Changing the content of the display
There are a few ways to change the content of the display. The display_element parameter of the trial method contains the DOM element for displaying content, so you can use various JavaScript methods for interaction with the display element. A common one is to change the innerHTML.

```javascript
var html_content = '<p>This is the first paragraph</p>';
html_content += '<p>This is the second paragraph</p>';

display_element.innerHTML = html_content;
jsPsych doesn't clear the display before or after each trial, so it is often appropriate to use innerHTML to clear the display at the end of a trial:


// clear the display
display_element.innerHTML = '';
Writing data
Plugins exist to collect data, so saving data is obviously a crucial thing to do. You can pass an object of data as the parameter to jsPsych.finishTrial():


var data = {
  correct: true,
  rt: 350
}

jsPsych.finishTrial(data);
```

The data recorded will be that correct is true and that rt is 350. Additional data for the trial will also be collected automatically by the jsPsych library.

The plugin template
An empty plugin template is included in the plugins/template folder.
