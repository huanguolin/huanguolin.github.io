---
title: tauri 使用经验之 —— state
description: "最近我用 tauri 做了一个桌面工具，淌了一些 tauri 使用经验。今天简单谈下 state 的使用。"
date: 2024-03-24T11:37:00
thumb: "tauri-state.jpeg"
tags:
    - tauri
    - state
---

[tauri](https://tauri.app/) 是一个类似 [electron](https://www.electronjs.org/) 的跨平台（支持 windows/macOS/linux）桌面程序开发平台。都是用前端技术来做桌面UI，不同的是 tauri 没有像 electron 一样打包一个 chromium，而是用系统自带的 webview, 这样打包体积大大减小；另外 tauri 后端使用 rust 语言替代了 electron 的 node.js。这个选择各有利弊，用 rust 有更好的安全、性能保障，但是陡峭的学习曲线可能让新手望而却步，好消息是，大部分时候可能并不需要写 rust, tauri 提供了很多 api 来访问操作系统的资源（比如：fs, shell, process 等，见[这里](https://tauri.app/v1/api/js/)）。不过取决于需求，比如我就没有那么幸运。

我要用它做了一个桌面工具，需要访问数据库并和一个 .net console 程序通信。所以我不得不写一点 rust 代码。今天我想简单谈下 state 的使用，特别是官网没有直接说的用法。

tauri 提供了 [command](https://tauri.app/v1/guides/features/command/) 以便于从前端调用后端“方法”。而我需要保存一些程序状态，并在不同 command 之间传递。官方文档对 state 讲的很粗略（[这里](https://tauri.app/v1/guides/features/command/#accessing-managed-state)），它仅仅给了如下示例代码：
```rust
struct MyState(String);

// 在命令的参数上获取 state
#[tauri::command]
fn my_custom_command(state: tauri::State<MyState>) {
  assert_eq!(state.0 == "some state value", true);
}

fn main() {
  tauri::Builder::default()
    .manage(MyState("some state value".into())) // 在这里注入 state
    .invoke_handler(tauri::generate_handler![my_custom_command])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

我的需求是，执行一个 A command 时，能存入一个东西（比如 url）到 state 中，然后在后续执行 B command 时能从 state 中取出之前存的 url。我刚开始的想法是，先定义一个结构体，里面可以放这个 url。初始时里面是空的，执行 A command 时修改它，执行 B command 时取出。嗯😐，想法完全没有问题，可是对 rust 不熟的我却不知到怎么写出来。我尝试了用 Option, Arc, Mutex ... 各种我不熟悉的数据结构交织在一起，但也没有消灭编译器给的所有红色警告……🤯

就这么简单需求，怎么就这么难呢？！

后来我去看了下 manage 的 [API 文档](https://docs.rs/tauri/1.6.1/tauri/trait.Manager.html#method.manage), 才发现它的用法很灵活。比如，manage 可在不同的位置，不同的时期，多次调用，还有一个 state API 能和 manage 配对使用来取出状态量。文档中就有如下代码：
```rust
// ... 省略其他无关代码
tauri::Builder::default()
  .setup(|app| {
    // 多次使用 manage
    app.manage(MyInt(0));
    app.manage(MyString("tauri".into()));

    // 取出 state
    let int = app.state::<MyInt>();
    assert_eq!(int.0, 0);
    let val: State<MyString> = app.state();
    assert_eq!(val.0, "tauri");
    // .... 省略其他无关代码
    Ok(())
  })
```

mange, state api 都能在 AppHandle 中访问。官网中有如何在 [command 中如何访问 AppHandle](https://tauri.app/v1/guides/features/command#accessing-an-apphandle-in-commands) 的示例：
```rust
#[tauri::command]
async fn my_custom_command(app_handle: tauri::AppHandle) {
  let app_dir = app_handle.path_resolver().app_dir();
  use tauri::GlobalShortcutManager;
  app_handle.global_shortcut_manager().register("CTRL + U", move || {});
}
```

至此我的需求解决了，简化代码如下：
```rust
#[tauri::command]
fn a_command(app_handle: tauri::AppHandle, front_end_url: String) {
  app_handle.manage(front_end_url);
}

#[tauri::command]
fn b_command(app_handle: tauri::AppHandle) {
  let url: State<String> = app_handle.state();
  // ...
}
```

哈哈是不是很简单！不过要注意的是，要取出什么，取决于前面定义的类型，且获取前确保 state 已经存入。

好了，今天就到这里了。谢谢！

