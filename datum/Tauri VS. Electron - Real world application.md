In this article I'm going to compare Electron and Tauri using a real world app: [Authme](https://authme.levminer.com/). Authme is a Simple cross-platform two-factor authenticator app for desktop. It's not a big app and not very complex, perfect for a little comparison. You can take a look at the source code of the Electron app on [GitHub](https://github.com/Levminer/authme) and the Tauri app is also on [GitHub](https://github.com/Levminer/authme-v4). My goal it that the Tauri app eventually replaces the Electron one.

## Electron app architecture

What is Electron?

> Electron is a framework for creating native applications with web technologies like JavaScript, HTML, and CSS. It takes care of the hard parts so you can focus on the core of your application. If you can build a website, you can build a desktop app. - electronjs.org

App architecture

The Electron app is written in plain-old HTML and JavaScript. For styling I'm using TailwindCSS and some custom CSS.

## Tauri app architecture

What's Tauri?

> Tauri is a framework for building tiny, blazingly fast binaries for all major desktop platforms. Developers can integrate any front-end framework that compiles to HTML, JS and CSS for building their user interface. The backend of the application is a rust-sourced binary with an API that the front-end can interact with. - Tauri GitHub

Architecture

My Tauri app is using more modern stack. The build tool is Parcel, the framework is Svelte, and of course TypeScript instead of JavaScript. The styling is done with TailwindCSS.

## Comparison

This is not a head-to-head comparison, but the app is basically the same.

Key points:

1.  Bundle
2.  Startup time
3.  Performance
4.  App backend
5.  Rendering your app
6.  Security
7.  Auto update
8.  Developer experience

## 1\. Bundle

We have a clear winner here: Tauri

The Tauri app installer is around ~2,5MB (!!!), while the Electron app installer is around ~85MB.

Full bundle size: ![Tauri bundle](https://cdn.levminer.com/blog/tauri-vs-electron/tauri-bundle.png) ![Electron bundle](https://cdn.levminer.com/blog/tauri-vs-electron/electron-bundle.png)

One major advantage that Tauri has: The app is compiled to a binary, which means you have to be an expert at reverse engineering to be able to de-compile the app. With Electron you can unpack the app with a simple [NPM command](https://medium.com/how-to-electron/how-to-get-source-code-of-any-electron-application-cbb5c7726c37).

And if your user has the correct runtime for the webview used by Tauri you can just send them a single executable, they don't have to install anything.

## 2\. Startup time

This is not a scientific test but running the apps and timing the startup yields a clear winner:

-   Tauri: ~ 2 seconds
-   Electron: ~ 4 seconds

## 3\. Performance

This is not a scientific test again, just a rough comparison. These tests are from my PC: i5-4570 CPU, 16GB RAM, GTX 1660 and Windows 11

Tauri (Windows)

| Test | CPU | RAM | GPU |
| --- | --- | --- | --- |
| Idle | 1% | ~ 80MB | 0% |

Electron (Windows)

| Test | CPU | RAM | GPU |
| --- | --- | --- | --- |
| Idle | 1% | ~ 120MB | 0% |

Honestly I expected more from Tauri, sure it uses less RAM, but not by much. Let's take a look at what's up on the Linux side.

Tauri ![Tauri](https://cdn.levminer.com/blog/tauri-vs-electron/tauri-linux.png)

Electron ![Electron](https://cdn.levminer.com/blog/tauri-vs-electron/electron-linux.png)

Wow, thats a big win for Tauri!

## 4\. App backend

Now here comes one of the disadvantages of Tauri (or its biggest advantage, I guess it depends on you). In your Electron app you write your app backend in JavaScript, because Electron uses the Node.js runtime. Tauri on the other hand is written in Rust. Now if you know Rust your probably happy, but if you have to learn Rust from scratch (like me) you are going to face some issues.

You have to rewrite your app's backend in Rust, so I think the winner in Electron here. For now. Alternative bindings for your backend are on Tauri's roadmap, like Python, C++, or Deno. Personally I'm excited for the Deno integration so I can write my app backend in JavaScript/TypeScript again.

## 5\. Rendering your app

Electron uses Chromium under the hood so your user sees the same on Windows, Linux and macOS. Tauri on the other hand uses the system webview: Edge Webview2 (Chromium) on Windows, WebKitGTK on Linux and WebKit on macOS. Now here comes the bad part, if you are a web developer you know that Safari (Based on WebKit) is always behind a step from every web browser. Just check out [Can I Use](https://caniuse.com/). There is always a bug that you are not seeing from Chrome, only your dear Safari users. The same issues exists in Tauri, and you can't do anything against it, you have include polyfills. The winner has to be Electron here.

One issue I had while developing Tauri is that my CSS bundle didn't include `-webkit` prefixes, so my app's CSS was very buggy.

## 6\. Security

Tauri is very secure by default, on the other hand I can't say the same about Electron. Tauri has many [security](https://tauri.app/about/security) features built-in by default. You can even explicitly enable or disable certain APIs. With Electron you have full access to Node APIs, so a hacker could easily exploit the very powerful Node APIs. This not the case with Tauri, you have to explicitly expose the Rust functions.

## 7\. Auto update

> ![Auto update comic](https://imgs.xkcd.com/comics/update.png) xkcd.com

Shipping an app without auto update in 2022 is a no go. If your user has to manually download every update I don't think they are going to be happy. Both Electron and Tauri has a built-in auto updater, but the Tauri one is so much simpler. In Electron I think most people use [electron-updater](https://www.npmjs.com/package/electron-updater). In Tauri you can use the [built in](https://tauri.app/v1/guides/distribution/updater) one, the one downside is you have to maintain your own [update server](https://github.com/KilleenCode/tauri-update-cloudflare) or you could use a simple [JSON file](https://tauri.app/v1/guides/distribution/updater#update-file-json-format) you have to update manually. Electron updater pulls the binaries form GitHub releases, which is way more convenient.

## 8\. Developer experience

In Tauri you get the whole package just by installing the Tauri CLI: Hot reload, building your app for all platforms, generating app icons. Electron gives you nothing, just the framework itself. You have setup hot reloading, bundling your app etc... Tauri takes care of everything for you. And here comes the best: Tauri is compatible with every web framework on earth. It just expects a localhost url or a folder where all your bundled code lives.

## Summary

Electron is being replaced? Yes, Tauri is way better, but it still misses a lot. In a couple of years I'm sure the Tauri team will catch app to Electron. The things I'm excited for: Deno as the backend, better auto update and iOS/Android support.

Thanks for reading my first article! I'm not a native english speaker, I'm sorry for any mistakes.