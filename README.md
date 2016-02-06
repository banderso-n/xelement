# Motherboard

Motherboard is a small (less than 3k gzipped), extensible foundation for client-side JavaScript applications.

## Origins

### Address Complex UI Behaviors

Angular, React, and similar frameworks are primarily concerned with updating views as data changes. A common problem encountered when building web apps is communicating between components, e.g. an ajax-form in a modal that should cancel its current request if the modal is closed. Motherboard makes it easy to implement an architecture to handle cross-module communication. 

### A Reaction Against Monolithic, Wheel-Reinventing Frameworks

Motherboard is meant to be a foundation to build on. It doesn't force a specific router, templating engine, etc. on you. You can use whatever components are appropriate for your project. If one stops being good, you can easily swap it out for a different (better) one. This also helps keep page-weight down. This [TodoMVC app using Motherboard](http://bpander.github.io/motherboard-todos/) is only **7.3 KB** of JavaScript (gzipped and minified). By comparison, the AngularJS core gzipped and minified (the framework alone) is 45 KB.

## Usage

### Custom Elements

Motherboard uses the [Custom Element API](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements) to directly tie an element to the UI component associated with it.

**HTML**
```html
<m-carousel></m-carousel>
```

**JS**
```js
var MCarousel = M.element('m-carousel', function (proto, base) {

    // All the lifecycle callbacks are exposed
    proto.createdCallback = function () {
        base.createdCallback.call(this);
        ...
    };

});
```

### Custom Attributes

Motherboard includes a custom attribute API for easy configuration.

**JS**
```js
var MCarousel = M.element('m-carousel', function (proto, base) {

    proto.customAttributes.push(

        M.attribute('autoplay', { type: Boolean }),

        M.attribute('delay', {
            type: Number,
            default: 3000,
            changedCallback: function (oldVal, newVal) {
                // I'll fire whenever the 'delay' attribute/property is changed
            }
        });

    );
});
```

**HTML**
```html
<m-carousel id="one"></m-carousel>
<m-carousel id="two" autoplay delay="5000"></m-carousel>
```

**JS**
```js
var one = document.getElementById('one');
one.autoplay; // false
one.delay; // 3000

var two = document.getElementById('two');
two.autoplay; // true
two.delay; // 5000
typeof one.delay; // "number"
```


### Responsive Attributes

Attributes can be flagged as `responsive` if their values should change depending on the current media.

**JS**
```js
var MCarousel = M.element('m-carousel', function (proto, base) {

    proto.customAttributes.push(

        M.attribute('slides-visible', {
            type: Number,
            responsive: true,
            default: '(min-width: 768px) 3, 1', // Works just like the img.sizes attribute
            mediaChangedCallback: function (oldVal, newVal) {
                // I'll fire whenever a breakpoint causes the value to change
            }
        })

    );

});

var carousel = new MCarousel();
carousel.slidesVisible; // "(min-width: 768px) 3, 1"
carousel.currentSlidesVisible; // `3` if the viewport is 768px or wider, otherwise `1`
```

### Normalized, Intuitive, and Performant DOM Querying

#### Querying random elements

**HTML**
```html
<m-carousel>
    <button data-tag="m-carousel.nextButton">Next</button>
    <ul>
        <li data-tag="m-carousel.slides"></li>
        <li data-tag="m-carousel.slides"></li>
        <li data-tag="m-carousel.slides"></li>
    </ul>
</m-carousel>
```
**Note:** The {m-tag.propertyName} pattern isn't forced, it's just a useful convention.

**JS**
```js
var MCarousel = M.element('m-carousel', function (proto, base) {

    proto.createdCallback = function () {
        base.createdCallback.call(this);

        // Gets the first child element with a matching `data-tag` attribute
        this.nextButton = this.findWithTag('m-carousel.nextButton');

        // Gets all child elements with a matching `data-tag` attribute
        this.slides = this.findAllWithTag('m-carousel.slides');

        // Element collections are returned as arrays to make them easier to work with
        this.slides.forEach(function (slide) {
            ...
        });

    };
});
```

#### Querying for a specific type

**HTML**
```html
<m-slideshow>
    <m-carousel></m-carousel>
    <m-carousel data-tag="m-slideshow.carousel"></m-carousel>
</m-slideshow>
```

**JS**
```js
var MSlideshow = M.element('m-slideshow', function (proto, base) {
    
    proto.createdCallback = function () {
        base.createdCallback.call(this);

        // Gets the first child element of the specified type
        this.carousel = this.getComponent(MCarousel);

        // Gets the first child element of the specified type and tag
        this.carousel = this.getComponent(MCarousel, 'm-slideshow.carousel');

        // Gets all matching child elements of the specified type
        this.carousels = this.getComponents(MCarousel);

        // Gets all matching child elements of the specified type and tag
        this.carousels = this.getComponents(MCarousel, 'm-slideshow.carousel');

        // Element collections are returned as arrays to make them easier to work with
        this.carousels.forEach(function (carousel) {
            ...
        });
    };
});
```

### Event Handling

**JS**
```js
// Basic example
var carousel = new MCarousel();
carousel.listen(carousel, 'click', function () {
    console.log('click detected');
});
carousel.enable();

// Slightly more complex example
var MCarousel = M.element('m-carousel', function (proto, base) {
    
    proto.createdCallback = function () {
        base.createdCallback.call(this);
        ...
        this.listen(this.nextButton, 'click', function (e) {
            this.advance(1); // `this` keyword is bound to the specific element instance
        });
        this.enable(); // Enables all listeners
        this.disable(); // Disables all listeners

        // `.listen` can be used with a single element or an array of elements
        this.dotsListener = this.listen(this.dots, 'click', function (e) {
            var index = this.dots.indexOf(e.target);
            this.goTo(index);
        });
        this.dotsListener.enable(); // Enable a specific listener
        this.dotsListener.disable(); // Disable a specific listener
    };
});
```

### Triggering Events

**JS**
```js
// Basic example
var carousel = new MCarousel();
var listener = carousel.listen(carousel, 'foo', e => console.log(e.detail));
carousel.enable();
carousel.trigger('foo', { foo: 'bar' }); // logs `{ foo: 'bar' }`

// Anoter basic example
document.body.addEventListener('foo', function (e) {
    console.log(e.target);
});
document.body.appendChild(carousel);
carousel.trigger('foo'); // logs the <m-carousel> element to the console

// Slightly more complex example
var MCarousel = M.element('m-carousel', function (proto, base) {

    proto.EVENT = {
        SLIDE_CHANGE: 'm-carousel.slidechange'
    };

    proto.advance = function (howMany) {
        ...
        this.trigger(proto.EVENT.SLIDE_CHANGE, { previous: previous, current: current });
    };
});
```

## Advanced Usage

### Upgrading Existing Elements

#### HTML
```html
<form is="m-ajax-form"></form>
```

#### JS
```js
var MAjaxForm = M.extend('form', 'm-ajax-form', function (proto, base) {
    ...
});
```

### Extending Existing "M" Elements

#### JS
```js
var MVideoModal = M.extend(MModal, 'm-video-modal', function (proto, base) {
    ...
});
```

### Cross-Module Communication

Previously, I gave an example of cross-module communication that'd be difficult to pull off with the current landscape of frameworks: an ajax-form in a modal that should cancel its current request if the modal is closed. This is some pseudocode to explain roughly how it'd be accomplished with Motherboard.

**HTML**
```html
<m-ajax-modal>
    <form is="m-ajax-form">
    </form>
</m-ajax-modal>
```

**JS**
```js
var MAjaxModal = M.extend(MModal, 'm-ajax-modal', function (proto, base) {

    proto.createdCallback = function () {
        this.ajaxForm = this.getComponent(MAjaxForm);
    };

    proto.close = function () {
        base.close.call(this); // `super` call
        this.ajaxForm.cancel();
    };
});
```
