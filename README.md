# Voxa

**Build 3D websites with pure Python.**

Write your scene in Python. Voxa serves it as a Three.js website with a built-in dev server and hot reload. No JavaScript required.

```python
from voxa import App, Scene, Box, Sphere, Light, Bounce, Pulse

app = App(title="My 3D Site")

@app.page("/")
def home():
    scene = Scene(background="#111827")
    scene.add(Box(color="tomato", spin=1.0, link="/about").animate(Bounce()))
    scene.add(Sphere(radius=0.5, position=(2, 0, 0), color="skyblue").animate(Pulse()))
    scene.add(Light.ambient())
    scene.add(Light.point(position=(5, 5, 5)))
    return scene
```

> **Status:** early MVP (v0.4). The API may change.

## Features

- **Primitives:** `Box`, `Sphere`
- **3D models:** load `.glb` / glTF files with `Model`
- **Animations:** `Bounce`, `Move`, `ColorShift`, `Pulse`, plus `spin`
- **Multiple pages:** `@app.page("/path")` routing
- **Clickable objects:** `link="/about"` opens another page when tapped or clicked
- **Dev server:** `voxa dev` with automatic browser reload
- **Orbit controls:** drag to rotate, pinch or scroll to zoom

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

Open <http://127.0.0.1:8000> in your browser. Edit `app.py` and the page reloads on its own.

> The browser loads Three.js from a CDN, so an internet connection is needed when viewing a site.

## Guide

### Scenes and lights

```python
scene = Scene(background="#000000", camera_position=(0, 1.5, 5))
scene.add(Light.ambient(intensity=0.4))
scene.add(Light.point(position=(5, 5, 5), intensity=1.0))
```

If you add no lights, Voxa adds a default ambient and point light.

### Objects

```python
Box(size=(1, 1, 1), color="tomato", position=(0, 0, 0), rotation=(0, 0, 0), spin=1.0)
Sphere(radius=0.5, color="skyblue", position=(2, 0, 0))
Model("duck.glb", scale=1.5, spin=0.5)   # file lives in your project's assets/ folder
```

`spin` is the rotation speed around the Y axis in radians per second.

### Animations

Attach one or more animations with `.animate(...)`:

```python
Box().animate(Bounce(height=1.0, speed=3.0))
Sphere().animate(Move(to=(2, 0, 0), duration=2.0), Pulse(amount=0.3))
Box(color="red").animate(ColorShift(to="gold", duration=2.0))
```

| Animation    | What it does                          | Works on        |
| ------------ | ------------------------------------- | --------------- |
| `Bounce`     | Hops up and down                      | all objects     |
| `Move`       | Moves to a position and back          | all objects     |
| `Pulse`      | Grows and shrinks                     | all objects     |
| `ColorShift` | Fades to another colour and back      | Box, Sphere     |

### Pages and links

```python
@app.page("/")
def home():
    scene = Scene()
    scene.add(Box(link="/about"))    # tap the box to open /about
    return scene

@app.page("/about")
def about():
    scene = Scene()
    scene.add(Sphere(link="/"))
    return scene
```

### 3D models

Put a `.glb` file in your project's `assets/` folder and reference it by name:

```python
scene.add(Model("duck.glb"))
```

Files in `assets/` are served at `/assets/...`.

## CLI

| Command                | Description                                   |
| ---------------------- | --------------------------------------------- |
| `voxa new <name>`      | Create a new project with a starter `app.py`  |
| `voxa dev`             | Run the dev server with auto reload           |
| `voxa dev --port 9000` | Use a different port                          |
| `voxa dev --host 0.0.0.0` | Make the site reachable from other devices |

## Project layout

```
myproject/
├── app.py        # your pages
└── assets/       # .glb models and other static files
```

How it works: Python builds a scene description, the server exposes it as JSON, and a small Three.js runtime in the browser renders it.

## Roadmap

- [ ] Hover and click events (`on_click`)
- [ ] Text and HTML overlays
- [ ] Textures and materials
- [ ] `voxa build` for static export
- [ ] Deploy guide and PyPI release

## Contributing

Issues and pull requests are welcome. To run the tests:

```bash
pip install pytest
pytest
```

## License

MIT
