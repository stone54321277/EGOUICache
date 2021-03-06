# EGOUICache

Created by enormego

**EGOUICache** is a collection of functions written on top of **EGOCache** to cache intensive `drawRect:` methods.  **EGOUICache** will cache these methods for you, and return a `UIImage` that you can quickly draw.  **EGOUICache** should not be used in `UITableViewCell`'s with data that is constantly change (like a Twitter app, for example).  It's intended to be used in areas where the data is pretty much set, but you need the option for it to change, giving you a dynamically generated, "static" cache.

# Methods

**EGOUICache** is powered by **EGOCache**, but it's written using C functions since they're faster and speed is what we're trying to achieve here.

*	`UIImage* EGOUICachedDrawing(id target, SEL selector, CGRect rect, NSString* salt);`
	*	*Caches the selector performed on target, this is the primary function and should be used with UIImage's drawInRect method*
	*	**Parameters**
		*	`target`: usually self, target that selector will be performed on
		*	`selector`: selector that will draw the "fresh" content
		*	`rect`: Rect to draw in
		*	`salt`: Unique string for the content in this cached drawing.
		
*	`EGOUICacheSetCacheTimeoutInterval(NSTimeInterval cacheTimeoutInterval);`
	*	*This should be called prior to calling `EGOUICachedDrawing()`, or within it as long as you're not nesting cache calls.*
	*	**Parameters**
		* `cacheTimeoutInterval` number of seconds the cache should persist for
	*	**Convenience methods**
		* `EGOUICacheSetCacheTimeoutOneYear();`
		* `EGOUICacheSetCacheTimeoutOneMonth();`
		* `EGOUICacheSetCacheTimeoutOneWeek();`
		* `EGOUICacheSetCacheTimeoutOneDay();`
		* `EGOUICacheSetCacheTimeoutOneHour();`

*	`EGOUICacheCancel();`
	*	Cancels one level of caching.  This should be called if data hasn't yet loaded or something isn't what it's expected to be.  The image returned by `EGOUICachedDrawing()` will be nil.

# Example

Let's say you have a button that gets drawn a bunch of times, that uses intensive blending, and really only needs to be drawn once since the text rarely changes:

	- (void)drawRect:(CGRect)rect {
		EGOUICacheSetCacheTimeoutOneMonth();
		[EGOUICachedDrawing(self, @selector(freshDrawRect:), rect, self.currentTitle) drawInRect:rect];
	}

	- (void)freshDrawRect:(CGRect)rect {
		UIFont* font = [UIFont boldSystemFontOfSize:22.0f];
	
		CGRect textRect;
		textRect.size = [text sizeWithFont:font];
		textRect.origin.x = floorf((rect.size.width - titleSize.width) / 2);
		textRect.origin.y = floorf((rect.size.height - titleSize.height) / 2) + 1.0f;
	
		// A bunch of complicated code that draws the text with a gradient, and glow around it, with a 1px shadow
	}

The `EGOUICachedDrawing()` call in `drawRect` will check to see if it has a cache for the target/selector with the salt being the current title, if it does, it returns the `UIImage` right away.  If it doesn't detect the cache, it calls `freshDrawRect:`, and then caches and returns an image with the contents drawn in `freshDrawRect:`.

# License

EGOUICache is available under the MIT license:

*Copyright (c) 2009-2010 enormego*

*Permission is hereby granted, free of charge, to any person obtaining a copy*
*of this software and associated documentation files (the "Software"), to deal*
*in the Software without restriction, including without limitation the rights*
*to use, copy, modify, merge, publish, distribute, sublicense, and/or sell*
*copies of the Software, and to permit persons to whom the Software is*
*furnished to do so, subject to the following conditions:*

*The above copyright notice and this permission notice shall be included in*
*all copies or substantial portions of the Software.*

*THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR*
*IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,*
*FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE*
*AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER*
*LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,*
*OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN*
*THE SOFTWARE.*