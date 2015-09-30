# Frontend Best Practices :: BOF

*Monday 5:15 - 6:00pm*


<style>
.wrong { background:#d28484; color:white; padding: 5px; }
.right { background:#80b276; color:white; padding: 5px;  }
</style>


Discussion of best practices with code and visual regression when reviewing PRs and processes on multi-team builds. Topics cover common issues with frontend Drupal theming and their correct alternatives based on scale and performance.


<!-- https://confluence.acquia.com/download/attachments/24290053/Rotary-Frontend-Workshop.pdf?version=1&modificationDate=1438903802000&api=v2 -->

**Primary areas of focus**
* Sass / CSS
* JavaScript
* Images
* Preprocessing functions
* Template files

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Sass / CSS

#### Excessive nesting of selectors in Sass

<!--wrong: http://screencast.com/t/rO2ca96bukQ
right: http://screencast.com/t/eNv0Ti4e-->

<span class="wrong">Wrong</span>

```scss
.region-header nav.block-system-menublock,
footer,
div.sidebar {
  background: $darkblueish;
  ul.menu {
    margin: 0;
    li {
      display: inline-block;
    }
  }
  li.menu-item a {
    color: #FFF;
    padding: 12px 10px;
    display: inline-block;
    text-transform: uppercase;
    text-decoration: none;
    &:hover {
      background: $greyish;
      color: $darkblueish;
    }
  }
}
```

<span class="right">Right</span>


```scss
.region-header nav.block-system-menublock {
  background: $darkblueish;
  ul.menu {
    margin: 0;
    li {
      display: inline-block;
    }
  }
  li.menu-item a {
    @extend .common-link;
  }
}

footer li  a { @extend .common-link; }
div.sidebar li  a { @extend .common-link; }
```




#### Abusing overly-specific selectors

<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>
```scss
body.fancy > .nope > div.why > div.dude > span strong {
  display: $yellow;
}
```
<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

```
span.yellow strong {
  display: $yellow;
}
```

#### Failing to write Sass code as components
<!--wrong: http://screencast.com/t/sXjTMPGaYWkQ-->
<!--right: http://screencast.com/t/aCRH1jgHC-->

<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>
```scss
div.views-fancy .markup-element {
  div.header {
    float: left;
    width: 20%;
    color: $orange;
    background: $blue;
  }
  div.body-section {
    float: left;
    width: 80%;
    background: $fancy-gradient;
  }
}
```

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

```scss
div.views-fancy .markup-element .col-4 { // applying col-4 to .header
  color: $orange;
  background: $blue;
}

div.views-fancy .markup-element .col-8 { // applying col-8 to .body-section
    background: $fancy-gradient;
}
```


#### Avoiding bloated code caused by @extend and similiar

<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>
```scss
.foo.bar + .sibling {
  color: $green;
  font-weight: bold;
}
.foo .descendant {
  @extend .sibling;
}

// Rendered CSS
.foo.bar + .sibling,
.foo .foo.bar + .descendant {
  color: green;
  font-weight: bold;
}
```

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

```scss
.green {
  color: $green;
}
.bold-font {
  font-weight: bold;
}
.foo .descendant {
  @extend .green;
  @extend .bold-font;
}
```

####   Other Topics
* Leaning on complex Sass instead of fixing the markup
* Not using variables or placeholders for common blocks of code
* Understanding the proper scenarios to use extends, mixins, or similiar



## JavaScript

####  Using the data-* tag to interact with DOM
<!--example: http://screencast.com/t/2cVqPpmDXZ-->

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>
```
// HTML
<article id="dude" data-columns="5" data-hamburger="yummy" data-fudge="dark">
  .....
</article>

// JS
var dude = document.getElementById('dude');
dude.dataset.columns
dude.dataset.hamburger
dude.dataset.fudge
```


####  Separation of functionality in various Drupal behaviors

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

```JS
Drupal.behaviors.Widget_A = {
attach: function (context, settings) {
  if (context === document) { // only fires on document load
    console.log('loaded Widget A');
  }
}};


Drupal.behaviors.Widget_B = {
attach: function (context, settings) {
  if (context === document) { // only fires on document load
    console.log('loaded Widget B');
  }
}};
  ```



#### Using timeout functions to prevent event bubbling

<!--wrong: http://screencast.com/t/q4FfgAcJf5S-->
<!--right: http://screencast.com/t/lPKlPY9F-->


<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>


![](http://content.screencast.com/users/BedimStudios/folders/Jing/media/0ce40cca-7514-46cd-910d-8cb2a312a482/00000765.png)


<span style="background:#80b276; color:white; padding: 5px; ">Right</span>
```JS
var timerEl;
$(window).bind("resize", function () {
  if (timerEl) {
    clearTimeout(timerEl);
  }
  timerEl = setTimeout(function () {

    // put code here ////////
    console.log('the page has been resized');

  }, 500); // this only fires every .5 seconds
});
```



####  Using .length to validate variables

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>
```JS
var testVar = $("body.section div.something");
if (testVar.length) { // checks to if exists
  console.log(testVar);
}
```


####  Declaring variables for performance

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>
```JS
// could be improved by storing as a variable
var something = $('.testelement .something');
something.click(function(e) {
  console.log('this is clicked');
});
```


####   Use vanilla JS instead of jQuery for  performance

![](http://content.screencast.com/users/BedimStudios/folders/Jing/media/8f30130d-f485-4fa1-a61f-15e13ce4dfa0/00000780.png)


#### Use ID selectors whenever possible
![](http://content.screencast.com/users/BedimStudios/folders/Jing/media/5202e68e-1591-443e-91f3-e13c0012980c/00000781.png)


####   Combine find() with ID selectors

```JS
// one of the faster combinations
var outside = $("#outside-container")
outside.find('.outside-container .certain-element');
```

####   Other Topics
* Refrain from using .animate() unless required
* Make use of async and defer
* Don't use inline CSS set by JS when classes will work
* Avoid removing, adding, or manipulating DOM elements



## Images

#### Use other options than PNG for non-transparent images
<!--compare: http://screencast.com/t/V60nORSrFZ3-->

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

![](http://content.screencast.com/users/BedimStudios/folders/Jing/media/f9642337-343d-4876-85ac-c948f9a64a0e/00000782.png)


#### Compressing images thru Grunt or Gulp

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

![](http://content.screencast.com/users/BedimStudios/folders/Jing/media/14fc663a-dfbb-409e-b0d0-dbbfa0937663/00000771.png)


####   Other Topics
* Using SVG images to replace common image assets
* Make use of sprites as a cachable element
* Lazy-loading images for parallax styles
* Make use of the ```<picture>``` HTML5 tag


## Preprocessing functions

#### Use Hook theme suggestions *** TODO

#### Utilizing preprocess function to namespace custom classes

<span style="background:#80b276; color:white; padding: 5px; ">Right</span>

```php
function example_preprocess_panels_pane(&$vars) {
  // add class per pane type
  switch ($vars['pane']->subtype) {
    case 'something-panel_pane_plan':
      $vars['classes_array'][] = 'my-grid-1';
      break;
    case 'another-panel_pane_plan':
      $vars['classes_array'][] = 'my-grid-2';
      break;
    case 'thisone-panel_pane_hub':
    case 'hellotest-panel_pane_hub':
      $vars['classes_array'][] = 'my-grid-3';
      break;
  }
}
```

####  Do not use the template.php to insert markup

<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>

```php
function thunder_preprocess_block(&$variables) {
  $variables['dumb_markup'] = '<div class="stuff">';
  $variables['dumb_markup'] .= 'This is a bad idea';
  $variables['dumb_markup'] .= '</div>';
}
```



#### Do not use cacheable functions designed for modules

<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>

```php
function thunder_preprocess_page(&$variables) {
  // no, no, no
  $node = node_load($variables['nid']);
  $block = module_invoke('module_name', 'block_view', 'block_delta');
  $entity = entity_load('field_collection_item', array($item_id))
  $sql_object = db_query("SELECT * FROM {blocks} WHERE module = '%s' AND delta = '%s'", $module, $delta);
}
```

#### Avoid commonly called preprocess function

<span style="background:#d28484; color:white; padding: 5px; ">Wrong</span>


```php
function thunder_preprocess_field(&$variables, $hook) {
  if ($variables['element']['#field_name'] == 'lots_o_fields') {
    $variables['items'][0]['#markup'] = 'have fun finding this later';
  }
}
```

####   Other Topics

* oxo
* oxo




## Template files

#### multiple page.tpl.ph files

#### excessive markup in TPL files

#### excessive PHP logic  in TPL files



