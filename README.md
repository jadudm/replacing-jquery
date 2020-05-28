# Replacing JQuery

Ticket [#1044](https://github.com/18F/tock/issues/1044) in the tock issues list is about replacing JQuery.

One way would be to rewrite all the UI actions that involve JQuery. This seems error-prone and time consuming.

My question is: should we replace JQuery with a tiny abstraction that *looks* like JQuery, but is just a function call or two around vanilla JS actions? This seems straight-forward (danger!), but it is testable (yay!). 

The result would be a lightweight "mini-JQuery" that eliminates the JQuery reliance. However, it would introduce a dangerous pathway to having our "own version of JQuery..."

## Example

```javascript
 1 let MiniQuery = class {
 2    constructor (queryString) {
 3        this.queryString = queryString;
 4        this.elements = document.querySelectorAll(queryString);
 5    }
 6
 7    hide () {
 8        this.elements.forEach(e => { e.style.display = "none" });
 9        return this;
10    }
11 }

12 function $ (selector) {
13    q = new MiniQuery(selector);
14    return q;
15 }
```

The JQuery operation `$(".red")` will return all elements on a page that are `class = "red"`. Therefore, this is equivalent to the `querySelectorAll()` search in vanilla JS.

The `MiniQuery` class can be built to support the subset of operations that we use from `$` in JQuery. Then, instead of rewriting all of the uses of JQuery in Tock, we simply change our import. 

In this example, I've implemented `.hide()` as a proof-of-concept.

```html

 1    <div id="one" class="a">One</div>
 2    <div id="two" class="a">Two</div>
 3    <div id="three" class="b">Three</div>
 4    <div id="four" class="b">Four</div>
 5    <div id="five" class="c">Five</div>
 6
 7    <script>
 8        $(".a").hide();
 9        $("#five").hide();
10    </script>
```

This example should yield a page that appears to contain the text "Three" and "Four," as the first two `div`s are hidden (by a `class` selector), and the last is hidden by an `id` selector.

## Testing

If there are UI tests in place, we should "just pass" those tests if everything is right. If not, we know we got something wrong. If there are no UI tests, perhaps we should put in UI tests first, and then enact the transformation?

## Other Possibilities

1. We could parse and rewrite the code automatically, replacing all JQuery calls with their appropriate vanilla JS calls. Probably more time-consuming than rewriting things by hand.

2. We could rewrite all of the JQuery calls by hand.

