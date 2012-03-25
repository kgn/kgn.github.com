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

I've been doing more and more drawing in code in my Cocoa projects recently and I'd like to share some tips, specifically how to cache drawing code into images for faster redraw. I've created a simple project for this post, the source of which can be found on [GitHub](https://github.com/kgn/PaintCodeExample). 

<!-- more -->

This project is very straight forward, there is a button that when pressed shows a popup that appears for a bit then dismisses it's self. There isn't a single image in the project it's all drawn in code!

{% img center http://kgn.github.com/content/drawing/animation.gif %}

The drawing code in this project was *painted* with [PaintCode](http://theindustry.cc/2012/03/21/paintcode-paint-cocoa-drawing-code), a great new mac app for generating OS X and iOS drawing code.

Drawing in code has several advantages, your app will take up less disk space because it doesn't have to ship with as many images. Depending on the drawing it can be easier to change a couple color values then it is to re-export a bunch of images. It also makes it easier to support multi-resolution displays because the drawing code will scale up and you don't need to manage 1x and 2x images.

{% img center http://kgn.github.com/content/drawing/multires.png %}

However drawing in code can cause performance issues because every time `setNeedsDisplay` is called on a view the drawing code needs to re-evaluate. There are also cases where images are expected, like buttons and image views. Fortunately drawing code can be cached into images and used just like an image loaded off of disk (*thanks to [@badeen](http://twitter.com/badeen) for originally turning me onto this approach*). I've put together a category for `UIImage` and `NSImage` in my [BBlock](https://github.com/kgn/BBlock) project that makes creating images with drawing code easier.

```
UIImage *image = [UIImage imageForSize:CGSizeMake(32, 32) withDrawingBlock:^{
    // Drawing code…   
}];
```

This code will cache your drawing code into a `UIImage` that's `32x32` or `64x64` on a retina display. The uprez for retina is automatically handled in this function, you can see what this function is doing [here](https://github.com/kgn/BBlock/blob/master/UIImage%2BBBlock.m).

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
    static UIImage *image = nil;
    if(image == nil){
        image = [UIImage imageForSize:CGSizeMake(120, 60) withDrawingBlock:^{
            // Drawing code...
        }];
    }
    return image;
```

This code uses the `BBlock` method mentioned above to draw into an image that's `120x60`, the resulting image is stored in a `static UIImage` so we can call this method multiple times and we will always get the same `UIImage`. If we used this technique for an image in a table view without the `static` we wouldn't get any savings from caching the code in a `UIImage` because the drawing code would be evaluated whenever the function was called which would be every time a row needed to be redrawn.

This also lets us get a little tricky in `debutsHighlightedImage`. The highlight state of this button is the same icon as the normal state, but with a pink glow. So we can draw the glow and then draw the `debutsImage` ontop of it!

{% img center http://kgn.github.com/content/drawing/buttonicons.png %}

```
- (UIImage *)debutsHighlightedImage{
    static UIImage *image = nil;
    if(image == nil){
        CGSize imageSize = CGSizeMake(42, 42);
        image = [UIImage imageForSize:imageSize withDrawingBlock:^{
            // Drawing code for the pink glow...
            
            // Draw the debuts image ontop of the glow
            [[self debutsImage] drawAtPoint:CGPointZero];
        }]; 
    }
    return image;
}
```

Again because the image returned from `debutsImage` is `static` the drawing code will only be evaluated once so we are free to use this function multiple times without paying for multiple evaluations of the drawing code.

## Conclusion

There are lots of other ways to used drawing code cached into images. In apps where I've written my own `drawRect` method I've used a similar approach where I cache the drawing code into an image and then draw that image into the rect, stretching it, tiling it, or creating an image that fills the view and drawing that.

```
- (void)drawRect:(CGRect)rect{
    static UIImage *image = nil;
    if(image == nil){
        image = [UIImage imageForSize:self.bounds withDrawingBlock:^{
            // Drawing code...
        }];
    }
    [image drawAtPoint:CGPointZero];
}
```

*This code example only works for static views were the size doesn't change. If the size of the view changed there would need to be some sort of cache invalidation, like storing and comparing the size of the view. Also `static` couldn't be used, but you could use a `UIImage` ivar instead.*

Obviously it's not possible to cache everything, like a gradient in a widget that scales horizontally and vertically, but as much as possible it's a win to cache drawing code into images. 

Hit me up on twitter([@_kgn](http://twitter.com/_kgn)) if you want to discuss this post.

