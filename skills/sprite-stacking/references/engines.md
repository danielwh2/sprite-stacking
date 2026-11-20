# Engine recipes

All of these run the loop from SKILL.md step 3: rotate each slice around its centre, scale y by `squash`, lift by `-round(z * spacing)`, whole-pixel position, nearest filtering.

## Web (Canvas 2D, React)

For a crisp outer outline, render into an offscreen canvas at 1x art-pixel size, outline its alpha once, then draw it onto the display canvas at an integer scale with `imageSmoothingEnabled = false`. `scripts/viewer.html` does exactly this, plus orbit, slice scan, exploded view and 16-direction export.

Without an outline, draw straight to the canvas:

```ts
function drawStack(
  ctx: CanvasRenderingContext2D,
  sheet: HTMLImageElement | HTMLCanvasElement,
  count: number,
  x: number,
  y: number,
  turn: number,
  squash = 0.75,
  spacing = 1,
) {
  const w = sheet.width / count;
  const h = sheet.height;
  const ox = -Math.floor(w / 2);
  const oy = -Math.floor(h / 2);
  const px = Math.round(x);
  const py = Math.round(y);
  ctx.imageSmoothingEnabled = false;
  for (let z = 0; z < count; z++) {
    ctx.save();
    ctx.translate(px, py - Math.round(z * spacing));
    ctx.scale(1, squash);
    ctx.rotate(turn);
    ctx.drawImage(sheet, z * w, 0, w, h, ox, oy, w, h);
    ctx.restore();
  }
}
```

## Godot 4 (`Node2D`)

Set the default texture filter to Nearest and keep the node on whole pixels.

```gdscript
extends Node2D

@export var sheet: Texture2D
@export var slices := 15
@export var turn := 0.0:
	set(v):
		turn = v
		queue_redraw()
@export_range(0.1, 1.0) var squash := 0.75
@export var spacing := 1.0

func _draw() -> void:
	if sheet == null:
		return
	var w := float(sheet.get_width()) / slices
	var h := float(sheet.get_height())
	var rect := Rect2(-floorf(w / 2.0), -floorf(h / 2.0), w, h)
	var spin := Transform2D(turn, Vector2.ZERO)
	for z in slices:
		var lift := Transform2D(Vector2(1, 0), Vector2(0, squash), Vector2(0, -roundf(z * spacing)))
		draw_set_transform_matrix(lift * spin)
		draw_texture_rect_region(sheet, rect, Rect2(z * w, 0, w, h))
```

`_draw()` only runs again after `queue_redraw()`, which is why `turn` has a setter. For an outer outline, draw the stack into a `SubViewport` and run a 1px silhouette outline shader on its texture once.

## Unity

One child `SpriteRenderer` per slice with ascending sorting order, Point filter mode, parent snapped to whole pixels. To outline, render all slices into a `RenderTexture` once and outline that.

## Phaser, Pygame, anything else

The same loop: rotate each slice around its centre, scale y by `squash`, lift by `-round(z * spacing)`, nearest filter.
