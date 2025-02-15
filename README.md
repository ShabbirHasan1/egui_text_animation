# egui-text-animation

`egui-text-animation` provides simple text animation utilities for the [egui](https://github.com/emilk/egui) library.

This library offers the `TextAnimator` struct to create various text animations. It's designed to be easy to integrate
into your existing egui applications. The core of the library is the `TextAnimator` struct, which manages the state of
the animation, and the `AnimationType` enum, which determines the type of animation to perform.

## Features

* **Fade-in Animation:** Animate text to gradually appear, character by character. See `AnimationType::FadeIn`.
* **Typewriter Animation:** Animate text to appear as if it's being typed. See `AnimationType::Typewriter`.
* **Customizable Speed:** Control the speed of the animation with `TextAnimator::set_speed`.
* **Easy Integration:** Simply create a `TextAnimator`, call `TextAnimator::process_animation` each frame, and then
  render with `TextAnimator::render`.
* **Automatic Repainting:** Call `ctx.request_repaint()` inside your update loop to ensure smooth animation.
* **Animation Control:** You can check if the animation is finished with `TextAnimator::is_animation_finished` and reset
  it with `TextAnimator::reset`.

## Installation

Add the following to your `Cargo.toml`:

```toml
[dependencies]
egui-text-animation = "0.1.0" # Replace with the actual version
eframe = "0.31.0" # Or the latest version that suits your needs.
```

Replace `"0.1.0"` with actual released version (if any).

## Example

```rust
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

use eframe::egui;
use egui_text_animation::{AnimationType, TextAnimator};

fn main() -> eframe::Result<()> {
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default().with_inner_size([480.0, 320.0]),
        ..Default::default()
    };
    eframe::run_native(
        "My egui App",
        options,
        Box::new(|_cc| Box::<MyApp>::default()),
    )
}

struct MyApp {
    fade_animator: TextAnimator,
    typewriter_animator: TextAnimator,
    animation_running: bool,
}

impl Default for MyApp {
    fn default() -> Self {
        Self {
            fade_animator: TextAnimator::new(
                "Fade-in!",
                egui::FontId::new(18.0, egui::FontFamily::Proportional),
                egui::Color32::WHITE,
                0.5,
                AnimationType::FadeIn,
            ),
            typewriter_animator: TextAnimator::new(
                "Typewriter...",
                egui::FontId::new(18.0, egui::FontFamily::Proportional),
                egui::Color32::WHITE,
                0.5,
                AnimationType::Typewriter
            ),

            animation_running: false,
        }
    }
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            if self.animation_running {
                self.fade_animator.process_animation(ctx);
                self.typewriter_animator.process_animation(ctx);
                ctx.request_repaint(); // Request repaint for continuous animation
            }

            if ui.button("Start Animation").clicked() {
                self.animation_running = true;
                self.fade_animator.reset();
                self.typewriter_animator.reset();
            }

            ui.add_space(10.0);
            self.fade_animator.render(ui);
            ui.add_space(10.0);
            self.typewriter_animator.render(ui);
        });
    }
}

```

## Animation Types

The `AnimationType` enum provides the following animation types:

* `AnimationType::FadeIn`: Characters gradually fade in from transparent to fully opaque.
* `AnimationType::Typewriter`: Characters appear one by one, simulating a typewriter effect.

## API Reference

See the [docs.rs documentation](<https://docs.rs/egui-text-animation>) for a complete API reference.  (Note: This link
will only work *after* you publish the crate to crates.io.) If you haven't published, you can generate local
documentation with `cargo doc --open`.

## Notes

* The `unstable_dt` value from `ctx.input(|i| i.unstable_dt)` is used for frame-independent animation timing. This
  ensures the animation runs at the correct speed regardless of the application's frame rate.
* It's crucial to call `ctx.request_repaint()` during the animation to ensure that egui re-renders the UI, thus updating
  the animation.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details. (You'll need to create a
LICENSE file and put the MIT license text in it.)