---
layout: post
title: "Caching Drawing Code"
date: 2012-03-21 20:52

comments: true
sharing: true
footer: true

categories: 
- PaintCode
- Cocoa
- iOS
- Mac
- Code

---

***Updates**: On March 31st this post was updated based on feedback on twitter about using `NSCache` instead of `static`.*

I've been doing more and more drawing in code in my Cocoa projects recently and I'd like to share some tips, specifically how to cache drawing code into images for faster redraw. I've created a simple project for this post, the source of which can be found on [GitHub](https://github.com/kgn/PaintCodeExample). 

<!-- more -->

This project is very straight forward, there is a button that when pressed shows a popup that appears for a bit then dismisses it's self. There isn't a single image in the project it's all drawn in code!

{% img center http://kgn.github.com/content/drawing/animation.gif %}

The drawing code in this project was *painted* with [PaintCode](http://theindustry.cc/2012/03/21/paintcode-paint-cocoa-drawing-code), a great new mac app for generating OS X and iOS drawing code.

Drawing in code has several advantages, your app will take up less disk space because it doesn't have to ship with as many images. Depending on the drawing it can be easier to change a couple color values then it is to re-export a bunch of images. It also makes it easier to support multi-resolution displays because the drawing code will scale up and you don't need to manage 1x and 2x images.

{% img center http://kgn.github.com/content/drawing/multires.png %}

However drawing in code can cause performance issues because every time `setNeedsDisplay` is called on a view the drawing code needs to re-evaluate. This is especially noticeable in `UITableView` [scroll performance](http://stackoverflow.com/a/1352594/239380). There are also cases where images are expected, like buttons and image views. Fortunately drawing code can be cached into images and used just like an image loaded off of disk (*thanks to [@badeen](http://twitter.com/badeen) for originally turning me onto this approach*). I've put together a category for `UIImage` and `NSImage` in my [BBlock](https://github.com/kgn/BBlock) project that makes creating images with drawing code easier.

```
UIImage *image = [UIImage imageForSize:CGSizeMake(32, 32) withDrawingBlock:^{
    // Drawing code…   
}];
```

This code will cache your drawing code into a `UIImage` that's `32x32` or `64x64` on a retina display. The uprez for retina is automatically handled in this function, you can see what this function is doing [here](https://github.com/kgn/BBlock/blob/master/UIImage%2BBBlock.m).

**Update**: There is now a new function in [UIImage+BBlock.h](https://github.com/kgn/BBlock/blob/master/UIImage%2BBBlock.h) that caches the image in a `NSCache`. 

```
UIImage *image = [UIImage imageWithIdentifier:@"icon" forSize:CGSizeMake(32, 32) andDrawingBlock:^{
    // Drawing code…   
}];
```

Originally I recommended using `static` to store the resulting image but it was pointed out that `NSCache` is better for memory management. This post has been updated with more information about the advantages of `NSCache` over `static`.

## The example app

Ok let's jump into the code of the example app! As I said this app contains a button and a popup. The popup consists of a `UIView` that contains a `UIImageView` and `UILabel`. The `UILabel` is just a standard label, nothing fancy.

{% img center http://kgn.github.com/content/drawing/layout.png %}

In `viewDidLoad` the image displayed in the popup's `UIImageView` is set to `[self popupImage]` and the normal and highlight image for the button are set to `[self debutsImage]` and `[self debutsHighlightedImage]`.

```
- (void)viewDidLoad{
    [super viewDidLoad];
    
    [self.popupImageView setImage:[self popupImage]];    
    
    [self.debutsButton setImage:[self debutsImage] forState:UIControlStateNormal];
    [self.debutsButton setImage:[self debutsHighlightedImage] forState:UIControlStateHighlighted];    
}
```
*In the `viewDidLoad` of the example app there is also some code to setup the popup for animation, but that's not important to this post so it was left out of the above code snippet.*

These functions, that return images, are defined in [PCViewController+PaintCode.m](https://github.com/kgn/PaintCodeExample/blob/master/PaintCodeExample/PCViewController%2BPaintCode.m). The drawing code was split out into it's own category so the 500 lines of drawing code doesn't clutter the main view controller found in [PCViewController.m](https://github.com/kgn/PaintCodeExample/blob/master/PaintCodeExample/PCViewController.m).

The structure of these functions is similar but with different drawing code, for example here is an abridged version of `popupImage`:

```
- (UIImage *)popupImage{
    return [UIImage imageWithIdentifier:@"popupImage" forSize:CGSizeMake(120, 60) andDrawingBlock:^{
        // Drawing code...
    }];
}
```

This code uses the `BBlock` method mentioned above to draw into an image that's `120x60` or `240x120` on retina.

**Update**: This method originally used `static` to ensure that the drawing code was only executed once. However this new `BBlock` method now caches the image in an `NSCache` with the identifier name given. If the identifier doesn't exist in the cache the drawing code is executed and rendered to the returned image. If the identifier exists in the cache the cached image is returned and the drawing code is not run.

This caching lets us get a little tricky in `debutsHighlightedImage`. The highlight state of this button is the same icon as the normal state, but with a pink glow. So we can draw the glow and then draw the `debutsImage` ontop of it!

{% img center http://kgn.github.com/content/drawing/buttonicons.png %}

```
- (UIImage *)debutsHighlightedImage{
    return [UIImage imageWithIdentifier:@"debutsHighlightedImage" forSize:CGSizeMake(42, 42) andDrawingBlock:^{
        // Drawing code for the pink glow...
        
        // Draw the debuts image ontop of the glow
        [[self debutsImage] drawAtPoint:CGPointZero];
    }];
}
```

Again because the image returned from `debutsImage` is cached the drawing code will only be evaluated once so we are free to use this function multiple times without paying for multiple evaluations of the drawing code.

## Update: NSCache

As I've mentioned the code examples here and in the sample app originally use `static UIImage` to ensure that the drawing code was only evaluated once. However [@henrinormak](https://twitter.com/#!/henrinormak/status/185638889552228353) on twitter wondered about memory warnings, luckily [@crizzler](https://twitter.com/#!/crizzler/status/185419980567883778) had suggested using `NSCache` instead of static. I had never used `NSCache` before so I did some reading on it and some experimenting and it is a perfect fit for caching drawing code! It is much better than `static` for memory warnings because a static's object should not be released. Also `NSCache` automatically handles releasing cached objects when a memory warning is received on iOS! So when caching images into `NSCache` if the app receives a memory warning `NSCache` will release objects, then the next time the drawing code is run it will re-add the image to the cache.

I had really wanted to add an internal caching mechanism, like `[UIImage imageNamed]` does, to `BBlock` but I wasn't sure of the best approach. `NSCache` is a perfect fit for this, so a new method was added to `BBlock` which you can check out [here](https://github.com/kgn/BBlock/blob/master/UIImage%2BBBlock.m).

[OS X doesn't receive memory warnings](http://www.cocoabuilder.com/archive/cocoa/285674-simple-low-memory-warning.html) the way that iOS does but using `NSCache` is still very convenient and if they ever add memory warnings on OS X then your app will automatically take advantage of them. `BBlock` contains an identical api of it's [drawing code category for NSImage](https://github.com/kgn/BBlock/blob/master/NSImage%2BBBlock.h).

## Conclusion

There are lots of other ways to use drawing code cached into images. In apps where I've written my own `drawRect` method I've used a similar approach where I render the drawing code into an image and then draw that image into the view, stretching it, tiling it, or creating an image that fills the view and drawing that.

```
- (void)drawRect:(CGRect)rect{
    UIImage *image = [UIImage imageWithIdentifier:@"customView" forSize:self.bounds.size andDrawingBlock:^{
        // Drawing code...
    }];
    [image drawAtPoint:CGPointZero];
}
```

*This code example only works for static views were the size doesn't change. If the size of the view changed there would need to be some sort of cache invalidation, like storing and comparing the size of the view. Also `BBlock`'s internal cache couldn't be used, but you could use a `UIImage` ivar or your own `NSCache` instead.*

Another way to use drawing code cached into an image is to set the contents of a view's `CALayer`, this wasn't crucial to the sample app so I didn't want to dive into it in this post but there is a [CALayer branch](https://github.com/kgn/PaintCodeExample/tree/CALayer) in the sample app that demonstrates this on the popup view if you are curious.

Obviously it's not possible to cache everything, like a gradient in a widget that scales horizontally and vertically, but as much as possible it's a win to cache drawing code into images. I've tried to make this super easy to do and wrap up all the retina and caching logic into [BBlock](https://github.com/kgn/BBlock) and I hope you find it useful!

Hit me up on twitter([@_kgn](http://twitter.com/_kgn)) if you want to discuss this post.

