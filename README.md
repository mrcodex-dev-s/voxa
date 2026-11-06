# Voxa

**Build 3D websites with pure Python.**

Write pages in Python. Voxa turns them into a Three.js website with file-based routing, click handlers, physics and a dev server with hot reload. No JavaScript required.

```python
# pages/index.py
from voxa import Scene, Box, Sphere, Light, Bounce

def toggle(obj):                      # plain Python, runs on every click
    obj.color = "gold" if obj.color != "gold" else "skyblue"

def page():
    scene = Scene(background="#111827")
    scene.add(Box(color="tomato", link="/about", hover_scale=1.15).animate(Bounce()))
    scene.add(Sphere(position=(2, 0, 0), color="skyblue", on_click=toggle))
    scene.add(Light.ambient())
    scene.add(Light.point(position=(5, 5, 5)))
    return scene
```

> **Status:** early release (v0.9). The API may change.

## Features

- **File-based routing:** every file in `pages/` is a page, including dynamic routes like `pages/blog/[slug].py`
- **Primitives and models:** `Box`, `Sphere`, and `.glb` / glTF files with `Model`
- **Materials and textures:** images, metal, glass, glow, wireframe, plus shadows and studio reflections
- **Text and buttons:** `Text("Hello")` in any language, including Bangla, with optional button background
- **Animations:** `Bounce`, `Move`, `ColorShift`, `Pulse`, plus `spin`
- **Interaction in Python:** `on_click` handlers, hover effects, clickable links between pages
- **Physics:** gravity, collisions and bouncing with one argument
- **Dev server:** `voxa dev` with automatic browser reload
- **Static export:** `voxa build` makes plain files you can host for free (GitHub Pages, Netlify, Cloudflare Pages)
- **Works on a phone:** touch drag to orbit, tap to click

## Installation

Voxa is not on PyPI yet, so install from source:

```bash
git clone https://github.com/mrcodex-dev-s/voxa.git
cd voxa
pip install -e .
```

Requires Python 3.9 or newer.

## Quick start

```bash
voxa new myproject
cd myproject
voxa dev
```

Open <http://127.0.0.1:8000>. Edit any file in `pages/` and the browser reloads on its own.

> The browser loads Three.js (and the physics engine, when used) from a CDN, so viewing a site needs an internet connection.

## Project layout

```
myproject/
├── pages/
│   ├── index.py          # /
│   ├── about.py          # /about
│   ├── physics.py        # /physics
│   └── blog/
│       └── [slug].py     # /blog/anything
└── assets/               # .glb models and other static files, served at /assets
```

Every page file defines a `page()` function that returns a `Scene`. Files and folders that start with `_` are ignored.

An `app.py` with `@app.page("/path")` still works, and can be used together with `pages/`.

## Guide

### Pages and dynamic routes

```python
# pages/blog/[slug].py   ->   /blog/hello, /blog/world, ...
from voxa import Scene, Sphere

def page(slug):                       # the URL part arrives as an argument
    scene = Scene()
    scene.add(Sphere(color="skyblue", link="/"))
    return scene
```

### Scenes and lights

```python
scene = Scene(background="#000000", camera_position=(0, 1.5, 5), gravity=-9.8)
scene.add(Light.ambient(intensity=0.4))
scene.add(Light.point(position=(5, 5, 5), intensity=1.0))
```

If you add no lights, Voxa adds a default ambient and point light.

### Objects

```python
Box(size=(1, 1, 1), color="tomato", position=(0, 0, 0), rotation=(0, 0, 0), spin=1.0)
Sphere(radius=0.5, color="skyblue", position=(2, 0, 0))
Model("duck.glb", scale=1.5, spin=0.5)      # file lives in assets/
```

`spin` is the rotation speed around the Y axis in radians per second. Every object also accepts `scale`.

### Materials and textures

```python
Sphere(color="#fbbf24", metalness=1.0, roughness=0.15)              # polished gold
Box(texture="wood.jpg", texture_repeat=(4, 4))                      # image from assets/, tiled
Box(color="#ef4444", opacity=0.5)                                   # glass-like
Sphere(color="#ffffff", emissive="#22d3ee")                         # glowing
Sphere(wireframe=True)                                              # just the edges
```

| Option           | Meaning                                                              |
| ---------------- | -------------------------------------------------------------------- |
| `texture`        | Image file in `assets/` (or a full URL). Stretched over the object   |
| `texture_repeat` | `(u, v)` number of times the image tiles                             |
| `metalness`      | `0` plastic or stone, `1` metal                                      |
| `roughness`      | `0` mirror-smooth, `1` matte                                         |
| `opacity`        | Below `1` is see-through                                             |
| `emissive`       | Glow colour                                                          |
| `wireframe`      | Draw only the edges                                                  |

With a texture, `color` tints the image. Leave it unset to show the image as it is.

Scene options:

```python
Scene(shadows=True)         # objects cast and receive shadows from the point light
Scene(environment=True)     # soft studio reflections; on automatically when an object has metalness > 0.3
```

Metal needs something to reflect, which is why `environment` switches on by itself for metallic objects. Click handlers can change material options too (for example `obj.metalness = 1`). Materials apply to `Box` and `Sphere`; `Model` and `Text` keep their own look.

### Text and buttons

```python
Text("Hello", size=0.7, position=(0, 2, 0), color="#f8fafc")
Text("বাংলা লেখা", size=0.4)                                   # any language the device has a font for
Text("Tap me", size=0.3, background="#2563eb", on_click=tap)    # looks and works like a button
```

Text always faces the camera. It supports `link`, `on_click`, `hover_scale` and animations, and a click handler can change it:

```python
def tap(obj):
    obj.taps = getattr(obj, "taps", 0) + 1
    obj.text = f"Taps: {obj.taps}"
```

Text does not support `hover_color` or `physics`.

### Animations

```python
Box().animate(Bounce(height=1.0, speed=3.0))
Sphere().animate(Move(to=(2, 0, 0), duration=2.0), Pulse(amount=0.3))
Box(color="red").animate(ColorShift(to="gold", duration=2.0))
```

| Animation    | What it does                     | Works on    |
| ------------ | -------------------------------- | ----------- |
| `Bounce`     | Hops up and down                 | all objects |
| `Move`       | Moves to a position and back     | all objects |
| `Pulse`      | Grows and shrinks                | all objects |
| `ColorShift` | Fades to another colour and back | Box, Sphere |

Objects with dynamic physics ignore `Bounce` and `Move`, because the physics engine controls their position.

### Links, clicks and hover

```python
Box(link="/about")                              # tap to open another page
Box(hover_scale=1.2, hover_color="gold")        # reacts when the pointer is over it

def grow(obj):                                  # runs in Python on every click
    obj.scale += 0.3

Box(on_click=grow)
```

A handler can also take the scene, so one click can change other objects:

```python
def handler(obj, scene):
    obj.color = "green"
    scene.objects[1].position = (0, 3, 0)
```

The decorator form works too:

```python
box = Box()

@box.clicked
def _(obj):
    obj.color = "gold"
```

Each visitor gets their own copy of the scene, so one person's clicks do not change what another sees. Handlers can change `color`, `position`, `rotation`, `scale`, `spin`, `animations` and the hover settings. If a handler adds or removes objects, the page reloads. Errors inside a handler are printed in the terminal and the server keeps running.

### Physics

```python
scene.add(Box(size=(10, 0.4, 6), position=(0, -2.5, 0), physics="static"))      # ground
scene.add(Sphere(radius=0.5, position=(0, 3, 0), physics=True, bounciness=0.6))  # falls and bounces
```

| Option       | Meaning                                                       |
| ------------ | ------------------------------------------------------------- |
| `physics`    | `True` or `"dynamic"` falls and collides, `"static"` stays put |
| `mass`       | Weight of a dynamic object (default `1.0`)                    |
| `bounciness` | `0` no bounce, `1` very bouncy (default `0.3`)                |

Setting `obj.position` from a click handler moves the physics body too, so a ball can be thrown back up. Physics is not supported on `Model` yet.

### 3D models

Put a `.glb` file in `assets/` and reference it by name:

```python
scene.add(Model("duck.glb"))
```

## CLI

| Command                   | Description                                      |
| ------------------------- | ------------------------------------------------ |
| `voxa new <name>`         | Create a project with example pages              |
| `voxa dev`                | Run the dev server with auto reload              |
| `voxa dev --port 9000`    | Use a different port                             |
| `voxa dev --host 0.0.0.0` | Make the site reachable from other devices       |
| `voxa build`              | Export the site as static files into `dist/`     |
| `voxa build --base /name` | Same, for a site hosted under `/name/`           |

## Deploying

```bash
voxa build                      # writes the site into dist/
python -m http.server -d dist   # preview it locally at http://localhost:8000
```

`dist/` is plain HTML, so any static host works. For a GitHub Pages **project site** (served at `username.github.io/name/`), build with the sub-path so links and models resolve:

```bash
voxa build --base /name
```

What works on a static site: pages, links, hover, animations, physics, text and models. What does not: `on_click` handlers, because they run in Python and need a server (use `voxa dev` or host the server yourself).

Dynamic pages are only built for the values you list. Add a `paths()` function to the page file:

```python
# pages/blog/[slug].py
def paths():
    return [{"slug": "hello"}, {"slug": "world"}]   # builds /blog/hello and /blog/world
```

`voxa build` never deletes a folder it did not create; point `--out` at a new folder name if it complains.

## How it works

Python builds a scene description, the server exposes it as JSON, and a small Three.js runtime in the browser renders it. Click events are sent back to the server, your Python handler runs, and only the changed objects are sent to the browser.

## Roadmap

- [x] File-based routing and dynamic routes
- [x] Click and hover events
- [x] Physics
- [x] Text and buttons
- [ ] HTML overlays (forms, menus)
- [x] Textures, materials and shadows
- [ ] Multiplayer: a live shared scene
- [ ] Phone gyroscope, VR and AR
- [x] `voxa build` for static export
- [ ] PyPI release

## Version history

| Version | Highlights |
| ------- | ---------- |
| **v0.1** | First MVP: `Scene`, `Box`, `Sphere`, `Light`, `@app.page`, `voxa new`, `voxa dev` |
| **v0.2** | 3D models: `Model` loads `.glb` files from `assets/` |
| **v0.3** | Animations: `Bounce`, `Move`, `ColorShift`, `Pulse` |
| **v0.4** | Page links: tap an object to open another page |
| **v0.5** | Python event handlers: `on_click`, hover effects, per-visitor state |
| **v0.6** | File-based routing (`pages/`), dynamic routes, physics |
| **v0.7** | `Text` and buttons (any language), better phone camera |
| **v0.8** | `voxa build`: static export for free hosting |
| **v0.9** | Textures, materials, shadows and reflections |

### v0.1: the first MVP

The foundation: write a scene in Python and see it in the browser.

- `Scene`, `Box`, `Sphere` and `Light` (`Light.ambient()`, `Light.point()`)
- `App` with the `@app.page("/path")` decorator for multiple pages
- `voxa new <name>` creates a project, `voxa dev` runs the dev server
- FastAPI server with a bundled Three.js runtime that loads the scene as JSON
- Browser reloads by itself when the server restarts
- Drag to orbit, pinch or scroll to zoom
- Dependencies kept small (`fastapi`, `uvicorn`, `typer`), so it also installs on phones with Termux

### v0.2: 3D models

- `Model("duck.glb", scale=1.5, spin=0.5)` loads glTF / GLB files
- Files in the project's `assets/` folder are served at `/assets/...`
- `voxa new` creates the `assets/` folder
- Full URLs and absolute paths are accepted as the model source

### v0.3: animations

- New module with `Bounce`, `Move`, `ColorShift` and `Pulse`
- Attach with `.animate(...)`, any number per object: `Box().animate(Bounce(), ColorShift(to="gold"))`
- `Model` accepts `animations=[...]`
- `ColorShift` works on `Box` and `Sphere` only

### v0.4: page links

- `link="/about"` on any object opens that page when tapped or clicked
- A short tap counts as a click, a drag only rotates the camera
- The cursor becomes a pointer over linked objects
- The starter project got two pages that link to each other

### v0.5: events in Python

- `on_click=handler` runs a plain Python function on every click; a handler can take `(obj)` or `(obj, scene)`
- Decorator form: `@box.clicked`
- `hover_color` and `hover_scale` react to the pointer without a server round trip
- Every visitor gets their own copy of the scene, so clicks never affect other people
- Handlers can change `color`, `position`, `rotation`, `scale`, `spin`, `animations` and hover settings; only changed objects are sent back
- If a handler adds or removes objects the page reloads; errors inside a handler are printed in the terminal and the server keeps running
- Every object gained a `scale` option

### v0.6: file-based routing and physics

- **Routing:** every `.py` file in `pages/` is a page. `index.py` is `/`, `about.py` is `/about`, folders become path segments. Each file defines `page()` and returns a `Scene`
- **Dynamic routes:** `pages/blog/[slug].py` matches `/blog/anything` and receives `slug` as an argument
- Files and folders starting with `_` are ignored; `app.py` with `@app.page` still works alongside `pages/`
- **Physics:** `physics=True` (falls and collides) or `physics="static"` (solid ground), plus `mass`, `bounciness` and `Scene(gravity=...)`
- Moving a physics object from a click handler moves its body too
- `voxa new` now creates a `pages/` project with `index`, `about`, `physics` and `blog/[slug]` examples instead of a single `app.py`
- The physics engine (cannon-es) is only downloaded by pages that use it
- Physics is not supported on `Model` or `Text`

### v0.7: text and phone support

- `Text("Hello", size=0.7, color="#f8fafc")` draws text in the scene, always facing the camera
- Works in any language the device has a font for, including Bangla
- `background="#2563eb"` gives it a button look; `link`, `on_click`, `hover_scale` and animations work on it
- A click handler can change the text (`obj.text = "Taps: 3"`), for example to build a counter
- On tall, narrow screens the camera steps back automatically so wide scenes are not cut off
- The starter home page got a title and a "Tap me" counter button
- `Text` does not support `hover_color` or `physics`

### v0.8: static export

- `voxa build` writes the whole site into `dist/` as plain HTML, ready for GitHub Pages, Netlify or Cloudflare Pages
- `voxa build --base /name` for sites served from a sub-path such as `username.github.io/name/`
- `--out` chooses the output folder; `assets/` is copied along and a `.nojekyll` file is added
- Dynamic pages are built only for the values returned by `paths()` in the page file; pages without `paths()` are skipped and reported
- `voxa build` never deletes a folder it did not create
- Links, hover, animations, physics, text and models work on a static site; `on_click` handlers do not, because they need a server
- Preview with `python -m http.server -d dist`

### v0.9: textures and materials

- `texture="wood.jpg"` (from `assets/`, or a full URL) and `texture_repeat=(u, v)` for tiling
- `metalness`, `roughness`, `opacity`, `emissive` and `wireframe` on `Box` and `Sphere`
- With a texture, `color` tints the image; left unset, the image shows as it is
- `Scene(shadows=True)` for shadows from the point light
- `Scene(environment=True)` for soft studio reflections; it switches on automatically when an object has `metalness > 0.3`
- Click handlers can change material options at runtime
- `voxa new` creates a demo texture (`assets/checker.png`) and a `/materials` page showing gold, matte, glass, glow, wireframe and a textured floor
- Materials do not apply to `Model` and `Text`

### Upgrading

- From v0.5 to v0.6, project layout changed for new projects only. Old `app.py` projects keep working; you can mix both styles
- Static builds (v0.8) skip dynamic pages until you add `paths()` to them
- Every version keeps the earlier options working, so old scenes need no changes

## Contributing

Issues and pull requests are welcome. To run the tests:

```bash
pip install pytest
pytest
```

## License

MIT
