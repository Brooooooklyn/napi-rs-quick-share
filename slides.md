---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## NAPI-RS
  A framework for building compiled Node.js add-ons in Rust via Node-API 

  Learn more at [NAPI-RS](https://napi.rs)
# persist drawings in exports and build
drawings:
  persist: false
# page transition
transition: slide-left
# use UnoCSS
css: unocss
---

# Developing high-performance Node.js addons with Rust and NAPI-RS

[@Brooooooklyn](github.com/brooooooklyn/)

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!-- This is a quick sharing on NAPI-RS. -->

---

# Self Introduction

My name is LongYinan, you can also call me [@Brooooooklyn](github.com/brooooooklyn/) as well.

<br/>

- The creator of [NAPI-RS](https://napi.rs).
- Worked at Vercel, ByteDance, LeetCode and Teambition.
- Leading the `OctoBase` team at AFFiNE, building the local-first collaborative infrastructure.

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
First, let me introduce myself. I'm the creator of the NAPI-RS.
-->

---

# What is NAPI-RS

> NAPI-RS is the bridge between Rust and JavaScript in Node.js.

<p>
  <img src="/prisma.svg"/>
  <span><img src="/parcel.avif" style="height: 40px;"/><img src="/parcel.svg" style="height: 20px;" /></span>
  <img src="/swc.png" style="height: 40px;" />
  <img src="/next-logo.png" style="height: 40px; margin-right: 5px;" /><img src="/next-text.svg"/>
  <img src="/rspack.png" style="height: 50px;" />
  <span class="rspack-text">Rspack</span>
</p>

<div class="tweet-container">
  <Tweet class="tweet-card" id="1395401586503864326" scale="0.6" />
  <Tweet class="tweet-card" id="1453031104097574916" scale="0.6" />
  <Tweet class="tweet-card" id="1398026549320982531" scale="0.7" />
  <Tweet class="tweet-card" id="1634009336161918978" scale="0.7" />
</div>

<style>
  p, span {
    display: flex;
    align-items: center;
  }
  img {
    margin-left: 10px;
  }
  .rspack-text {
    background: -webkit-linear-gradient(120deg, #ffa500 30%, #f4f468);
    background-clip: text;
    -webkit-text-fill-color: transparent;
    font-weight: 700;
    font-family: "Inter var experimental", "Inter var", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  }
  .tweet-container {
    display: flex;
  }
  .tweet-card:not(:first-child) {
    margin-left: -40px;
  }
</style>

<!--
In simple terms, NAPI-RS is a bridge between Rust and Node.js in the Node.js ecosystem.
As you can see, in recent years, some performance-sensitive libraries and tools in the frontend community have been replacing slower JavaScript components with Rust.

Why do these popular libraries choose NAPI-RS, and what are the features of NAPI-RS that they cannot resist?
-->

---
layout: two-cols
---

# Rust codes

```rust {all|4|5-7|all}
use napi::bindgen_prelude::*;
use napi_derive::napi;

#[napi]
pub fn v4() -> String {
  uuid::v4()
}
```

<style>
.slidev-code {
  margin-right: 10px;
}
</style>

::right::

# Generated API

```ts
export function v4(): string;
```

```bash
crypto.randomUUID:
  810 767 ops/s, ±0.50%      | slowest, 92.69% slower

napi-uuid-v4:
  11 084 015 ops/s, ±0.43%   | fastest

Finished 2 cases!
  Fastest: napi-uuid-v4
  Slowest: crypto.randomUUID
```

[Learn More](github.com/brooooooklyn/uuid/)

<!--
First of all, NAPI-RS allows for very concise syntax to turn a piece of Rust code into a function that can be directly called from JavaScript.

Compared to other Node-API libraries, NAPI-RS not only generates a callable library, but also generates a corresponding .d.ts file for it.

In the code example on the left, we can see that with just a few lines of code, we can provide the functionality of the Rust library `uuid` to any Node.js caller.

Not only does it include the functionality of uuid v4, but it also has amazing performance. Compared to the crypto.randomUUID provided by Node.js itself, it is about 13 times faster.
-->

---

# Dumb Bundler

#### `src/lib.rs`

```rust {all|5|6-7|9-13|12|all}
use napi::{bindgen_prelude::*, threadsafe_function::ThreadsafeFunction};
use napi_derive::napi;

#[napi]
pub async fn bundler(
  #[napi(ts_arg_type = "(err: Error | null, codes: string) => Promise<string>")]
  plugin: ThreadsafeFunction<String>,
) -> Result<String> {
  let codes = "const a: number = 1;";
  let plugin_result: Promise<String> =
    plugin.call_async(Ok(codes.to_owned())).await?;
  let result_from_promise = plugin_result.await?;
  Ok(result_from_promise)
}

```

#### `index.d.ts`

```ts
export function bundler(plugin: (err: Error | null, codes: string) => Promise<string>): Promise<string>
```

<!--
Here's a more complex example - let's pretend we wrote a bundler using NAPI-RS.

The `bundler` function takes a parameter called `plugin`, which accepts a string passed in by the bundler. We'll assume that this string is source code, and that `plugin` returns a `Promise<string>` type. Let's take a look at how NAPI-RS handles this situation.

From the implementation of `plugin`, we can see that `ts_arg_type` is used to override the types automatically generated by NAPI-RS. This is because Rust and TypeScript have significantly different type systems, so in some complex cases, NAPI-RS may not be able to generate perfect TypeScript types automatically. Therefore, in some cases, we can declare more precise types using the APIs provided by NAPI-RS.

In the specific logic, we pretend that `codes` is the code that the bundler needs to process, and pass it to the `plugin`. The `call_async` here means that we are asynchronously waiting for the return value of this function, so that we will not block the execution of JavaScript.

The biggest magic here is that the Promise returned from the plugin can be directly awaited in Rust. We await its return value here and return it.

Returning to the generated .d.ts file from NAPI-RS, we can see that the async fn in Rust is returned as a Promise in JavaScript, which means you can also await it in JavaScript!
-->

---

# Call the bundler

<br />

<div class="columns-2">

```js
import { transform } from '@swc/core'

import { bundler } from './index.js'

const plugin = async (err, codes) => {
  if (err) {
    console.error(err)
  } else {
    const { code } = await transform(codes, {
      jsc: {
        parser: {
          syntax: 'typescript'
        }
      }
    })
    return code
  }
}

const res = await bundler(plugin)

console.log(res)
```

![](/bundler-output.png)

</div>

<!--
Once this bundler is compiled, you can call it like this. The plugin here is a classic-style Node.js callback, where the first argument represents whether an error has occurred.
-->

---

# Distribution

- Support **zero** postinstall script.
- Generate JavaScript bindings file to load different platform binary which compatible with modern bundler and deploy platform.
- Minimal download size with modern `packageManager` via `libc` `os` and `cpu` field in `package.json`.
- Minimal `glibc(2.17)` requirement for Linux gnu platform via official Docker builder.

<!--
Now that we've learned how to develop a NAPI-RS module, it's time to start thinking about how to distribute it.

In traditional native addon solutions, developers may use postinstall scripts to compile native code from source during installation, or download pre-compiled binaries from a CDN.

You may have gone crazy dealing with various postinstall script failures when using native addons.

One of the best features of NAPI-RS is its out-of-the-box pre-compiled and distribution solution with zero postinstall, which is especially convenient for GitHub Actions users.

Of course, NAPI-RS also has other advantages, such as compatibility with serverless environments on platforms like Vercel/Netlify.
-->

---

|                       | node12 | node14 | node16 | node18 | node20 |
| --------------------- | ------ | ------ | ------ | ------ | ------ |
| Windows x64           | ✓      | ✓      | ✓      | ✓      | ✓      |
| Windows x86           | ✓      | ✓      | ✓      | ✓      | ✓      |
| Windows arm64         | ✓      | ✓      | ✓      | ✓      | ✓      |
| macOS x64             | ✓      | ✓      | ✓      | ✓      | ✓      |
| macOS aarch64         | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux x64 gnu         | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux x64 musl        | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux aarch64 gnu     | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux aarch64 musl    | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux arm gnueabihf   | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux riscv64 gnu     | N/A    | N/A    | ✓      | ✓      | ✓      |
| Linux aarch64 android | ✓      | ✓      | ✓      | ✓      | ✓      |
| Linux armv7 android   | ✓      | ✓      | ✓      | ✓      | ✓      |
| FreeBSD x64           | ✓      | ✓      | ✓      | ✓      | ✓      |

<style>
  .slidev-layout {
    padding-top: 10px;
  }
  .slidev-layout h1 {
    font-size: 16px;
    margin-bottom: 0;
  }
  table {
    font-size: 8px;
  }
</style>

<!--
In terms of pre-compilation, NAPI-RS maintains a complex and rigorously tested toolchain that supports most of the mainstream platforms on the market.

You can see this huge table, which shows that NAPI-RS can precompile addons for practically all platforms on which Node.js can run.
-->

---

# Cross build

> Available on Linux and macOS.

- Windows x64
- Windows x86
- Windows arm64
- macOS x64
- macOS arm64
- Linux x64 gnu (glibc 2.17)
- Linux x64 musl
- Linux arm64 gnu (glibc 2.17)
- Linux arm64 musl
- Linux armv7 gnueabihf
- Android arm64
- Android armv7

[Learn More](https://napi.rs/docs/cross-build/summary)

<!--
In addition to the multi-platform pre-compiled solution provided by GitHub Action, NAPI-RS also offers cross-platform compilation solutions for other CI platform users or enterprise users. You can use NAPI-RS cli to compile binaries for the following platforms on Linux or macOS, and easily integrate them into CI.
-->

---

# Trade off
> If NAPI-RS were perfect, I would have already rewritten all npm packages with the help of ChatGPT.

<br/>

- Cross boundary call overhead.
- Trusting.
- Debugging.

<!--
Okay, up until now we've been talking about the advantages of NAPI-RS, but if it had no drawbacks, why haven't I rewritten all the packages on npm using NAPI-RS? So, now I'll discuss some of the trade-offs of using NAPI-RS.

It's including the following points.
-->

---

# Cross boundary call overhead

|                             | JavaScript | napi-rs                    | napi-rs-compat                | neon                         | node-bindgen |
| --------------------------- | --------------------------- | -------------------------- | ----------------------------- | ---------------------------- | --------------------------- |
| Sum (a + b)                 | 1 020 324 001 ops/s, ±0.33% **2686%** | 37 987 529 ops/s, ±0.07%  **100%** | 33 650 128 ops/s, ±0.26%   **73.03%** | 23 977 894 ops/s, ±0.09%  **61.65%** | 19 623 929 ops/s, ±0.11% **51.66%** |
| Concat(Hello + " world")    | 1 018 192 820 ops/s, ±0.33%  **10220%** | 9 962 492 ops/s, ±0.29% **100%** | 8 406 430 ops/s, ±0.25%  **84.38%** | 9 615 266 ops/s, ±0.38%  **96.51%** | 6 283 797 ops/s, ±0.21% **63.07%** |
| Rect Area { width, height } | 767 466 974 ops/s, ±17.59%  **17711%** | 4 333 189 ops/s, ±0.26% **98.18%** | 4 413 512 ops/s, ±0.23% **100%** | 3 614 248 ops/s, ±0.25% **81.89%** | 2 458 340 ops/s, ±0.27% **55.70%** |

<!--
Contrary to making complex tasks, this table showcases making Rust do the simplest calculations, and wrapping them with different Rust N-API frameworks, then comparing them to pure JavaScript.

You can see that in these simple computations, JavaScript can be several tens to hundreds of times faster than Rust.

But why is this the case? Shouldn't Rust be much faster than JavaScript? Actually, this is due to the cross boundary call overhead.
-->

---
layout: two-cols
---
# JavaScript

<br/>

All operation in V8 under the hood, running in the high optimized JIT runtime.

::right::

# NAPI-RS

```rust
#[napi]
pub fn sum(a: u32, b: u32) -> u32 {
  a + b
}
```

- Get arguments information via [napi_get_cb_info](https://nodejs.org/api/n-api.html#napi_get_cb_info)
- Convert arguments[0] from JavaScript number into Rust `u32` via [napi_get_value_uint32](https://nodejs.org/api/n-api.html#napi_get_value_uint32)
- Convert arguments[1] from JavaScript number into Rust `u32` via [napi_get_value_uint32](https://nodejs.org/api/n-api.html#napi_get_value_uint32)
- <span color="green">Perform `a + b`</span>
- Convert Rust `u32` into JavaScript number via [napi_create_uint32](https://nodejs.org/api/n-api.html#napi_create_uint32)

<!--
"Cross boundary call overhead" refers to the performance cost incurred when making function calls between different binaries or programming languages. In the case of Node.js and Rust addons, they are compiled into separate binaries, and traditional native language compiler optimization techniques such as LTO and PGO do not work when making cross-binary calls.

Secondly, this call crosses the JavaScript engine, so there is a lot of additional work to be done during the call, resulting in more performance overhead. As we can see in the 'sum' function, in addition to the necessary 'a + b' operation, there are five additional Node-API calls compared to a pure JavaScript implementation, which are used to handle type conversions for different runtimes.
-->

---
layout: two-cols
---
# Pure JavaScript execution

<br/>

```js {all|2,3,4}
function someHotPath() {
  someHeavyJavaScriptFunction1()
  someHeavyJavaScriptFunction2()
  someHeavyJavaScriptFunction3()
}
```

<style>
.slidev-code {
  margin-right: 10px;
}
</style>

::right::

# With Rust addon

<br/>

```js {monaco-diff}
function someHotPath() {
  someHeavyJavaScriptFunction1()
  someHeavyJavaScriptFunction2()
  someHeavyJavaScriptFunction3()
}
~~~
function someHotPath() {
  someHeavyJavaScriptFunction1()
  nativeAddonFunction()
  someHeavyJavaScriptFunction3()
}
```

<!--
Cross boundary calls can also cause other issues. In this example, the pure JavaScript implementation has the opportunity to be deeply optimized by the JavaScript engine. However, in the implementation on the right, the inserted nativeAddonFunction() may cause the entire function to be abandoned by the engine for optimization, which introducing more overhead.
-->

---

# Trusting

> Malicious code is very common on npm, and native addons make it easier for attackers to hide their malicious code.

<br/>

<p class="columns-3">
  <div class="-outline-offset-1 w-[280px] max-w-full cursor-pointer overflow-hidden rounded-[0.85714em] border-[1px] border-[#e1e8ed] leading-[1.3em]"><div class="h-[160px] border-b-[1px] border-[#e1e8ed] bg-cover bg-center bg-no-repeat" style="background-image: url(&quot;https://eu-images.contentstack.com/v3/assets/blt66983808af36a8ef/blt79105a2a91f85310/62c499735211587678f50af6/opensource_Izel_Photography_Alamy.jpg&quot;);"></div><div class="break-words border-[#dadde1] p-[0.75em] antialiased"><div class="block border-separate select-none overflow-hidden break-words text-left"><div class="m-0 mb-[0.15em] truncate text-[12px] font-semibold leading-[18px]" style="color: white;">Supply Chain Attack Deploys Hundreds of Malicious NPM Modules to Steal Data</div><div class="mt-[.32333em] block max-h-[2.6em] border-separate select-none overflow-hidden truncate whitespace-nowrap break-words text-left text-[12px] leading-[18px]" style="-webkit-line-clamp: 1; -moz-box-orient: vertical; color: white;">A widespread campaign uses more than 24 malicious NPM packages loaded with JavaScript obfuscators to steal form data from multiple sites and apps, analysts report.</div></div><div class="mt-[0.32333em] overflow-hidden truncate whitespace-nowrap text-[12px] lowercase leading-[18px] text-[#8899a6]">darkreading.com</div></div></div>

  <div class="-outline-offset-1 w-[300px] max-w-full cursor-pointer overflow-hidden rounded-[0.85714em] border-[1px] border-[#e1e8ed] leading-[1.3em]"><div class="h-[160px] border-b-[1px] border-[#e1e8ed] bg-cover bg-center bg-no-repeat" style="background-image: url(&quot;https://github.blog/wp-content/uploads/2022/04/Engineering-Security@2x.png?fit=2400%2C1260&quot;);"></div><div class="break-words border-[#dadde1] p-[0.75em] antialiased"><div class="block border-separate select-none overflow-hidden break-words text-left"><div class="m-0 mb-[0.15em] truncate text-[14px] font-semibold leading-[18px]">Security alert: Attack campaign involving stolen OAuth user tokens issued to two third-party integrators | The GitHub Blog</div><div class="mt-[.32333em] block max-h-[2.6em] border-separate select-none overflow-hidden truncate whitespace-nowrap break-words text-left text-[14px] leading-[18px]" style="-webkit-line-clamp: 1; -moz-box-orient: vertical;">On April 12, GitHub Security began an investigation that uncovered evidence that an attacker abused stolen OAuth user tokens issued to two third-party OAuth integrators, Heroku and Travis-CI, to download data from dozens of organizations, including npm. Read on to learn more about the impact to GitHub, npm, and our users.</div></div><div class="mt-[0.32333em] overflow-hidden truncate whitespace-nowrap text-[14px] lowercase leading-[18px] text-[#8899a6]">github.blog</div></div></div>

  <div class="-outline-offset-1 w-[300px] max-w-full cursor-pointer overflow-hidden rounded-[0.85714em] border-[1px] border-[#e1e8ed] leading-[1.3em] text-white"><div class="h-[160px] border-b-[1px] border-[#e1e8ed] bg-cover bg-center bg-no-repeat" style="background-image: url(&quot;https://media.jfrog.com/wp-content/uploads/2022/03/23201257/npm-Attack-Targets-Azure-Developers-Security-research-team_social.png&quot;);"></div><div class="break-words border-[#dadde1] p-[0.75em] antialiased"><div class="block border-separate select-none overflow-hidden break-words text-left"><div class="m-0 mb-[0.15em] truncate text-[12px] font-semibold leading-[18px]">Malicious Packages in npm Targeting Azure Developers</div><div class="mt-[.32333em] block max-h-[2.6em] border-separate select-none overflow-hidden truncate whitespace-nowrap break-words text-left text-[12px] leading-[18px]" style="-webkit-line-clamp: 1; -moz-box-orient: vertical;">JFrog discovers hundreds of npm malicious packages in a large-scale typosquatting attack designed to steal PII from Azure developers. Find out more &gt;</div></div><div class="mt-[0.32333em] overflow-hidden truncate whitespace-nowrap text-[12px] lowercase leading-[18px] text-[#8899a6]">jfrog.com</div></div></div>
</p>

<!--
The second major issue is Trusting. Recently, there have been more and more news reports of attackers using npm to distribute malicious code and steal secrets from developers' machines.

Using NAPI-RS or other frameworks to publish native add-ons will exacerbate this issue, as the released binary cannot be effectively audited.
-->

---

# Trusting

> npm provided a strategy to verifiably link npm packages to their source repository and build instructions recently.

<br />

<div class="columns-2">

<div class="-outline-offset-1 w-[320px] max-w-full cursor-pointer overflow-hidden rounded-[0.85714em] border-[1px] border-[#e1e8ed] leading-[1.3em] text-white"><div class="h-[160px] border-b-[1px] border-[#e1e8ed] bg-cover bg-center bg-no-repeat" style="background-image: url(&quot;https://github.blog/wp-content/uploads/2023/04/introducing-npm-package-provenance.jpg&quot;);"></div><div class="break-words border-[#dadde1] p-[0.75em] antialiased"><div class="block border-separate select-none overflow-hidden break-words text-left"><div class="m-0 mb-[0.15em] truncate text-[14px] font-semibold leading-[18px]">Introducing npm package provenance | The GitHub Blog</div><div class="mt-[.32333em] block max-h-[2.6em] border-separate select-none overflow-hidden truncate whitespace-nowrap break-words text-left text-[14px] leading-[18px]" style="-webkit-line-clamp: 1; -moz-box-orient: vertical;">How to verifiably link npm packages to their source repository and build instructions.</div></div><div class="mt-[0.32333em] overflow-hidden truncate whitespace-nowrap text-[14px] lowercase leading-[18px] text-[#8899a6]">github.blog</div></div></div>

<img src="/swc-npm.png" />

</div>

<!--
However, this situation has recently improved as the official npm has released the provenance feature to sign published npm packages, preventing them from being tampered with during the publishing process.
-->

---

# Debugging

> Debugging is a pain point because NAPI-RS projects often remove debug symbols before publishing to npm in order to reduce download size, but this makes it very difficult to identify the cause of a panic when it occurs.


<div class="columns-3">

  <p class="inline-block">

  [#3297](https://github.com/web-infra-dev/rspack/issues/3297)
  ```bash
  [rspack-cli] [Error: error[internal]: 88aa83392a-572612741d is not a valid expression.
  Raw: panicked at '88aa83392a-572612741d is not a valid expression', /Users/runner/.cargo/registry/src/index
  .crates.io-6f17d22bba15001f/swc-0.261.21/src/config/mod.rs:1751:22
    0: _napi_register_module_v1
    1: <unknown>
    2: _napi_register_module_v1
    3: <unknown>
    4: <unknown>
    5: _napi_register_module_v1
    6: _napi_register_module_v1
    7: <unknown>
    8: _napi_register_module_v1
    9: __pthread_deallocate
  ] {
    code: 'GenericFailure'
  }
  ERROR: "build:rspack" exited with 2.
  ```

  </p>

  <p class="inline-block">

  [#1179](https://github.com/web-infra-dev/rspack/issues/1179)
  ```bash
  Panic: PanicInfo { payload: Any { .. }, message: Some(assertion failed: start <= end), location: Location { file: "/Users/bytedance/.cargo/registry/src/[github.com](http://github.com/)-1ecc6299db9ec823/swc_common-0.29.13/src/[input.rs](http://input.rs/)", line: 31, col: 9 }, can_unwind: true }
  Backtrace:    0: _napi_register_module_v1
    1: <unknown>
    2: _napi_register_module_v1
    3: <unknown>
    4: _napi_register_module_v1
    5: __pthread_deallocate

  npm ERR! code ELIFECYCLE
  npm ERR! errno 1
  npm ERR! @byted-shadow/platform@0.0.2 start: `npm run clean && NODE_ENV=development rspack server --config rspack.config.js`
  npm ERR! Exit status 1
  ```
  </p>

  <p class="inline-block">

  [#2585](https://github.com/web-infra-dev/rspack/issues/2585)

  ```bash
  [Error: error[internal]: Failed to resolve render_manifest results: JoinError::Panic(Id(123), ...).
  Raw: panicked at 'Failed to resolve render_manifest results: JoinError::Panic(Id(123), ...)', /build/crates/rspack_core/src/compiler/compilation.rs:926:8
    0: <unknown>
    1: <unknown>
    2: <unknown>
    3: <unknown>
    4: start_thread
    5: clone
  ] {
    code: 'GenericFailure'
  }
  undefined
  ```
  </p>

</div>

<!--
The final major issue is debugging. In order to reduce installation size, most NAPI-RS binaries have debug symbols removed during release. The screenshot here is from issues on the official rspack repository, which shows that when issues occur, the error stack cannot be displayed at all, making it much more difficult to locate and fix the problem.
-->

---

# Debugging

<br/>

NAPI-RS will support build optimized binary with split debug symbols and download them via `@napi-rs/cli` while debugging installed NAPI-RS packages in the next major release.

<!--
To solve this problem, NAPI-RS will provide a debug symbols download feature in the next major release to facilitate the downloading of debug symbols for printing error messages that contain source code when errors occur.
-->

---

# Conclusion

- NAPI-RS is very easy to use and provides an end-to-end solution from development to deployment.
- NAPI-RS can improve Node.js performance progressively.
- Not all scenarios are suitable for developing with NAPI-RS.

# Learn More

Please star and give it a try: https://github.com/napi-rs/napi-rs

[Documentations](https://napi.rs) · [GitHub](https://github.com/napi-rs/napi-rs) · [Showcases](https://napi.rs/docs/ecosystem/snappy)
