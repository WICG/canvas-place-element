# Open Design Questions

## **Privacy-preserving rendering**

placeElement can taint the canvas2D as needed, so even cross-origin content could eventually be placed without privacy issues. For drawElement, there’s no tainting mechanism for WebGL/WebGPU but even if it existed, the existence of shaders makes it that all rendered content should be assumed to be readable by the page. This is why WebGL/WebGPU today disallows rendering cross-origin images.

To support drawElement at all, we need to make sure that the rendering of the elements is done in a privacy-preserving way. Visited links may be changing soon to be domain-based. But other issues (like dictionary spelling) may need to be disabled.

## **Complexity of rendering certain elements**

It’s possible that some elements may not be renderable to a texture (2D or 3D). For those cases, we are ready to have an initial disallowed list of elements to simplify implementation by UAs. For those cases, placeElement() could return null to indicate this placeElement is not supported.

## **Re-compositing for placeElement**

placeElement requires the browser to recompose the canvas buffer each time the element changes. This should be equivalent to rendering all commands executed on the canvas with the latest contents for each placed element. If implemented as such, this would be inefficient since the buffer of commands can grow unbounded.

The browser could optimize by squashing sets of commands into buffers at placeElement boundaries whenever possible (each saveLayer would also need to split up?).

## **Hit Testing Data structures**

The canvas subtree is laid out based on the DOM/style applied to the elements. However, their contents are not painted based on the position or ordering of their boxes. It is instead defined by the canvas commands which render the element’s image into the canvas buffer.

This affects both hit-testing and behaviour of platform APIs (like IntersectionObserver) which allows authors to observe the position of these elements. This is the data provided by CanvasDrawElements::updateElement API for drawElement mode and done automatically by the browser for placeElement mode.

## Other issues

1. Should placed elements be direct children of the canvas or any descendant is ok? Might be harder to fake the element’s rendered position in the canvas if there are boxes between the canvas and the element in the tree.
2. How are non-placed canvas children represented in the layout tree. We likely need to layout the entire subtree (especially if a placed element can be a descendant) and make everything not placed visibility: hidden so they are not painted and inert.
3. Should we force containment (layout or paint) on canvas to ensure the children don’t affect the layout of the rest of the Document?
4. Is it worth providing the invalidation rect (based on element bounds) or will authors always do full redraw?
5. Should draw commands after placeElement block the interaction rect of elements (since they would occlude the element)? Should there be a context state to control this behavior?
6. When painting the elements, which box should be painted? Should it include shadow? If so, how does the developer know what is the full rendered size?
7. If developers want to align placeElements (either by baseline or box), how are they supposed to do it?
8. What happens to placed inline elements?
9. Clarify the timing for when the image is generated when using drawImage. Ideally it should be at the end of all script callbacks during [update the rendering](https://html.spec.whatwg.org/\#update-the-rendering) but unclear whether that’s possible for 3D.
10. The author might draw a subset of the element. Should we add a “src rect” to updateElement.
11. Should we allow a place element to be a descendant of another place element? It will cause double draw of the descendant and also make it harder to reason about the element’s onscreen position.
12. Placed elements must be stacking contexts. Do we need any other restriction?
13. Should we force blockifying canvas children?
14. place element vs draw element for 2d canvas


