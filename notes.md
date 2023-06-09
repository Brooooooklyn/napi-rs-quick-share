This is a quick sharing on NAPI-RS


First, let me introduce myself. I'm the creator of the NAPI-RS.


In simple terms, NAPI-RS is a bridge between Rust and Node.js in the Node.js ecosystem.
As you can see, in recent years, some performance-sensitive libraries and tools in the frontend community have been replacing slower JavaScript components with Rust.

Why do these popular libraries choose NAPI-RS, and what are the features of NAPI-RS that they cannot resist?

First of all, NAPI-RS allows for very concise syntax to turn a piece of Rust code into a function that can be directly called from JavaScript.

Compared to other Node-API libraries, NAPI-RS not only generates a callable library, but also generates a corresponding .d.ts file for it.

In the code example on the left, we can see that with just a few lines of code, we can provide the functionality of the Rust library `uuid` to any Node.js caller.

Not only does it include the functionality of uuid v4, but it also has amazing performance. Compared to the crypto.randomUUID provided by Node.js itself, it is about 13 times faster.

Here's a more complex example - let's pretend we wrote a bundler using NAPI-RS.

The `bundler` function takes a parameter called `plugin`, which accepts a string passed in by the bundler. We'll assume that this string is source code, and that `plugin` returns a `Promise<string>` type. Let's take a look at how NAPI-RS handles this situation.

From the implementation of `plugin`, we can see that `ts_arg_type` is used to override the types automatically generated by NAPI-RS. This is because Rust and TypeScript have significantly different type systems, so in some complex cases, NAPI-RS may not be able to generate perfect TypeScript types automatically. Therefore, in some cases, we can declare more precise types using the APIs provided by NAPI-RS.

In the specific logic, we pretend that `codes` is the code that the bundler needs to process, and pass it to the `plugin`. The `call_async` here means that we are asynchronously waiting for the return value of this function, so that we will not block the execution of JavaScript.

The biggest magic here is that the Promise returned from the plugin can be directly awaited in Rust. We await its return value here and return it.

Returning to the generated .d.ts file from NAPI-RS, we can see that the async fn in Rust is returned as a Promise in JavaScript, which means you can also await it in JavaScript!


Once this bundler is compiled, you can call it like this. The plugin here is a classic-style Node.js callback, where the first argument represents whether an error has occurred.

Now that we've learned how to develop a NAPI-RS module, it's time to start thinking about how to distribute it.

In traditional native addon solutions, developers may use postinstall scripts to compile native code from source during installation, or download pre-compiled binaries from a CDN.

You may have gone crazy dealing with various postinstall script failures when using native addons.

One of the best features of NAPI-RS is its out-of-the-box pre-compiled and distribution solution with zero postinstall, which is especially convenient for GitHub Actions users.

Of course, NAPI-RS also has other advantages, such as compatibility with serverless environments on platforms like Vercel/Netlify.

In terms of pre-compilation, NAPI-RS maintains a complex and rigorously tested toolchain that supports most of the mainstream platforms on the market.

You can see this huge table, which shows that NAPI-RS can precompile addons for practically all platforms on which Node.js can run.

In addition to the multi-platform pre-compiled solution provided by GitHub Action, NAPI-RS also offers cross-platform compilation solutions for other CI platform users or enterprise users. You can use NAPI-RS cli to compile binaries for the following platforms on Linux or macOS, and easily integrate them into CI.

Okay, up until now we've been talking about the advantages of NAPI-RS, but if it had no drawbacks, why haven't I rewritten all the packages on npm using NAPI-RS? So, now I'll discuss some of the trade-offs of using NAPI-RS.

It's including the following points.

Contrary to making complex tasks, this table showcases making Rust do the simplest calculations, and wrapping them with different Rust N-API frameworks, then comparing them to pure JavaScript.

You can see that in these simple computations, JavaScript can be several tens to hundreds of times faster than Rust.

But why is this the case? Shouldn't Rust be much faster than JavaScript? Actually, this is due to the cross boundary call overhead.

"Cross boundary call overhead" refers to the performance cost incurred when making function calls between different binaries or programming languages. In the case of Node.js and Rust addons, they are compiled into separate binaries, and traditional native language compiler optimization techniques such as LTO and PGO do not work when making cross-binary calls.

Secondly, this call crosses the JavaScript engine, so there is a lot of additional work to be done during the call, resulting in more performance overhead. As we can see in the 'sum' function, in addition to the necessary 'a + b' operation, there are five additional Node-API calls compared to a pure JavaScript implementation, which are used to handle type conversions for different runtimes.

Cross boundary calls can also cause other issues. In this example, the pure JavaScript implementation has the opportunity to be deeply optimized by the JavaScript engine. However, in the implementation on the right, the inserted nativeAddonFunction() may cause the entire function to be abandoned by the engine for optimization, which introducing more overhead.

The second major issue is Trusting. Recently, there have been more and more news reports of attackers using npm to distribute malicious code and steal secrets from developers' machines.

Using NAPI-RS or other frameworks to publish native add-ons will exacerbate this issue, as the released binary cannot be effectively audited.

However, this situation has recently improved as the official npm has released the provenance feature to sign published npm packages, preventing them from being tampered with during the publishing process.

The final major issue is debugging. In order to reduce installation size, most NAPI-RS binaries have debug symbols removed during release. The screenshot here is from issues on the official rspack repository, which shows that when issues occur, the error stack cannot be displayed at all, making it much more difficult to locate and fix the problem.


To solve this problem, NAPI-RS will provide a debug symbols download feature in the next major release to facilitate the downloading of debug symbols for printing error messages that contain source code when errors occur.

