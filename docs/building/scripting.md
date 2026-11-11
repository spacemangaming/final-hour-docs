# Map scripts and templates

Maps can contain a script that reacts to things happening on the map, and the map file itself can use templates to avoid repeating yourself.

## Templates

Before a map is read, the whole file is run through the [Nunjucks](https://mozilla.github.io/nunjucks/templating.html) template language. You can use loops, variables and comments to generate repetitive parts of a map:

```xml
{# Five concrete stepping stones, each 4 tiles apart #}
{% for i in range(5) %}
<platform id="stone_{{ i }}" bounds="{{ i * 4 }} {{ i * 4 + 1 }} 0 1 0 0" type="concrete"/>
{% endfor %}
```

You can include another file from the maps folder with `{% include 'other.map' %}`. Files outside the maps folder can't be included.

## Scripts

Add a `<script>` element directly inside `<map>`, after `<body>`. Wrap the code in `CDATA` so characters like `<` don't break the XML:

```xml
<map bounds="0 20 0 20 0 10">
    <head></head>
    <body>
        ...
    </body>
    <script><![CDATA[
        events.on("playerAdded", (player) => {
            player.speak(`Welcome to the map, ${player.id}!`);
        });
    ]]></script>
</map>
```

Scripts are JavaScript. The script runs once when the map loads, and again each time the map is updated.

### What a script can use

| Name | What it does |
| --- | --- |
| `bounds` | The map's bounds: `{ minx, maxx, miny, maxy, minz, maxz }`. |
| `players` | The players on the map. |
| `entities` | The entities (zombies, objects) on the map. |
| `getEntitiesIn(query, includePlayers = true, excludeUnhittables = true)` | Entities in an area. `query` is `{ x, y, z, width, height, max_z }`. |
| `speak(text, interrupt, buffer, sound, excludes)` | Speaks to everyone on the map. |
| `playUnbound(sound, x, y, z, volume, streaming)` | Plays a sound at a position. |
| `getPlayerSpawn()` / `getZombieSpawn()` | A spawn position `{ x, y, z }` from the map's spawn zones. |
| `getElementById(id)` | A map element by its `id`, with its properties plus `inBound(x, y, z)`, `intersects(bounds)`, `update(properties)`, `remove()` and `destroy()`. |
| `events` | Map events: `playerAdded`, `playerRemoved`, `entityAdded`, `entityRemoved`. |

Players and entities have `id`, `type` (`"Player"`, `"Entity"` or `"GameObject"`), `x`, `y`, `z`, `playSound(...)`, and their own `events` (`hit`, `move`, `death`). Entities and players also have `move(x, y, z)`, and players have `speak(text)`.

Events support `on`, `once` and `off`.

### Limits

Scripts run in a sandbox, separate from the server:

- They can only use the functions above. There is no file, network or server access.
- The script's top-level code must finish within 1 second.
- Each event handler must finish within a quarter of a second.
- A script can use at most 32 MB of memory.

If a script breaks a limit or throws an error, the error is spoken to the players on the map (or to builders if the map is empty) and the server keeps running. A syntax error stops the map from loading, so `setmapdata` reports it and keeps the old map.
