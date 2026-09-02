# Malmö Network Analysis — Cycling Fragmentation & Missing Links

Measures how Malmö's protected cycling network breaks into disconnected pieces around
Triangeln station, and finds the shortest new links that would knit those pieces back
together.

**Study area:** ~1 km² centred on Triangeln, Malmö (geocoded from OSM, not hand-typed).

## Method

1. Download the OSM bicycle network for the study area (OSMnx).
2. Classify every edge as **protected** (physically separated from motor traffic),
   **painted lane** (in the carriageway), or **mixed traffic** — reading both the
   `highway=cycleway` idiom and the `cycleway:left/right` tags-on-a-road idiom.
3. Take the protected edges as a subgraph and find its **connected components** — the
   islands of continuous protected riding.
4. Search for **missing links**: multi-source Dijkstra on the network with all protected
   edges removed, so every candidate path runs purely over unprotected street. Components
   are absorbing, which keeps each reported gap atomic — one hop between two islands.
5. **Score** each link as `rescued_km / gap_length × 100` — km of stranded protected
   network rescued per 100 m of new infrastructure.
6. Resolve the ranking into a **greedy build order** with union-find, so a second link
   between two already-joined islands is never counted twice.

## Layout

```
IN/     Color Pallette White Arkitekter.png — source for the notebook's design tokens
        240604_GML_Bao_Libny.ipynb — the Colab notebook this work grew out of, kept
        as provenance rather than to run
WORK/   Network Analyses White Malmo.ipynb — cycling fragmentation and missing links
        Nearest-facility and closest-facility allocation (schools, supermarkets).ipynb
OUT/    generated results — rebuilt by re-running a notebook
```

`OUT/` is gitignored: every file in it follows from the notebooks, so tracking them made
each re-run a diff of regenerated binaries. Two figures are excepted because the READMEs
embed them, and a README image has to be in the repo to render —
`triangeln_cycling_fragmentation_map.png` and `triangeln_supermarket_trip_load.png`.
`cache/` (OSMnx's HTTP cache), `.venv/` and `.ipynb_checkpoints/` are gitignored too.

## Outputs

| File | Contents |
| --- | --- |
| `triangeln_components.csv` | Each protected component: node count, length, network share |
| `triangeln_missing_links_ranked.csv` | Candidate links with gap length, rescued km, score |
| `triangeln_build_order.csv` | Greedy build sequence with marginal return per step |
| `triangeln_cycling_fragmentation.gpkg` | Network and proposed links as geopackage layers |
| `triangeln_cycling_fragmentation_map.png` | The map |
| `triangeln_missing_link_priority.png` | Priority ranking bar chart |

## Running it

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Then run `WORK/Network Analyses White Malmo.ipynb` top to bottom. The notebook includes
compatibility shims for both OSMnx 1.x and 2.x, and writes to `OUT/`.

## Colour

Marks use the White Arkitekter palette, with one deliberate departure documented in the
notebook: brand Teal `#389BAA` sits below the chroma floor a 2 px map line needs and is not
separable from brand Slate or Purple at map scale under colour-vision deficiency, so the
ordinal ramp is rebuilt around it. The notebook carries the full validation — contrast
ladder, ΔE separation under simulated CVD, and the reasoning.

## Caveats

- **Edge effects.** A 1 km² window truncates the network. A component that looks stranded
  here may continue outside the frame. The method is scale-independent; the numbers are not.
- **OSM completeness.** A missing `cycleway:*` tag reads as mixed traffic and will invent a
  gap. Spot-check top-ranked links against aerial imagery.
- **Topological vs. legal continuity.** Two protected ways meeting at a node count as
  connected even where the real movement requires dismounting or crossing.
