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

> **Status:** early release (v0.6). The API may change.

## Features

- **File-based routing:** every file in `pages/` is a page, including dynamic routes like `pages/blog/[slug].py`
- **Primitives and models:** `Box`, `Sphere`, and `.glb` / glTF files with `Model`
- **Animations:** `Bounce`, `Move`, `ColorShift`, `Pulse`, plus `spin`
- **Interaction in Python:** `on_click` handlers, hover effects, clickable links between pages
- **Physics:** gravity, collisions and bouncing with one argument
- **Dev server:** `voxa dev` with automatic browser reload
- **Works on a phone:** touch drag to orbit, tap to click

## Installation

Voxa is not on PyPI yet, so install from source:

```bash
git clone https://github.com/YOUR_USERNAME/voxa.git
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

## How it works

Python builds a scene description, the server exposes it as JSON, and a small Three.js runtime in the browser renders it. Click events are sent back to the server, your Python handler runs, and only the changed objects are sent to the browser.

## Roadmap

- [x] File-based routing and dynamic routes
- [x] Click and hover events
- [x] Physics
- [ ] Text and UI overlays (titles, buttons)
- [ ] Textures and materials
- [ ] Multiplayer: a live shared scene
- [ ] Phone gyroscope, VR and AR
- [ ] `voxa build` for static export and deployment
- [ ] PyPI release

## Contributing

Issues and pull requests are welcome. To run the tests:

```bash
pip install pytest
pytest
```

## License

MIT
