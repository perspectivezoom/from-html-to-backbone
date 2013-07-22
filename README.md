# Step by step from HTML to Backbone

This page is a step by step refactoring of a plain, non-javascripted HTML page into a basic Backbone app.

As the name suggests, it is inspired by kjbekkelund's excellent [Step by step from jQuery to Backbone](https://github.com/kjbekkelund/writings/blob/master/published/understanding-backbone.md), but with a much more junior audience in mind. The material for this article stems from a two hour guest lecture at [Hack Reactor](http://hackreactor.com/) (one of the many "Become a professional coder in X weeks" boot camps that have popped up lately) and as such is intended to be useful for those who:

1. Have a good grasp of HTML and CSS
* Just started using jQuery to manipulate a web page
* Kind of know what a Javascript framework is, but not really

If you have a strong programming background, and can understand [jQuery to Backbone's](https://github.com/kjbekkelund/writings/blob/master/published/understanding-backbone.md) opening setup and central thesis of separation of concerns, then the following material is likely too simple for you.

In this step by step, parts of Backbone are integrated one at a time and explained in detail. The goal is to eliminate the mysticism behind using Backbone, and for you to understand how everything works. Though not really framework itself (the most appropriate definition I've heard is a "framework framework"), understanding Backbone will give you a much firmer grasp of the more magical frameworks you might encounter.

## Part 1: From static to static

_in which a basic HTML page remains visibly unchanged, but at the end just so happens to have Backbone doing its thing behind the scenes._

### Step 0: The starting HTML

We'll be backbone-ifying a simple website named Game Tracker. It consists of a table with two columns: The name of the game, and the number of minutes played. Game Tracker starts out with no javascript, and is only 31 lines long:

```html
<!DOCTYPE html>
<html>
  <head>
    <link href="styles.css" rel="stylesheet" type="text/css" media="all">
  </head>
  <body>
    <div class='header'>Game Tracker</div>
    <table class='content'>
      <thead>
        <tr>
          <th>Name</th>
          <th>Minutes Played</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Super Mario World</td>
          <td>290</td>
        </tr>      
        <tr>
          <td>Donkey Kong Country</td>
          <td>140</td>
        </tr>
        <tr>
          <td>Mega Man X</td>
          <td>60</td>
        </tr>
      </tbody>
    </table>
  </body>
</html>
```

You can view the page [here](http://perspectivezoom.com/from-jquery-to-backbone/index00mockup.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index00mockup.html) 

### Step 1: Adding a template

Before using Backbone, we first need to consider how to render templates. Backbone is template-agnostic; you get to choose how you want Backbone to render the resultant HTML. 

In a real app, you'll likely want something more powerful, but for our purposes Underscore.js' [built-in templating system](http://underscorejs.org/#template) is sufficient and (more importantly) also relatively simple to understand.

In this step, we'll be using Underscore templates exclusively, holding off on actually adding in Backbone until Step 2. Later, in Step 3, we'll integrate the work we've done in this step into a Backbone View.

To get started, we add the Underscore and jQuery libraries to our repository and reference them.

```diff
  <head>
    <link href="styles.css" rel="stylesheet" type="text/css" media="all">
+   <script src='vendor/jquery.js'></script>
+   <script src='vendor/underscore.js'></script>
  </head>
  <body>
```

Next, as we'll be using our template to generate our table, we delete the rows from our HTML.

```diff
	<table class='content'>
	  <thead>
	    <tr>
	      <th>Name</th>
	      <th>Minutes Played</th>
	    </tr>
	  </thead>
	  <tbody>
-       <tr>
-         <td>Super Mario World</td>
-	      <td>290</td>
-       </tr>
-       <tr>
-         <td>Donkey Kong Country</td>
-         <td>140</td>
-       </tr>
-       <tr>
-         <td>Mega Man X</td>
-         <td>60</td>
-       </tr>
	  </tbody>
	</table>
```

The data that we just deleted is instead declared as an array of objects.

```diff
	<head>
    <link href="styles.css" rel="stylesheet" type="text/css" media="all">
    <script src='vendor/jquery.js'></script>
    <script src='vendor/underscore.js'></script>
+   <script>
+     var games = [
+       { name: 'Super Mario World', minutes: 290 },
+       { name: 'Donkey Kong Country', minutes: 140 },
+       { name: 'Mega Man X', minutes: 60 }
+     ];
+   </script>
  </head>
```

We create an Underscore template that, when given a game, will generate the HTML for the game's row in the table.

```diff
    <script>
      var games = [
        { name: 'Super Mario World', minutes: 290 },
        { name: 'Donkey Kong Country', minutes: 140 },
        { name: 'Mega Man X', minutes: 60 }
      ];
+
+     var gameListing = _.template("<tr><td><%= name %></td><td><%= minutes %></td></tr>");
    </script>
```

Finally (after jQuery's `(document).ready` tells us that the page has fully loaded) we take each game, generate a table row using our template, and insert it into the table.

```diff
      var games = [
        { name: 'Super Mario World', minutes: 290 },
        { name: 'Donkey Kong Country', minutes: 140 },
        { name: 'Mega Man X', minutes: 60 }
      ];
 
      var gameListing = _.template("<tr><td><%= name %></td><td><%= minutes %></td></tr>");
+
+     $(document).ready(function () {
+       _.each(games, function (gameData) {
+         $('tbody').append(gameListing(gameData));
+       });
+     });
```

Ok, this step is done. If you load it in a browser, you'll see nothing different, but we know that different stuff is happening in the background.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index01addTemplate.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index01addTemplate.html) 

### Step 2: Adding a Backbone Model

Time to add in Backbone. We'll start with a [Backbone Model](http://backbonejs.org/#Model), which as you might expect, will hold our game data in the Backbone App.

Add and reference Backbone:

```diff
    <script src='vendor/jquery.js'></script>
    <script src='vendor/underscore.js'></script>
+   <script src='vendor/backbone.js'></script>
    <script>
```

We declare our Game "class" by [extending](http://underscorejs.org/#extend) the standard Backbone.Model and then adding on any extra functionality. Right now, we have no extra functionality, so our Game model is just a renamed standard Backbone Model.

```diff
      var games = [
        { name: 'Super Mario World', minutes: 290 },
        { name: 'Donkey Kong Country', minutes: 140 },
        { name: 'Mega Man X', minutes: 60 }
      ];
 
+     var Game = Backbone.Model.extend({});
      var gameListing = _.template("<tr><td><%= get('name') %></td><td><%= get('minutes') %></td></tr>");
```

To use our model, instead of taking each game object and passing it to the template directly, we pass the object to the  Backbone Game "class", and then pass the newly "instantiated" game into the template. In a Backbone Model, you retrieve stored data with the [get](http://backbonejs.org/#Model-get) function, so we'll have to modify our template accordingly.

```diff
    <script>
      var Game = Backbone.Model.extend({});
-     var gameListing = _.template("<tr><td><%= name %></td><td><%= minutes %></td></tr>");
+     var gameListing = _.template("<tr><td><%= get('name') %></td><td><%= get('minutes') %></td></tr>");
 
      $(document).ready(function () {
        _.each(games, function (gameData) {
-         $('tbody').append(gameListing(gameData));
+         var game = new Game(gameData)
+         $('tbody').append(gameListing(game));
        });
      });
    </script>
```

Ok, we've added a Backbone Model. It seems a bit unnecessary, because it doesn't really do much at the moment. Truth be told, even at the end of this step-by-step, the model still won't do much, but hopefully, after Step 7, you'll see how it *could* be useful in a bigger app.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index02addModel.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index02addModel.html) 

### Step 3: Adding a Backbone View

Unlike Backbone Models, we're going to be doing quite a lot with a [Backbone View](http://backbonejs.org/#View). Let's first add in the "class definition" for our view, and then figure out what each line does.

```diff
      var Game = Backbone.Model.extend({});
-     var gameListing = _.template("<tr><td><%= get('name') %></td><td><%= get('minutes') %></td></tr>");
+     var GameView = Backbone.View.extend({
+       tagName: 'tr',
+       template: _.template("<td><%= get('name') %></td><td><%= get('minutes') %></td>"),
+       render: function () {
+         this.$el.html(this.template(this.model));
+         return this;
+       }
+     });
      
      $(document).ready(function () {
```

The first thing to notice is that the Underscore template moved from its own variable to the `template` property of the View. Strictly speaking, this wasn't necessary; our render function could have just as easily been `this.$el.html(gameListing(this.model));` instead of `this.$el.html(this.template(this.model));`, but Backbone Views provide us with the convention of storing the View's template in the template property. More importantly, it makes sense to store it there; the template doesn't really belong floating out in the open.

Which brings us to the render function itself. As mentioned previously, Backbone lets you choose how to render HTML. In the standard Backbone.View, the `render` function is completely blank, forcing you to define it yourself. The sole job of this.render is to populate the View's `el` property, where the View stores its HTML. For reasons of convenience and [convention](http://backbonejs.org/#View-render), we define render to return the view after it has finished populating its el. This will allow us to employ method chaining later on in this step.

By default, `el` is just an empty `<div></div>`. We want our View's `el` to be a HTML table row, so we use the tagName property to make its containing `el` a tr instead. Thus, when the View is newed up, before we call its render function, our View's `el` is just `<tr></tr>`. We also remove the `<tr></tr>` tags from the template. 

When render is called, we take our model, run it through our template, and stuff the resultant HTML into `el`'s `<tr></tr>`. Helpfully, Backbone gives us `this.$el`, a convenient shorthand for `$(this.el)`.

Ok, time to use the view:

```diff
      $(document).ready(function () {
        _.each(games, function (gameData) {
          var game = new Game(gameData)
-         $('tbody').append(gameListing(game));
+         var gameView = new GameView({ model: game });
+         $('tbody').append(gameView.render().el);
        });
```

Previously, we took our template, fed in our Backbone Game Model, and took that HTMl and appended it to the table. This time, we first new up a Backbone View and feed it our model. Instead of calling the template directly, we indirectly use it through the view's render function, which populates the view's `el`. Finally, taking advantage of the method chaining provided by `gameView.render`'s return value, we take the view's populated `el`, and, as before, append it to the table. 

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index03addView.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index03addView.html) 

Once again, nothing visibly different at the moment, but we'll take full advantage of Backbone Views in the second half. To give you a little food for thought: we've effectively assigned ownership of portions of HTML. Each table row is its own newed up Backbone View, and that view is responsible for the elements inside of it. That single concept, of sections of HTML being owned by a responsible object, is among the most powerful new insights that have emerged from Backbone and similar libraries.

### Step 4: Adding a Backbone Collection

Time to add in the last ingredient in Backbone's core compositional trinity: the [Collection](http://backbonejs.org/#Collection). Compared to the view, understanding and implementing a collection is much more straightforward. Collections can be described as a fancy array for Backbone Models. We add our "class definition" as follows:

```diff
      var Game = Backbone.Model.extend({});
+     var GameCollection = Backbone.Collection.extend({
+       model: Game
+     });
      var GameView = Backbone.View.extend({
        tagName: 'tr',
```

For now, the only modification that we're making from the standard Backbone collection is to declare the model type that this collection accepts. It will be useful momentarily, for when we add games to the collection.

Using a collection streamlines the way we use the previous two Backbone components:

```diff
      $(document).ready(function () {
-       _.each(games, function (gameData) {
-         var game = new Game(gameData);
-         var gameView = new GameView({ model: game });
-         $('tbody').append(gameView.render().el);
-       });
+       var gameCollection = new GameCollection();
+       gameCollection.add(games);
+       gameCollection.each(function (game) {
+         var gameView = new GameView({ model: game });
+         $('tbody').append(gameView.render().el);         
+       });
      });
```

We first new up an empty collection, then add in our games. Then, iterating over the collection, we make a new view from each game, render the view, and add the view's `el` to the table. We use `gameCollection.each(myIteratorFunction)` as a Backbone-provided shorthand for the more explicit Underscore#each invocation: `_.each(gameCollection.models, myIteratorFunction)`

Now, before we move on, I'd like to draw attention to two small pieces of Backbone magic that we've used.

The first piece of magic occurred when passing a model in as an option to the view: `new GameView({ model: game })`. If including a model upon newing, Backbone will automatically put the model into the view's `this.model`. This is not true of all keys in an object; `new GameView({ foo: "bar" })` will populate the `this.options.foo` propery, not `this.foo`. See the [Backbone docs](http://backbonejs.org/#View-constructor) to see which keywords receive special treatment.

The second piece of magic is adding models to the collection. You'll notice that when we add data to the collection with `gameCollection.add(games)`, the games variable is an array of regular javascript objects. As it turns out, according to the [docs](http://backbonejs.org/#Collection-model):
> if [a Collection's `model` property] is defined, you can pass raw attributes objects (and arrays) to add, create, and reset, and the attributes will be converted into a model of the proper type.

So that's nice. We don't have to create a model only to stuff it into a collection. As long as we tell the collection what to expect, Backbone takes care of that for us.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index04addCollection.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index04addCollection.html) 

### Step 5: Adding a second view

Though we've added in three of the core Backbone components, it isn't quite yet up to convention. There's still two steps to be done before we can consider call this page canonical. 

The first is to add in a second view. We have a view for each table row, but we don't yet have one for the table itself. Although it is not strictly necessary at the moment, we'll see in Step 9 just how useful having a "Collection View" (to borrow a term from the [Backbone-Marionette](https://github.com/marionettejs/backbone.marionette) framework) can be.

In preparation for this second view, we rename the existing GameView to be a little bit more descriptive:

```diff
      var GameCollection = Backbone.Collection.extend({
        model: Game,
      });
-     var GameView = Backbone.View.extend({
+     var GameTableRowView = Backbone.View.extend({
        tagName: 'tr',
        template: _.template("<td><%= get('name') %></td><td><%= get('minutes') %></td>"),
```

Next, we remove some table HTML, as it will instead exist in the Table View's template. Amusingly, what began as 22 lines within the HTML body is now reduced to 2.

```diff
  <body>
    <div class='header'>Game Tracker</div>
-   <table class='content'>
-     <thead>
-       <tr>
-         <th>Name</th>
-         <th>Minutes Played</th>
-       </tr>
-     </thead>
-     <tbody>
-     </tbody>
-   </table>
+   <div class='content'></div>
  </body>
```

Ok, time to define the Table View:

```diff
      var GameTableRowView = Backbone.View.extend({
        tagName: 'tr',
        template: _.template("<td><%= get('name') %></td><td><%= get('minutes') %></td>"),
        render: function () {
          this.$el.html(this.template(this.model));
          return this;
        }
      });
+     var GameTableView = Backbone.View.extend({
+       tagName: 'table',
+       template: _.template('<thead><tr><th>Name</th><th>Minutes Played</th></tr></thead><tbody></tbody>'),
+       render: function () {
+         this.$el.html(this.template());
+         this.collection.each(function (game) {
+           var gameView = new GameTableRowView({ model: game });
+           this.$('tbody').append(gameView.render().el);
+         }, this);
+         return this;
+       }
+     });
      
      $(document).ready(function () {
```

Structurally, GameTableView is identical to GameTableRowView, with both views having `tagName`, `template`, and `render` defined. Though GameTableView's `tagName` and `template` are fairly straightforward, `render` is slightly more complicated. `render` first populates the view's `el` with the table header and body via its static template. Then, it takes each Game model in its collection (which we have to pass in at runtime), makes a GameTableRowView for that Game, and then adds the GameTableRowView's rendered `el` to the end of GameTableView's table body.

When we added the rendered `el` for the GameTableRowView, we use `this.$('tbody')`. As you might expect, `this.$('tbody')` is a convenience for `this.$el.find('tbody')` (which is itself short for `$(this.el).find('tbody')`).

This time, when using `this.collection.each`, we pass in a context object at the end. If we hadn't passed in `this` as the context for `this.collection.each(myIteratorFunction, context)`, the `this` inside the iteratorFunction would not be the correct `this`, and thus the line `this.$('tbody').append(gameView.render().el)` would not behave as intended. (For those that are still confused, add a `console.log(this);` right above `this.$('tbody').append…` line, and see what happens in the [console](http://stackoverflow.com/questions/4743730/what-is-console-log-and-how-do-i-use-it) when you switch between `}, this);` and `});`)

Using our new GameTableView:

```diff
      $(document).ready(function () {
        var gameCollection = new GameCollection();
        gameCollection.add(games);
-       gameCollection.each(function (game) {
-         var gameView = new GameView({ model: game });
-         $('tbody').append(gameView.render().el);
-       });
+       var gameTable = new GameTableView({ collection: gameCollection });
+       $('.content').html(gameTable.render().el);
      });      
```
We remove code that is now covered by GameTableView's `render`. Instead, we simply new up a GameTableView with our collection, call the `render` function, and then stuff the rendered `el` into our page.

All our table building functionality is now localized into a place where it makes sense to be. To anthropomorphize, at the very top level, all we say is "I want a Game table here. I don't particularly care how you do it, just as long as it contains this collection of games". The GameTableView encapsulates how exactly that happens.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index05addCollectionView.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index05addCollectionView.html) 

### Step 6: Adding a Namespace

This is the last step of our [green-green](http://www.example.com), no added-functionality refactor. Time to pause for a second and get our ducks in a row before we move on. Let's talk a little about app organization.

For pedagogical purposes, I've kept (and will continue to keep) GameTracker in a single HTML page. A real app, however, will split the app into a logical file structure. Each Model, View, Collection, and Template would live in its own file, and then later assembled with a javascript loader or asset pipeline.

In addition, it is customary to not pollute the global namespace with many variables, but to instead enclose apps in a single containg object. Let's go ahead and do that:

```diff
    <script>
      var games = [
        { name: 'Super Mario World', minutes: 290 },
        { name: 'Donkey Kong Country', minutes: 140 },
        { name: 'Mega Man X', minutes: 60 }
      ];
 
+     var GameTracker = { Models: {}, Collections: {}, Views: {} };
+ 
+     // MODELS
-     var Game = Backbone.Model.extend({});
+     GameTracker.Models.Game = Backbone.Model.extend({});
+ 
+     // COLLECTIONS
-     var GameCollection = Backbone.Collection.extend({
-       model: Game
+     GameTracker.Collections.GameCollection = Backbone.Collection.extend({
+       model: GameTracker.Models.Game
      });
+ 
+     // VIEWS
-     var GameTableRowView = Backbone.View.extend({
+     GameTracker.Views.GameTableRow = Backbone.View.extend({
        tagName: 'tr',
        template: _.template("<td><%= get('name') %></td><td><%= get('minutes') %></td>"),
        render: function () {
          this.$el.html(this.template(this.model));
          return this;
        }
      });
-     var GameTableView = Backbone.View.extend({
+     GameTracker.Views.GameTable = Backbone.View.extend({
        tagName: 'table',
        template: _.template('<thead><tr><th>Name</th><th>Minutes Played</th></tr></thead><tbody></tbody>'),
        render: function () {
          this.$el.html(this.template());
          this.collection.each(function (game) {
-           var gameView = new GameTableRowView({ model: game });
+           var gameView = new GameTracker.Views.GameTableRow({ model: game });
            this.$('tbody').append(gameView.render().el);
          }, this);
          return this;
        }
      });

+     // START APP 
      $(document).ready(function () {
-       var gameCollection = new GameCollection();
+       var gameCollection = new GameTracker.Collections.GameCollection();
        gameCollection.add(games);
-       var gameTable = new GameTableView({ collection: gameCollection });
+       var gameTable = new GameTracker.Views.GameTable({ collection: gameCollection });
        $('.content').html(gameTable.render().el);
      });
    </script>
```

Now, the only variable we've added (before runtime) is the GameTracker object, which holds all of our Model, View, and Collection definitions. We've also imposed some organization, with each Backbone piece getting its own section.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index06addNamespace.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index06addNamespace.html) 

## Part 2: Adding functionality

_in which we do new things._

### Step 7: Adding a model helper

Our first user-visible change will be relatively simple and self contained. We'll use a model helper to display the time in hours and minutes, instead of just minutes. The change will occur in just two places. In the Game model, we define our helper method:

```diff
      // MODELS
-     GameTracker.Models.Game = Backbone.Model.extend({});
+     GameTracker.Models.Game = Backbone.Model.extend({
+       formattedMinutes: function () {
+         var minutes = this.get('minutes');
+         return parseInt(minutes / 60, 10) + ' hours ' + minutes % 60 + ' minutes';
+       }
+     });
```

Then, in our GameTableRow view, we call the helper instead of getting the minutes.

```diff
      GameTracker.Views.GameTableRow = Backbone.View.extend({
        tagName: 'tr',
-       template: _.template("<td><%= get('name') %></td><td><%= get('minutes') %></td>"),
+       template: _.template("<td><%= get('name') %></td><td><%= formattedMinutes() %></td>"),
        render: function () {
```

Now, when you load the page, the minutes played will be slightly more readable. We could certainly be a little more fancy with the helper, de-pluralize single hours or omit zero hours and so on, but I think you get the point.

Ironically, this particular step, with the fewest changed lines, is also the most controversial. The choice to define formattedMinutes in a model helper is only one among several equally valid options.

Consider that, right now, we only use formattedMinutes in one place, the GameTableRow. To prevent [YAGNI](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it), it probably makes more sense to localize the function to this particular view. (in which case, we would give the template the view instead of the model, and access the model name through view.model as `model.get('name')`). If it weren't for the fact that there otherwise would not be any added Backbone Model functionality, I would probably have localized formattedMinutes.

Our current implementation would make sense if we used formattedMinutes in two different views. At that point, there's the implication that formattedMinutes is the canonical way to display a Game's minutes, and that you should always use formattedMinutes whenever display a Game's time played.

Finally, there's the case where we would need to use formattedMinutes for other models. Perhaps GameTracker becomes a more generic TimeTracker and we need formattedMinutes for a Car Backbone model. At that point, we can globalize the helper in one of two and a half ways. We could create a common Backbone Model or a common Backbone View with formattedMinutes in it, and then have all models/views extend the common view. Alternately, we could place formattedMinutes in its own independent helper section, such as GameTracker.Helpers.formattedMinutes.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index07addModelHelper.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index07addModelHelper.html) 

### Step 8: Adding Backbone Events

Time to add in our first interaction to the app. What we'll do is make our games sortable. By clicking on the Name or Minutes Played table header, we want to sort our list either alphabetically or by the time each game's been played.

We first modify our GameTable template slightly, so that when we click each header, we'll have more formalized information to work with:

```diff
      GameTracker.Views.GameTable = Backbone.View.extend({
        tagName: 'table',
-       template: _.template('<thead><tr><th>Name</th><th>Minutes Played</th></tr></thead><tbody></tbody>'),
 +      template: _.template("<thead><tr><th data-sort='name'>Name</th><th data-sort='minutes'>Minutes Played</th></tr></thead><tbody></tbody>"),
        render: function () {
          this.$el.html(this.template());
          this.collection.each(function (game) {
```

Next up is to use a Backbone Event to kick off a sort when the header is clicked. Backbone Events are very similar to jQuery events, but are conveniently limited in scope to the view's `el` html; declaring a binding on a `th` element, for example, will not put a binding on any `th`s outside of the view.

To define our Backbone Event, we declare an events hash that will attach the event to the headers when the view is rendered:

```diff
      GameTracker.Views.GameTable = Backbone.View.extend({
        tagName: 'table',
        template: _.template("<thead><tr><th data-sort='name'>Name</th><th data-sort='minutes'>Minutes Played</th></tr></thead><tbody></tbody>"),
+       events: {
+         'click th': 'sort'
+       },
        render: function () {
          this.$el.html(this.template());
          this.collection.each(function (game) {
```

Now, when we click any `th` element in our view, we call `view.sort`. Time to define that sort function:

```diff
        render: function () {
          this.$el.html(this.template());
          this.collection.each(function (game) {
            var gameView = new GameTracker.Views.GameTableRow({ model: game });
            this.$('tbody').append(gameView.render().el);
          }, this);
          return this;
-       }
+       },
+       sort: function (event) {
+         var sortColumn = $(event.currentTarget).data('sort');
+         this.collection.comparator = function (game) { return game.get(sortColumn) };
+         this.collection.sort();
+         this.render();
+       }
      });
```
The sort we've defined uses Backbone's [Collection#comparator](http://backbonejs.org/#Collection-comparator). The comparator chosen is of the "sortBy" type:

>"sortBy" comparator functions take a model and return a numeric or string value by which the model should be ordered relative to others.

In our sort function, we first figure out which property to sort by from the click target's date-sort HTML property. Then, we set the view's collection's comparator to use the correct property in the model as the sort value. We call [sort](http://backbonejs.org/#Collection-sort) on the collection (a completely different function than the `sort` function we defined in our view) to have the collection sort the models according to the newly defined comparator rules. Finally, we regenerate the view's html to reflect the collection's new order by calling `render`. As the view's `el` has been already added to the page's html, the changes are reflected immediately.

That's it for this step. Loading the web page will now give you clickable headers that sort the game list. This is the first step that really utilizes the power of Backbone. 

Note that the only section of our app that we've modified in this step is the GameTable view. Thanks to the way we've compartmentalized our app into different sections, it's relatively easy to find and modify functionality without accidentally disturbing other sections of the app.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index08addEvents.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index08addEvents.html) 

### Step 9: Adding an Add Form

Compartmentalizing different sections of your app into views sounds all fine and good in theory, but in practice can be frustrating. When you introduce a bit of order into your app, you necessarily sacrifice some flexibility. Before, without compartmentalization, every part of the web page could talk to and manipulate everything else. Now it can't; A Backbone View should only manipulate html exclusively inside its `el`.

To live with this limitation, views need to be able to talk to each other. In this step, we'll create an Add Form to create a new game model, and then see how to relay that information to the table view.

Let's define and use our Add Form View:

```diff
        }
      });
 
+     GameTracker.Views.AddForm = Backbone.View.extend({
+       events: {
+         'click button': 'add'
+       },
+       render: function () {
+         this.$el.html("<div><label>Name</label><input class='name'></input></div><div<label>Minutes</label><input class='minutes'></input></div><br /><button>Add</button>");
+         return this;
+       },
+       add: function () {
+         var name = this.$('input.name').val();
+         var minutes = this.$('input.minutes').val();
+         this.collection.add(new GameTracker.Models.Game({ name: name, minutes: minutes }));
+       }
+     });
+ 
      // START APP
      $(document).ready(function () {
        var gameCollection = new GameTracker.Collections.GameCollection();
        gameCollection.add(games);
        var gameTable = new GameTracker.Views.GameTable({ collection: gameCollection });
        $('.content').html(gameTable.render().el);
+       var gameForm = new GameTracker.Views.AddForm({ collection: gameCollection });
+       $('.form').html(gameForm.render().el);
      });
      
```
As the add form will always look the same, instead of using an underscore template, our add form's render function simply adds in html directly. There is a single event that triggers the view's only other function when the user clicks on the sole button in the form. When called, the add function reads whatever was inputted in the form, turns that information into a new Game model, and adds the new model to the view's collection.

When we start our app, we give it the same collection that our gameTable view uses. *Because both gameForm and gameTable use the same collection, we can use that common collection as a bridge to communicate between the two views.*

Looking at the documentation, we see that whenever a model is added to a Backbone Collection, [an "add" event is fired](http://backbonejs.org/#Collection-add). We can bind to that event to re-render the collection. Let's add an [initialize function](http://backbonejs.org/#View-constructor) to our gameTable view to set up that binding whenever the view is created:

```diff
        events: {
          'click th': 'sort'
        },
+       initialize: function () {
+         this.collection.on('add', this.render, this);
+       },
        render: function () {
          this.$el.html(this.template());
          this.collection.each(function (game) {
```

Now, when a model gets added to the collection, our gameTable view will take the collection's "add" event that was fired and invoke its render function, re-building the inner html in its `el` to match the new state of the collection.

You'll notice that there is a `this` third argument to `this.collection.on`, which [defines the context](http://backbonejs.org/#Events-on) when the binding invokes `this.render`. If we hadn't, when `render` was invoked via the collection's "add" event, all the references to `this` in the `render` function would refer the the collection, not our view. (In our case, we would get an error like: "could not find 'template'", since our collection does not, understandably, have a template property) In general, the fact that events call functions with a different `this` context is one of the biggest causes of debugging headache for developing Javascripters.

We've finished implementing a simple add form. If you load up a page, you can add a game and it will show up in our table. In a real app, there would be much more work to be done in terms of validation and error handling (for example, handling  non-numbers in the minutes field), but we'll put that outside the scope of this Step-by-Step.

Before we move on, I'd like to refactor gameTable's sort function in three ways. 

The first is to remove the explicit render, and instead use the same collection-event-triggering-a-render pattern that we've just used for the add form. 

The second is to rename the function from `sort` to `sortCollection`. 

The third is to move some of the sort logic from the view to the collection itself, by adding a sortByProperty function. If you think about it, that's where it really belonged in the first place.

```diff
      GameTracker.Collections.GameCollection = Backbone.Collection.extend({
-       model: GameTracker.Models.Game
+       model: GameTracker.Models.Game,
+       sortByProperty: function (property) {
+         this.comparator = function (game) { return game.get(property) };
+         this.sort();
        }
      });
```
```diff
      GameTracker.Views.GameTable = Backbone.View.extend({
        tagName: 'table',
        template: _.template("<thead><tr><th data-sort='name'>Name</th><th data-sort='minutes'>Minutes Played</th></tr></thead><tbody></tbody>"),
        events: {
-         'click th': 'sort'
+         'click th': 'sortCollection'
        },
        initialize: function () {
          _.bindAll(this, 'render');
          this.collection.on('add', this.render, this);
+         this.collection.on('sort', this.render, this);
        },
        render: function () {
          this.$el.html(this.template());
          this.collection.each(function (game) {
            var gameView = new GameTracker.Views.GameTableRow({ model: game });
            this.$('tbody').append(gameView.render().el);
          }, this);
          return this;
        },
-       sort: function (event) {
+       sortCollection: function (event) {
          var sortColumn = $(event.currentTarget).data('sort');
-         this.collection.comparator = function (game) { return game.get(sortColumn) };
-         this.collection.sort();
-         this.render();
+         this.collection.sortByProperty(sortColumn);
        }
      });
```
What I want to emphasize with these changes is the [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns) here: By clicking a column header, you are not sorting a table, nor the view. Instead, you are telling the view's internal collection how to sort. Then, after the collection has re-sorted itself, it lets you know that it's done. The act of telling the collection to sort, the collection actually sorting, and the act of the collection telling you to re-render are separate and distinct steps, and belong in different parts of the app. This refactor will also be highly useful for use in our final step.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index09addForm.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index09addForm.html) 

### Step 10: Adding a Backbone Router and Vent

For our last step, we are going to use the [Backbone Router](http://backbonejs.org/#Router) to inspect our URL and sort our table on loading the page. 

In the previous step, we used a common Backbone Collection as a bridge to communicate between two views. What if we didn't have that shared resource? In this step, we'll use an app-wide event aggregator that serves as an alternate means of app communication. [For historical reasons](http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/), this event bulletin board is called a vent. Let's add it in now:

```diff
      var GameTracker = { Models: {}, Collections: {}, Views: {} };
+     GameTracker.vent = _.extend({}, Backbone.Events);
 
      // MODELS
      GameTracker.Models.Game = Backbone.Model.extend({
```
Yep, it's just one line of code. All we need is a named bare bones Backbone Events object to receive and pass on messages.

To use the vent, we'll tell our collection to listen for a `sortByProperty` event:

```diff
      GameTracker.Collections.GameCollection = Backbone.Collection.extend({
        model: GameTracker.Models.Game,
+       initialize: function () {
+         GameTracker.vent.on('sort', this.sortByProperty, this);
+       },
        sortByProperty: function (property) {
          this.comparator = function (game) { return game.get(property) };
          this.sort();
        }
      });
```
Once again, we're passing in `this` as the third argument so that we maintain context. We want the `this` in the line `this.comparator = function …` to refer to the collection, not the vent.

Now that our collection is listening to the vent, its possible to sort the collection from anywhere within, or even outside the app, as long as it has access to `GameTracker.vent`. For exampe, after loading the page, you can call `GameTracker.vent.trigger('sort', 'minutes');` in the console to invoke a sort.

Time to add in the Backbone Router:

```diff
      GameTracker.Views.AddForm = Backbone.View.extend({
        events: {
          'click button': 'add'
        },
        render: function () {
          this.$el.html("<div><label>Name</label><input class='name'></input></div><div><label>Minutes</label><input class='minutes'></input></div><button>Add</button>");
          return this;
        },
        add: function () {
          var name = this.$('input.name').val();
          var minutes = this.$('input.minutes').val();
          this.collection.add(new GameTracker.Models.Game({ name: name, minutes: minutes }));
        }
      });
 
+     // ROUTER
+     GameTracker.Router = Backbone.Router.extend({
+       routes: {
+         'sort/:sortColumn' : 'sort'
+       },
+       sort: function (sortColumn) {
+         GameTracker.vent.trigger('sort', sortColumn);
+       }
+     });
+ 
      // START APP
      $(document).ready(function () {
        var gameCollection = new GameTracker.Collections.GameCollection();
        gameCollection.add(games);
        var gameTable = new GameTracker.Views.GameTable({ collection: gameCollection });
        $('.content').html(gameTable.render().el);
        var gameForm = new GameTracker.Views.AddForm({ collection: gameCollection });
        $('.form').html(gameForm.render().el);
+
+       new GameTracker.Router();
+       Backbone.history.start();
      });
```      
In much the same way as we used an events hash in our views, we've defined a routes hash to be triggered when we load up our app with extra params in the URL. In this case, we want the router call the `sort` function when we've appended `sort` to the URL. We also expect the column we want to sort by after the sort keyword, indicated by the colon-ed `:sortColummn` in the route definition. We also need to new up and start the router when the page loads.

With all this in place, it is now possible to open the page pre-sorted. You can [click here](http://perspectivezoom.com/from-jquery-to-backbone/index10addRouter.html#sort/name) to see it working online, or if you've opened up the page locally, the URL will look something like `file:///path_to_where_you_saved_game_tracker/Game%20Tracker/index10addRouter.html`. Add params at the end so it looks like `file:///path_to_where_you_saved_game_tracker/Game%20Tracker/index10addRouter.html#sort/minutes`. It will load the page => startup the router => inspect the params => trigger the vent `sort` event => invoke sortByProperty => trigger the collection `sort` event => invoke the gameTable view's render with the newly sorted collection. Cool. It looks like we're done.

[The current version of the page](http://perspectivezoom.com/from-jquery-to-backbone/index10addRouter.html) - [code](https://github.com/perspectivezoom/from-jquery-to-backbone/blob/gh-pages/index10addRouter.html) 

Although in this step-by-step, we've used only two inter-component communication strategies, please note that are more ways than just vents and shared resources. For example, Backbone Views that hold nested html, such as GameTable and GameTableRow, have a natural connection to each other, and although we never needed to do so, it would be perfectly reasonable in the near future for GameTable to keep a reference to its GameTableRows if it needs to directly talk to them. Something along the lines of GameTable creating a `this.gameViews` array and changing:

```diff
-   var gameView = new GameTracker.Views.GameTableRow({ model: game });
+   this.gameViews.push(new GameTracker.Views.GameTableRow({ model: game }));
```

### Final Thoughts

So there you have it. Starting from a plain HTML table, we refactored it into a basic Backbone application, and then (hopefully) added on enough functionality to justify refactoring the whole damn thing in the first place.

Along the way, I hope you picked up on the general theme behind every decision in implementing this app, the notion of Separation of Concerns. Although still an extremely simple app, you can see that there is already substantial complexity going on behind the scenes, a complexity that will only increase superlinearly as more and more features are added. The only way to manage this complexity is to organize the app into distinct parts, limited in scope and responsibility. 3 months down the line, when I inevitably add a fifth way to sort the collection, I have to trust that the 17 active views on the page are monitoring the collection for the change and know how to re-render themselves when that `sort` event is fired. Elsewise adding each additional feature will become more painful than the last, and your app will develop feature paralysis; you become afraid that changing one thing will break some seemingly unrelated function halfway across the page.

If it seems like this makes programming harder, well, that's because it does. To edit the table on a different section of the app, no longer can you just call `jQuery('.table-name .row-id').remove()` like a rampaging elephant clobbering everything in it path. There are now proper channels that need to be defined and utilized, an app bureaucracy, so to speak. All requests to alter the table need to go through the table's tableManager view. Unlike human bureaucracies, though, apps generally work exactly how you tell them to; once set up to do so, tableManager will always "get the memo", and will always follow up with the correct collectionManagers and tableRowUnderlings to let them know that things have changed. It's hard to set up and organize, but if done correctly, you'll find yourself with an easily extensible, [unit spec-able](http://en.wikipedia.org/wiki/Unit_testing) (a whole 'nother topic in and of itself) application that may actually be fun to work on.