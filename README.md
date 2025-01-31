# Tauri Plugin clipboard

> A Tauri plugin for clipboard IO. Support both text and image.
>
> The reason I built this plugin is becasue official Tauri API only supports clipboard with text, not image. So you can still use the official API for text.


> One thing about clipboard image io is that the input and output is always base64 png string.

## Installation

> If you are installing from npm and crate.io package registry, make sure the versions for both packages are the same, otherwise, the API may not match.

Crate: https://crates.io/crates/tauri-plugin-clipboard

`cargo add tauri-plugin-clipboard` to add the package.

Or add the following to your `Cargo.toml` for the latest unpublished version (not recommanded).

```toml
tauri-plugin-clipboard = { git = "https://github.com/CrossCopy/tauri-plugin-clipboard", tag = "v0.3.2" }
```

Run the following to install JavaScript/TypeScript API package.

```bash
npm i tauri-plugin-clipboard-api
# npm add https://github.com/CrossCopy/tauri-plugin-clipboard # or this for latest unpublished version (not recommended)
```

In `main.rs`, add the following to your `tauri::Builder`:

```rust
tauri::Builder::default()
    .plugin(tauri_plugin_clipboard::init())
    .run(tauri::generate_context!())
    .expect("failed to run app");
```

Read more in the [official doc](https://tauri.app/v1/guides/features/plugin/#using-a-plugin) about how to use.

## Example

```bash
npm run build
cd examples/svelte-app
npm run tauri dev
# there are a few buttons you can click to test the clipboard plugin
```

See [App.svelte](examples/svelte-app/src/App.svelte) for an example of how to use the plugin in JS/TS.

It works the same with other frontend frameworks like Vue, React, etc.


## Sample Usage (TypeScript API)

```ts
import {
    writeText,
    readText,
    readImage,
    writeImage,
} from "tauri-plugin-clipboard-api";
readText().then((text) => {
    // TODO
});

writeText("huakun zui shuai").then(() => {});

readImage()
.then((base64Img) => {
    imageStr = `data:image/png;base64, ${base64Img}`
})
.catch((err) => {
    alert(err);
});

writeImage(sample_base64_image).then(() => {
    // TODO
});
```

### Sample Usage (Rust API)

```rust
use tauri::Manager;
use tauri_plugin_clipboard::ManagerExt;

fn main() {
  tauri::Builder::default()
    .plugin(tauri_plugin_clipboard::init())
    .setup(|app| {
        let app_handle = app.app_handle();
        match app_handle.clipboard().read_text() {
            Ok(result) => {
                println!("content: {}", result);
            },
            Err(e) => print!("err is \"{}\"\n", e), // TODO: log
        };
        Ok(())
    })
    .build(tauri::generate_context!())
    .expect("failed to run app");
}
```

### Sample Listener Usage

We use Tauri's event system. Start a listener with Tauri's `listen()` function to start listening for event, and call `listenImage()` and `listenText()` to listen for clipboard update. When clipboard is updated, event will be emitted.

The following example is in svelte.

```ts
import { emit, listen, type UnlistenFn } from "@tauri-apps/api/event";
import {
    TEXT_CHANGED,
    IMAGE_CHANGED,
    listenText,
    listenImage,
    listenToClipboard,
} from "tauri-plugin-clipboard-api";

let listenTextContent = "";
let listenImageContent = "";
let tauriTextUnlisten: UnlistenFn;
let tauriImageUnlisten: UnlistenFn;
let listenToClipboardUnlisten: UnlistenFn;
let textUnlisten: () => void;
let imageUnlisten: () => void;

export async function startListening() {
    tauriTextUnlisten = await listen(TEXT_CHANGED, (event) => {
        console.log(event);
        listenTextContent = (event.payload as any).value;
    });
    tauriImageUnlisten = await listen(IMAGE_CHANGED, (event) => {
        console.log(event);
        listenImageContent = (event.payload as any).value;
    });
    // ==================== Choose from one of the two options below ====================
    // listen separately with JavaScript by comparing previous value and current value
    imageUnlisten = listenImage();
    textUnlisten = listenText();
    // or
    // listen on both text and image using clipboard-master crate (recommended, should be more efficient)
    listenToClipboardUnlisten = await listenToClipboard(); // start listener for both text and image
    // ==================================================================================
}

function stopListening() {
    imageUnlisten();
    textUnlisten();
    tauriTextUnlisten();
    tauriImageUnlisten();
    listenToClipboardUnlisten();
}

onMount(() => {
    startListening();
});

onDestroy(() => {
    stopListening();
});
```

The base64 image string can be converted to `Uint8Array` and written to file system using tauri's fs API. 

```ts
import { writeBinaryFile, BaseDirectory } from "@tauri-apps/api/fs";

writeBinaryFile('tmp/avatar.png', new Uint8Array(atob(base64Img).split('').map(char => char.charCodeAt(0))), { dir: BaseDirectory.Cache })
```
