# Opening (and closing!) a window

Let's start a new project:

`amethyst new pong`

If you run this project with `cargo run`, you'll end up with a window titled
"pong" that renders a really delightful shade of green. Press `Esc` to quit. If
you're having trouble getting the project to run, double check the
[Getting Started][gs] guide.

We've opened and closed a window, so we're basically done! But let's write this
functionality ourselves so we're sure we know what's going on.

**In `src` there's a `main.rs` file. Delete everything in that file, then add these imports:**

```rust,no_run,noplaypen
extern crate amethyst;

use amethyst::prelude::*;
use amethyst::renderer::{DisplayConfig, DrawSprite, Event, Pipeline,
                         RenderBundle, Stage, VirtualKeyCode};
```

We'll be learning more about these as we go through this tutorial. The prelude
includes the basic (and most important) types like `Application`, `World`, and
`State`.

Now we create our core game struct:

```rust,no_run,noplaypen
pub struct Pong;
```

We'll be implementing the [`State`][st] trait on this struct, which is used by
Amethyst's state machine to start, stop, and update the game. But for now we'll
just implement two methods:

```rust,no_run,noplaypen
# extern crate amethyst;
# use amethyst::input::{is_close_requested, is_key_down};
# use amethyst::prelude::*;
# use amethyst::renderer::{DisplayConfig, DrawFlat, Pipeline,
#                          PosTex, RenderBundle, Stage};
# struct Pong;
impl<'a, 'b> SimpleState<'a, 'b> for Pong {
}
```

The `SimpleState` already implements a bunch of stuff for us, like the `update` 
and `handle_event` methods that you would have to implement yourself were you 
using just a regular `State`. In particular, the default implementation for
`handle_event` returns `Trans::Quit` when a close signal is received
from your operating system, like when you press the close button in your graphical
environment. This allows the application to quit as needed. The default 
implementation for `update` then just returns `Trans::None`, signifying that
nothing is supposed to happen.

Now that we know we can quit, let's add some code to actually get things
started! We'll start with our `main` function, and we'll have it return a
`Result` so that we can use `?`. This will allow us to automatically exit
if any errors occur during setup.

```rust,no_run,noplaypen
# extern crate amethyst;
# use amethyst::prelude::*;
fn main() -> amethyst::Result<()> {

    // We'll put the rest of the code here.

    Ok(())
}
```

Inside `main()` we first start the amethyst logger with a default `LoggerConfig`
so we can see errors, warnings and debug messages while the program is running.

```rust,no_run,noplaypen
# extern crate amethyst;
# use amethyst::prelude::*;
# fn main() -> amethyst::Result<()> {
amethyst::start_logger(Default::default());
# Ok(())
# }
```

After the logger is started, we define a path for our display_config.ron file
and load it.

```rust,no_run,noplaypen
# extern crate amethyst;
# use amethyst::prelude::*;
# use amethyst::renderer::DisplayConfig;
# fn main() {
let path = "./resources/display_config.ron";

let config = DisplayConfig::load(&path);
# }
```

This .ron file was automatically generated by `amethyst new`. If you didn't use
`amethyst new`, now would be a good time to create this config file inside a
folder named resources. If you already have this file, we have some changes to
make, anyway:

```rust,ignore
(
  title: "Pong!",
  dimensions: Some((500, 500)),
  max_dimensions: None,
  min_dimensions: None,
  fullscreen: false,
  multisampling: 0,
  visibility: true,
  vsync: true,
)
```

This will set the default window dimensions to 500 x 500, and make the title bar
say "Pong!" instead of the sad, lowercase default of "pong".

Now, back inside our `main()` function in main.rs, let's copy and paste some
rendering code so we can keep moving. We'll cover rendering in more depth later
in this tutorial.

```rust,no_run,noplaypen
# extern crate amethyst;
# use amethyst::renderer::{Pipeline, DrawFlat, PosTex, Stage, DrawSprite};
# fn main() {
let pipe = Pipeline::build().with_stage(
    Stage::with_backbuffer()
        .clear_target([0.0, 0.0, 0.0, 1.0], 1.0)
        .with_pass(DrawSprite::new()),
);
# }
```

The important thing to know right now is that this renders a black background.
If you want a different color you can tweak the RGBA values inside the
`.clear_target` method. Values range from 0.0 to 1.0, so to get that cool green
color we started with back, for instance, you can try
`[0.00196, 0.23726, 0.21765, 1.0]`.

Now let's pack everything up and run it:

```rust,no_run,noplaypen
# extern crate amethyst;
# use amethyst::prelude::*;
# use amethyst::renderer::{DisplayConfig, DrawSprite, Pipeline,
#                        RenderBundle, Stage};
# fn main() -> amethyst::Result<()> {
# let path = "./resources/display_config.ron";
# let config = DisplayConfig::load(&path);
# let pipe = Pipeline::build().with_stage(Stage::with_backbuffer()
#       .clear_target([0.0, 0.0, 0.0, 1.0], 1.0)
#       .with_pass(DrawSprite::new()),
# );
# struct Pong;
# impl<'a, 'b> SimpleState<'a,'b> for Pong { }
let game_data = GameDataBuilder::default()
    .with_bundle(RenderBundle::new(pipe, Some(config)).with_sprite_sheet_processor())?;
let mut game = Application::new("./", Pong, game_data)?;
game.run();
# Ok(())
# }
```

We've discovered Amethyst's root object: [Application][ap]. It binds the OS
event loop, state machines, timers and other core components in a central place.
Here we're creating a new `RenderBundle`, adding the `Pipeline` we created,
along with our config, and building. There is also a helper function
`with_basic_renderer` on `GameDataBuilder` that you can use to create your
`Pipeline` and `RenderBundle`, that performs most of the actions above. In the
full `pong` example in the `Amethyst` repository that function is used instead.

Then we call `.run()` on `game` which begins the gameloop. The game will
continue to run until our `State` returns `Trans::Quit`, or when all states have
been popped off the state machine's stack.

Finally, let's create a `texture` folder in the root of the project. This
will contain the [spritesheet texture][ss] `pong_spritesheet.png` we will need
to render the elements of the game. To go with it, we need a file describing
where the sprites are on the sheet. Let's create, right next to it, a file
called `pong_spritesheet.ron`. It will contain the following sprite sheet
definition:

```text,ignore
(
    spritesheet_width: 8.0,
    spritesheet_height: 16.0,
    sprites: [
        (
            x: 0.0,
            y: 0.0,
            width: 4.0,
            height: 16.0,
        ),
        (
            x: 4.0,
            y: 0.0,
            width: 4.0,
            height: 4.0,
        ),
    ],
)
```

Success! Now we should be able to compile and run this code and get a window.
It should look something like this:

![Step one](../images/pong_tutorial/pong_01.png)


[st]: https://www.amethyst.rs/doc/master/doc/amethyst/trait.State.html
[ap]: https://www.amethyst.rs/doc/master/doc/amethyst/struct.Application.html
[gs]: ../getting-started.html
[ss]: ../images/pong_tutorial/pong_spritesheet.png
