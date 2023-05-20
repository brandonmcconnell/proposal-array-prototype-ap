# proposal-array-prototype-ap
The `Array.prototype.ap` method applies an array of functions to a single argument. Given an array `fns` of functions, the expression `fns.ap(x)` is analogous to `fns.map(fn => fn(x))`. A new array is formed elements resulting from applying each function from the array to the provided argument.
