# egui-text-animation

`egui-text-animation` provides simple text animation utilities for the [egui](https://github.com/emilk/egui) library.

This library offers the `TextAnimator` struct to create various text animations. It's designed to be easy to integrate
into your existing egui applications. The core of the library is the `TextAnimator` struct, which manages the state of
the animation, and the `AnimationType` enum, which determines the type of animation to perform.

## Features

* **Fade-in Animation:** Animate text to gradually appear, character by character. See `AnimationType::FadeIn`.
* **Typewriter Animation:** Animate text to appear as if it's being typed. See `AnimationType::Typewriter`.
* **Hacker Animation:** Animate text to appear as if it's being decoded. See `AnimationType::Hacker`.
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
egui-text-animation = "0.1.0" # Replace with the actual version (or use a git dependency)
eframe = "0.31.0" # Or the latest version that suits your needs.
```

Replace `"0.1.0"` with the actual released version (if any), you also can use a
git dependency:

```toml
[dependencies]
egui-text-animation = { git = "https://github.com/dest4590/egui-text-animation" } # Replace with your repo URL
eframe = "0.31.0"
```

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
        "Text Animation Example",
        options,
        Box::new(|_cc| Ok(Box::<MyApp>::default())),
    )
}

struct MyApp {
    fade_animator: TextAnimator,
    typewriter_animator: TextAnimator,
    hacker_animator: TextAnimator,
    animation_running: bool,
    speed: f32,
    selected_animation: AnimationType,
}

impl Default for MyApp {
    fn default() -> Self {
        Self {
            fade_animator: TextAnimator::new(
                "Hello, Fade In!",
                egui::FontId::new(18.0, egui::FontFamily::Proportional),
                egui::Color32::WHITE,
                0.5,
                AnimationType::FadeIn,
            ),

            typewriter_animator: TextAnimator::new(
                "Hello, Typewriter!",
                egui::FontId::new(18.0, egui::FontFamily::Proportional),
                egui::Color32::WHITE,
                0.5,
                AnimationType::Typewriter,
            ),
            hacker_animator: TextAnimator::new(
                "Access Granted",
                egui::FontId::new(18.0, egui::FontFamily::Proportional),
                egui::Color32::GREEN,
                2.0,
                AnimationType::Hacker,
            ),

            animation_running: false,
            speed: 2.0,
            selected_animation: AnimationType::FadeIn,
        }
    }
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            // --- Animation Selection ---
            ui.horizontal(|ui| {
                ui.label("Select Animation:");
                ui.radio_value(
                    &mut self.selected_animation,
                    AnimationType::FadeIn,
                    "Fade In",
                );
                ui.radio_value(
                    &mut self.selected_animation,
                    AnimationType::Typewriter,
                    "Typewriter",
                );
                ui.radio_value(
                    &mut self.selected_animation,
                    AnimationType::Hacker,
                    "Hacker",
                );
            });

            // --- Start/Stop Buttons ---
            ui.horizontal(|ui| {
                if ui.button("Start Animation").clicked() {
                    self.animation_running = true;
                    match self.selected_animation {
                        AnimationType::FadeIn => self.fade_animator.reset(),
                        AnimationType::Typewriter => self.typewriter_animator.reset(),
                        AnimationType::Hacker => self.hacker_animator.reset(),
                    }
                }
                if ui.button("Stop Animation").clicked() {
                    self.animation_running = false;
                }
            });

            // --- Speed Control ---
            ui.horizontal_wrapped(|ui| {
                ui.label("Speed:");
                if ui
                    .add(egui::Slider::new(&mut self.speed, 0.1..=10.0))
                    .changed()
                {
                    self.fade_animator.set_speed(self.speed);
                    self.typewriter_animator.set_speed(self.speed);
                    self.hacker_animator.set_speed(self.speed);
                }
            });

            // --- Font Size Control ---
            ui.horizontal(|ui| {
                ui.label("Font Size:");
                let mut font_size = self.fade_animator.font.size;
                if ui
                    .add(egui::Slider::new(&mut font_size, 1.0..=100.0))
                    .changed()
                {
                    self.fade_animator.font.size = font_size;
                    self.typewriter_animator.font.size = font_size;
                    self.hacker_animator.font.size = font_size;
                }
            });

            // --- Controlled Animation ---
            if self.animation_running {
                let (animator, finished) = match self.selected_animation {
                    AnimationType::FadeIn => {
                        self.fade_animator.process_animation(ctx);
                        let finished = self.fade_animator.is_animation_finished();
                        (&mut self.fade_animator, finished)
                    }
                    AnimationType::Typewriter => {
                        self.typewriter_animator.process_animation(ctx);
                        let finished = self.typewriter_animator.is_animation_finished();
                        (&mut self.typewriter_animator, finished)
                    }
                    AnimationType::Hacker => {
                        self.hacker_animator.process_animation(ctx);
                        let finished = self.hacker_animator.is_animation_finished();
                        (&mut self.hacker_animator, finished)
                    }
                };
                animator.render(ui);

                if !finished {
                    ctx.request_repaint();
                }
            } else {
                match self.selected_animation {
                    AnimationType::FadeIn => self.fade_animator.render(ui),
                    AnimationType::Typewriter => self.typewriter_animator.render(ui),
                    AnimationType::Hacker => self.hacker_animator.render(ui),
                };
            }

            if self.animation_running
                && match self.selected_animation {
                AnimationType::FadeIn => self.fade_animator.is_animation_finished(),
                AnimationType::Typewriter => self.typewriter_animator.is_animation_finished(),
                AnimationType::Hacker => self.hacker_animator.is_animation_finished(),
            }
            {
                ui.label("Animation finished!");
            }
        });
    }
}

```

## Animation Types

The `AnimationType` enum provides the following animation types:

* `AnimationType::FadeIn`: Characters gradually fade in from transparent to fully opaque.
* `AnimationType::Typewriter`: Characters appear one by one, simulating a typewriter effect.
* `AnimationType::Hacker`: Characters cycle through random characters before settling on the final character.

## API Reference

See the [docs.rs documentation](<https://docs.rs/egui-text-animation>) for a complete API reference.  (Note: This link
will only work *after* you've published your crate to crates.io and docs.rs has built the documentation. Before then,
you can generate local documentation with `cargo doc --open`.)

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
