[1]: http://notebookandpenguin.com/single-responsibility-bem
[2]: http://bradfrostweb.com/blog/post/atomic-web-design/
# Single Responsibility BEM

Single Responsibility BEM is a DRYer way to write BEM-style CSS with Sass. I came up with the concept because I was annoyed with having to repeat the block before an element and modifier. Seriously. Over time, it became a system you can adapt to your workflow for more cohesive, easily maintainable styling and markup.

## Pros and Cons

### Pros

+ Encourages and supports building out from components.
+ Keeps modules in their own scope to be moved around.
+ Promotes consistent, maintainable styling.
+ A single point of contact for modifications.

### Cons

- Pretty much requires Sass.
- Adjoining classes aren't supported in IE6.
- May require a shift in thought.
- A lot of potential for abuse.

## Main Features

### Block Encapsulation

The ```.block``` element is a container for its elements, built to encapsulate the module. Blocks can contain other blocks, but those internal blocks are meant to contain their own elements, and so they won't be nested in the selector of its parent block. 

#### Example of a Block

    .hero {
    }

    <div class="hero">
    </div>

#### Example of an Internal Block

    .examples {
    }

    .contact {
    }

    <div class="examples">
      <h3 class="__title">Example 2: Augmenting a Block</h3>
      <form class="contact" id="ex1" action="">
      </form>
    </div>

Blocks contain their own CSS properties, and rarely interfere with those of their elements.

### Element Scope

Elements within blocks have their own scope and are named with ```__element```. Having their own scope that doesn't depend on the styling of the blocks allow elements to be shared across blocks. In your Sass, elements should be nested inside the block selector. For blocks that share elements, the one that comes later in the cascade may need to override or have styling of its own. You only need to define what changes across the element's context in a block.

#### Example of an Element in Its Block

    .examples {
      background: $white-translucent;
      border-radius: 0.5em;
      box-shadow: 2px 2px 3px desaturate(darken($sky-blue, 10), 40);
      margin-bottom: 1em;
      padding: 0.5em;
      &:after {
        content: "";
        clear: both;
        display: table;
      }
      .__title {
        margin-top: 0;
      }

      .__button {
        background: $ocean-blue-dark;
        color: $white;
        float: left;
        font-weight: 600;
        margin: 0.3em;
        padding: 0.8em;
        text-decoration: none;
        &:hover,
        &:focus {
          @extend %_--pulse;
          @extend %_--rave;
        }
      }
    }

#### Example of Blocks that Share Elements

    .examples {
      background: $white-translucent;
      border-radius: 0.5em;
      box-shadow: 2px 2px 3px desaturate(darken($sky-blue, 10), 40);
      margin-bottom: 1em;
      padding: 0.5em;
      &:after {
        content: "";
        clear: both;
        display: table;
      }
      .__button {
        background: $ocean-blue-dark;
        color: $white;
        float: left;
        font-weight: 600;
        margin: 0.3em;
        padding: 0.8em;
        text-decoration: none;
        &:hover,
        &:focus {
          @extend %_--pulse;
          @extend %_--rave;
        }
      }
    }

    .contact {
      @extend %_--inverseForm;
      float: left;
      width: 48%;

      .__button {
        border: none;
        display: block;
      }
    }

### The ```.augment``` Namespace

The ```.augment``` class is a special "namespace" for modifiers. Augment syntax is much like this: ```.augment-nameOfAugment%placeholder```. Placeholder selectors in Sass are like other selectors except they don't show up in the output. So when you create an modifier, only the augment class gets attached to the element or block you call it upon. Take the following modifier: 

    .augment-transitionFocus%_--focus {
      /* Referencing %_--focus */
      @include transition(all 0.4s linear);
      background: $white;
      border: 1px solid darken($ocean-blue-dark, 10);
      border-left: 8px solid darken($ocean-blue-dark, 10);
    }

Essentially we're creating a modifier that will apply the properties in the selector to ```.augment-transitionFocus```. Since we've defined it under its own context, all we need to do if we want to add and remove the rules are add and remove the class. So we extend the placeholder selector to our ```.__field``` elements.

      .__field {
        border: 2px solid $ocean-blue-dark;
        color: $grey;
        padding: 0.2em;
        &:focus {
          @extend %_--focus;
        }
      }

And then we apply our transition through the ```.augment-transitionFocus``` class in the markup where we need it.

    <label for="name">Name</label>
    <input type="text" name="name" id="name" class="__field augment-transitionFocus">
    <label for="email">Email</label>
    <input type="email" name="email" id="email" class="__field augment-transitionFocus">
    <label for="url">Website</label>
    <input type="url"  name="url" id="url" class="__field augment-transitionFocus">
    <label for="message">What's your message?</label>
    <textarea name="message" id="message" class="__field augment-transitionFocus" cols="30" rows="5"></textarea>

All elements with that class will have the modifier behavior applied. And we have a single point of contact for modifying that behavior.

### Block Modifiers

Blocks themselves can be modified in this way. The second form in Example 2 has a modifier of ```.augment-themeInvert```. It looks like this in Sass.

    .augment-themeInvert%_--inverseForm {
      /* referencing %_--inverseForm */
      background: $ocean-blue-dark;
      border-radius: 0.5em;
      padding: 0.2em;
      label {
        color: $white;
      }
      .__field {
        border-radius: 0 0.5em 0.5em 0;
      }
      .__button {
        background: $white;
        border-radius: 0 0 0 0.3em;
        color: $ocean-blue-dark;
      }
    }

When our modifier class is applied to the second form, we're just overriding the initial styling. From that single point of contact, we can try out different alternate themes without otherwise polluting our CSS. As you might imagine, modifier classes can be applied dynamically. All we're doing under the hood is applying and removing a reference when we put it in our markup.

### Modifiers and Media Queries

Placeholder selectors in Sass can't be used within media queries, but because our modifier classes are standalone, that doesn't matter. Placeholders *can* contain their own media queries, so we can differentiate styles that way. In the last example, we have a few blocks.

      <div class="block">Initial block.</div>
      <div class="block augment-stateMQ">At breakpoint "small", blocks invert</div>
      <div class="block augment-stateMQ">At breakpoint "medium", blocks have box-shadow</div>
      <div class="block augment-stateMQ">At breakpoint "large", blocks will be gold.</div>
      <div class="block augment-stateMQ">At breakpoint "super", blocks will have rounded corners.</div>
      <div class="block augment-stateMQ">At breakpoint "massive", blocks have bigger, bolder text;</div>

The first block remains in its initial state, but then we have this modifier.

    .augment-stateMQ%_--shiftBlock {
      @include mq-flow('small') {
        /* Referencing %_--shiftBlock */
        background: $white;
        color: $ocean-blue-dark;
      }
      @include mq-flow('medium') {
        /* Referencing %_--shiftBlock */
        box-shadow: 2px 3px 3px $grey;
      }

      @include mq-flow('large') {
        /* Referencing %_--shiftBlock */
        background: gold;
        color: $grey;
      }

      @include mq-flow('super') {
        /* Referencing %_--shiftBlock */
        border-radius: 1.5em;
      }

      @include mq-flow('massive') {
        /* Referencing %_--shiftBlock */
        @include scale(xl);
        font-weight: 700;
      }
    }

As the page flows through its media queries, the styles will be applied. And again, there's a single point of contact to make changes that will remain consistent wherever we happen to extend our modifier. All we have to do is add and remove ```.augment-stateMQ```.

## Conclusion

As always, no system is perfect. The merits of Single Responsibility BEM are yours to explore and adapt in any way you want. I want to help encourage a shift in building out systems for great design.
