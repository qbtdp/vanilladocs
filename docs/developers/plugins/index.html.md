---
title: Plugins
layout: docs
categories: ["Developers","Plugins"]
---

## Extending Vanilla with plugins

Vanilla is a very customizable, flexible platform because there are many ways to hook into it without ever modifying its code.


### Custom Events & Handlers
Any class that extends the class "Pluggable" has the ability to call

```
$this->FireEvent('EventName');
```
and then plugins can attach to that event to perform an action. 

There is a PluginManager that detects any enabled plugins. When the `FireEvent` method is called, it pings the PluginManager class to see if there are any plugins that want to attach to the event name being fired.

Plugins attach to an event by creating a method named with the object name, event name, and the word "Handler" separated by underscores. Say the `DiscussionsController` fired an event named 'Kaboom'. Here is how you would use it:

```
class MyPlugin extends GDN_Plugin() {
   public function DiscussionsController_Kaboom_Handler($Sender, $Args) {
   	  // Do stuff at the 'Kaboom' event here.
   	  $Sender->Index(); // You can invoke DiscussionsController methods here.
   }
}
```
Notice that 2 parameters are passed. The first is the object invoking the event (usually `$this`). The second is *optional*, and is an array of event arguments being passed as additional info or options to be modified.

For example:

```
$this->EventArguments['DuckDodgers'] &= '24.5';
$this->FireEvent('Kaboom');
```

Now in our `Handler` method above, $Args would be an array of `'DuckDodgers' => '24.5'`. If we set `$Args['DuckDodgers'] = 0;` in our `Handler` method, it would also be changed back in the `DiscussionsController` because it was passed by reference.

To invoke a handler on ALL methods, use the prefix 'Base' instead of an object name. Example: `Base_Kaboom_Handler`. It is best to avoid unnecessary calls by using this *very* conservatively.

Custom events are added on a case-by-case basis as the need arises. If you feel you need a new event, request it on the community forum.


### Magic Events

Magic events were an elaborate system of hook possibilities that involved the method prefix 'x' and PHP's `__call()` method. Currently, there is only one undeprecated magic event in Vanilla: `Render_Before`. It invokes just before the page is rendered. Example use: `Base_Render_Before($Sender)`. **It is best to avoid when another event is usable.**

For a better alternative hook that reliably fires early on every request, try `Gdn_Dispatcher_AppStartup_Handler` instead. To universally include a CSS file, use `AssetModel_StyleCss_Handler`.


### Function Overrides

All functions in the framework are declared like so:

```
if (!function_exists('FunctionName')) {
   function FunctionName() {
   	  // Do something.
   }
}
```

Plugin authors may therefore override any core function simply by defining it in a plugin or theme hook file, which are included before the core functions.


### Magic Methods

Magic methods allow you to create new methods and add them to existing objects. They are created in much the same way that you plug into events. Imagine you wanted to add a method named `Kaboom` to the DiscussionsController:

````
class MyPlugin extends Gdn_Plugin {
   public function DiscussionsController_Kaboom_Create($Sender) {
        echo "Kaboom!";
   }
}
```

With this plugin enabled, going to the URL `/discussions/kaboom` would now output the text "Kaboom!". You can references other methods and properties on the extended object using the `$Sender` variable.

If you use a magic method to duplicate an existing method name, it will be overridden completely. And call to it will be directed to your plugin instead. The only exception is the `Index()` method.

Magic methods only work in classes that extend `Gdn_Pluggable`. For example, notice the `Gdn_Form` class does, but the `Gdn_Format` class does not. All models and controllers do.


### Controllers

Is your addon a bit more complex? Add a `controllers` folder and add your own controller classes. They will automatically be detected and be dispatched to like normal controllers. This functionality is new in Vanilla 2.3.

As a reminder, controllers should extend Gdn_Controller. [Read more about controllers](/developers/framework/controllers).


### Virtual Controller

A virtual controller let you simulate a single controller within a plugin. First, create a magic method using the namespace you want for your controller. Example: `PluginController_NewNamespace_Create`. This creates content under the URL `/plugin/newnamespace`. To put NewNamespace as the first level (instead of under "plugins") use the RootController. End the magic method with a re-dispatch: `$this->Dispatch($Sender, $Sender->RequestArgs);`.

Next, create methods named `Controller_X`, where X is the method namespace. Example: `Controller_Index($Sender)`.

Let's put it all together with a quick example. Let's say you wanted to create content at `/fancy/ping`. This would accomplish that:

```
<?php

class MyFancyPlugin extends Gdn_Plugin () {

   public function RootController_Fancy_Create($Sender) {
   	   // This is a good place to check permissions or include assets.
	   $this->Dispatch($Sender, $Sender->RequestArgs);
   }

   public function Controller_Ping($Sender) {
	   // This is what happens when you call /fancy/ping.
   }
}
```

This is obviously more resource-intense than a normal controller, so avoid this technique if normal controllers are available to you and/or you need more than one controller.