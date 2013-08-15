# The classList API

I have to be honest with you: I feel like a fraud writing about JavaScript for
HTML5 Doctor. I would feel like a fraud writing about JavaScript for a click-
driven ad-splattered content farm, never mind HTML5 Doctor.

The thing is though, I’m writing about the `classList` API, and it’s super easy.
If your JavaScript-fu isn’t great and you’re wary of HTML5 APIs, this one is at
the perfect temperature for toe-dipping, and it’ll leave you pleasantly
surprised at just how easy it is.

The `classList` API is a “does exactly what it says on the tin” API. It gets a
list of the classes on an HTML element and uses JavaScript to manipulate it.

Before this nugget of HTML5 came along, working with classes was a royal pain,
but what was once a twisty overgrown cowpath with wolves lurking in thickets is
now a bright sunlit path fit for rollerskating.

## Getting the `classList`

Getting the `classList` is a simple matter of `element.classList`, like so:

    <p id="bad-joke" class="oh my giddy aunt">Two dogs are in the park. One says, "Nice day, isn't it?" The other says, "Holy crap, a talking dog!"</p>

    <script>
        var joke = document.getElementById('bad-joke');
        console.log(joke.classList);
    </script>

That will log something like ["oh", "my", "giddy", "aunt"]. The output style
varies between browsers, but it will be an object containing a list of the
classes on the element.

More accurately, it’s an object that stringifies to the value of the class
attribute on the element in question. [Try it yourself with the console
open][1].

    {
        0: "oh",
        1: "my",
        2: "giddy",
        3: "aunt"
    }

There is no specification for `classList` per se ([it gets a sentence in the
DOM4 spec][2]), but there is for `DOMTokenList`, which is `classList`‘s type, so
look at the [`DOMTokenList` specification][3] to find out how to work with lists
of classes.

## The Type of `classList`

If you’re looking to confirm the type of `classList`, you’ll need to do
`element.classList instanceof DOMTokenList`, which returns `true`. `typeof
element.classList` returns “object”, which, given that everything in JavaScript
is an object, isn’t particularly informative.

## Doing Things with our List of Classes

The `DOMTokenList` spec provides a number of methods that can be used on the
`classList`:

* `add()` for adding a class to the list
* `remove()` for removing a class from the list
* `contains()` to check if a class is in the list
* `toggle()` for toggling a class in and out of the list (with a twist)
* `item()` to return the class at a specified position in the list
* `toString()` for turning the list into a string
* `length` to return the number of classes in the list
*  `value` to add custom properties and methods to the `classList` object

## `classList.add()`

It couldn’t be easier to add a class to an element. Just supply the class you
want to add as an argument to the `add()` method.

    joke.classList.add('beryl');

If you [view the demo][4] and look at the opening `<p>` tag in your favourite
inspector before and after pressing the button, you’ll see it change from `<p id
="bad-joke" class="oh my giddy aunt">` to `<p id="bad-joke" class="oh my giddy
aunt beryl">`.

Adding a class to the `<p>` is all done with that one line. No need to inject
inline styles or anything messy like that.

The corresponding `classList` object is:

    {
        0: "oh",
        1: "my",
        2: "giddy",
        3: "aunt",
        4: "beryl"
    }

## `classList.remove()`

Removing a class is just as easy. `joke.classList.remove('beryl')` brings us
back to where we started.

[View demo.][5]

## Adding and Removing Multiple Classes

The `DOMTokenList` spec refers to “tokens”, plural when describing how the
`add()` and `remove()` methods should be run.

> The add(tokens…) method […] If one of *tokens* is the empty string […] If one 
of *tokens* contains any ASCII whitespace […] For each *token* in *tokens* — 
[The DOM Spec][6]

<s>No browser so far has implemented a native method of adding/removing more 
than one class at a time, but it’s trivial to extend the DOMTokenList object 
prototype with a hand rolled function:</s>

It’s possible to add and remove multiple classes in the Blink powered browsers
with `element.classList.add('oh','my')`, and it’s trivial to extend the
`DOMTokenList` object prototype with a hand rolled function until support is
more widespread:

Thanks to David and Kalley for pointing that out in the comments.

    DOMTokenList.prototype.addmany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;
    
        for(i; i<ii; i++) {
            this.add(classes[i]);
        }
    }

Same goes for remove:

    DOMTokenList.prototype.removemany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;
    
        for(i; i<ii; i++) {
            this.remove(classes[i]);
        }
    }

[Here’s a demo where you can add and remove multiple classes at once.][7]

A [bug was filed][8] in 2011 (against a document that no longer exists)
suggesting allowing a space separated list or an array in `classList.add()` and
`classList.remove()`.

The use of the plural “tokens” could be interpreted as allowing an array, but it
doesn’t work. `InvalidCharacterError` is still specified as the exception to
throw if there are any spaces in “tokens”, so that’s definitely not going to
work.

If you want to replace an entire `classList` with a completely different set of
classes, you could use these two functions.

## `classList.contains()` 

This method returns a Boolean `true` or `false` when checking for the presence
of a class in the list.

In our example:

    joke.classList.contains('aunt') === true;
    joke.classList.contains('uncle') === false;

`contains()` is useful for checking if a class is in a list before performing an 
action that depends on its presence (or otherwise):

    if(joke.classList.contains('beryl') {
        joke.classList.remove('beryl');
    }

We can use `contains()` to make our previous functions for adding and removing
multiple classes better by ensuring they do not try to add duplicate classes on
the element or remove classes that don’t exist on it:

    DOMTokenList.prototype.addmany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;

        for(i; i<ii; i++) {
            if(!this.contains(classes[i])) {
                this.add(classes[i]);
            }
        }
    }

and:

    DOMTokenList.prototype.removemany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;

        for(i; i<ii; i++) {
            if(this.contains(classes[i])) {
                this.remove(classes[i]);
            }
        }
    }

## `classList.toggle()` 

For most use cases, `classList.toggle()` is very straightforward too. Typically,
an action by the program or the user will trigger a function that adds or
removes a class depending on whether it’s already in the list.

A simple show/hide is a good example of this:

    button.addEventListener('click', function() { element.classList.toggle('is-visible'); }, false);

[View a demo of a two-level menu][9] that takes up a little less room in smaller 
viewports by using `classList.toggle()` to enhance sub-menus.

## `force`

There’s a little twist to `toggle()`, though. It can take an optional second
parameter, `force`.

If `force` is set to `true`, the class will be added but not removed. If it’s
`false`, the opposite will happen — the class will be removed but not added.

Now that sounds a lot like `add()` and `remove()`, but there is a crucial
difference: `toggle()` with `force` returns `true` when a class is added, and
`false` when it’s removed — `add()` and `remove()` return `undefined`.

At the time of writing, only the Blink powered browsers honour the `force`
parameter, so view [the demo with true][10] and [the demo with false][11] in
Opera 15, Opera for Android, or the latest Chrome to see it working.

Browsers that don’t support `force` completely ignore it and merrily toggle the
class name on and off without throwing exceptions.

When it’s more widely supported, we can use a slightly leaner syntax for
ensuring the presence or absence of a class in the list. Instead of:

    var foo = function() {
        joke.classList.add('beryl');
    }

    foo();

    if(joke.classList.contains('beryl')) {
        // do stuff
    }

we could write:

    var foo = function() {
        return joke.classList.toggle('beryl', true);
    }

    if(foo()) {
        // do stuff
    }

## `classList.item()`

The `item()` method can be found elsewhere in JavaScript, usually on NodeList
objects. It returns the value of the item at `index` counting from zero.

In our example:

    <div id="bad-joke" class="oh my giddy aunt">
        <p>"Knock knock."</p>
        <p>"Come on in, the door's open."</p>
    </div>

    <script>
        var joke = document.getElementById('bad-joke'),

            whats_item_0 = function() {
                return joke.classList.item(0);
            },

            whats_item_2 = function() {
                return joke.classList.item(2);
            };

        console.log(whats_item_0()); // logs "oh"
        console.log(whats_item_2()); // logs "giddy"
    </script>

[View the demo][12].

`classList.item()` cannot be used for assignment, so 
`joke.classList.item(3) = 'uncle'` will throw an error.

If you were hoping `classList.item()` could assign values to items at specific
positions in the list, I’m afraid you’ll be disappointed. There’s no
`DOMTokenList` method that gives that level of control over the list.

## `classList.toString()`

If you ever need the `classList` turned into a string use this method.
`toString()` is another built in JavaScript method that isn’t specific to
`DOMTokenList`.

All the W3C spec says on the subject is that "DOMTokenList objects must
stringify to the underlying string." WHATWG leaves it at "The stringifier must
return the result of the set serializer for the associated list of tokens."

The only difference between the two is the amount of plain English used.

It takes no parameters when used with `classList` and returns all the classes on
the element as a space separated string:

    joke.classList.toString();
    // returns "oh my giddy aunt";

## `classList.length`

`length` is also a built-in JavaScript property. It returns the number of 
characters in a string, the number of items in an array, the number of arguments 
expected in a function, or (in our case) the number of classes in `classList`. 
In our example using “oh my giddy aunt”, it’s `4`.

    joke.classList.length
    // returns 4

Simple and obvious, hopefully.

## `classList.value`

On their own, the `item()`, `toString()`, and `length` methods aren’t especially 
useful. They can, however, be used in conjunction with each other and the rest 
of the methods of `DOMTokenList` to solve some of the problems I mentioned 
earlier.

Remember that `classList` is a regular JavaScript object, so we can add
properties to it the same as any other object:

    classList.use_in_production = false;
    classList.id = 5;
    classList.roof = 'thatch';

And of course we can add methods. Here’s a method that replaces the entire list
with a new one using `length`, `toString()`, `add()`, and `remove()`:

    element.classList.replace = function(classes) {
        var i = 0,
            ii = this.length,
            old_string = this.toString(),
            old_array = old_string.split(' '),
            new_array = classes.split(' '),
            j = 0,
            jj = new_array.length;

        // remove all the existing classes
        for(i; i<ii; i++) {
            this.remove(old_array[i]);
        }

        // add the new ones
        for(j; j<jj; j++) {
            this.add(new_array[j]);
        }
    };

And here’s a method that inserts a class in any position in the list you want.
It uses `contains()`, `item()`, `remove()`, `toString()`, and the `replace()`
method we just made:

    element.classList.insert = function(insert,position) {
        // check if the class is already in classList
        if(this.contains(insert)) {
            if(this.item(position) === insert) {
                // if it is already at the right position there's no need to continue
                return;
            } else {
                // remove it, we don't want it here
                this.remove(insert);
            }
        }

        var classes = this.toString(),
            classes_array = classes.split(' ');

            classes_array.splice(position, 0, insert);

            new_list = classes_array.join(' ');

            // use the custom replace method to replace the current classList
            this.replace(new_list);
    };

Being realistic, methods like these would probably be better used to extend
`DOMTokenList` like we did with `addmany()` and `removemany()`, but if you ever
need something specific to a particular `classList` you’re working on, you can
use `value`.

## Browser Support and Polyfills

The `classList` API [works in pretty much every up-to-date browser version][13].
Basic support at least has been in since Firefox 3.6, Opera 11.50, Chrome 8, and
Safari 5.1.

The big holes are IE9 and earlier and the [still widely used Android 2.3][14]
and earlier.

[There are at least two polyfills][15] available that can fill these holes.

[Devon Govett’s][16] was written quickly to add support to IE9 only, and it’s 
worth reading the comments for some good cross-browser JavaScript info.

[Eli Grey’s polyfill][17] has wider browser support. It works in IE8, and it 
provides basic `classList.add()`, `classList.remove()`, and `classList.toggle()` 
support to at least as far back as Android 2.1.

There’s also a [polyfill for adding `force` support][18] by *Егор Халимоненко*
to browsers that support `classList` but not `force` on `toggle()`.

If you only need a simple feature test you can use:

    if('classList' in document.createElement('a')) {
        // use classList API
    }

## Summary

You can think of the `classList` API on two levels. For most uses, it’s a simple
way to add, remove, toggle, and check for the presence of single classes in an
HTML element. I use it in every project I work on now and very rarely need to
stray beyond these basic methods. I can’t think of any use for `item()`,
`length`, or `toString()` as stand-alone methods, so don’t fret about them.

On another level, it’s about objects, it’s about finding powerful ways to
manipulate the state of web pages, it’s about separation of concerns, it’s about
progressive enhancement.

How far you take it is up to you, but as I said at the start, it’s an easy one
to get into and makes what was hard, easy. I encourage you to take what I’ve
written here and experiment in your own projects if you’re new to it, and take
to the comments to tell me where I could have said it better if you’re already
familiar with it.

Thank you!

[1]: http://html5doctor.com/demos/classlist/classlist.html
[2]: http://www.w3.org/TR/dom/#dom-element-classlist
[3]: http://dom.spec.whatwg.org/#domtokenlist
[4]: http://html5doctor.com/demos/classlist/classlist.add.html
[5]: http://html5doctor.com/demos/classlist/classlist.remove.html
[6]: http://dom.spec.whatwg.org/#dom-domtokenlist-add
[7]: http://html5doctor.com/demos/classlist/classlist.add-remove-many.html
[8]: http://lists.w3.org/Archives/Public/public-html-bugzilla/2011Sep/0032.html
[9]: http://html5doctor.com/demos/classlist/classlist.toggle.html
[10]: http://html5doctor.com/demos/classlist/classlist.toggle-true.html
[11]: http://html5doctor.com/demos/classlist/classlist.toggle-false.html
[12]: http://html5doctor.com/demos/classlist/classlist.item.html
[13]: http://caniuse.com/#search=classlist
[14]: http://developer.android.com/about/dashboards/index.html
[15]: https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#classlist
[16]: https://gist.github.com/devongovett/1381839
[17]: https://github.com/eligrey/classList.js
[18]: https://gist.github.com/termi/3952026