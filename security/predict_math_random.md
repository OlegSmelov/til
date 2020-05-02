# It's possible to predict Math.random()

**tl;dr** Output of pseudorandom number generators in most JavaScript engines can be predicted based on
a sample of 3-5 consecutive values of `Math.random()`.

------

JavaScript engines use XorShift128+ algorithm to generate a sequence of pseudorandom numbers.
By recreating the algorithm with [Z3 theorem prover][0], it's possible to recover algorithm's state
and recreate the same sequence of numbers.

A blog post by Douglas Goddard explains this in depth: [Hacking the JavaScript Lottery][1].

There's also a [reference implementation][2].

[0]: https://github.com/Z3Prover/z3
[1]: https://blog.securityevaluators.com/hacking-the-javascript-lottery-80cc437e3b7f
[2]: https://github.com/TACIXAT/XorShift128Plus
