---
name: floorplan-to-3d
description: Turn a venue floor plan (image/PDF first; DXF/DWG later) into a layout-ready 3D space: pipeline, units/coords/GLB conventions, user confirmation gates, pitfalls.
---
# Floor plan → 3D space

Public skill for agents that turn **bitmap floor plans** into a layout-ready 3D shell. This repo ships the skill text plus a static demo under `demos/`. Deeper research notes belong in `docs/` (optional).

## How to work
1. Prefer GitHub first (mature repos, issues, similar pipelines). Cite repo URL, last update, license.
2. License gate: GPL / CC BY-NC / ODA non-member builds are reference-only, not product code. There is no downloadable commercial-weight open model for bitmap recognition that is clearly product-safe.
3. Numbers must be measured or sourced. Unmeasured → "未测"; estimates → mark 【估算】 with basis.
4. Log each benchmark: image source link, method, machine, size error, edits needed, time; fold new pitfalls into the pitfall table.

## Pipeline priority (public demo = image path)
1. **Image / PDF (MVP demo)**: recognition on the backend; walls via classical CV (OpenCV double-line pairing) or an axis-aligned wall extractor. LLMs only count/semantic-check (doors, indoor/outdoor)—not geometry. Flow: detect → report **structure only (no meters)** → user taps one long wall and enters real length → second known dimension as check → clickable fix list → extrude.
2. **DXF (next phase)**: browser preview with dxf-parser / three-dxf; parse with ezdxf (MIT). Expand blocks → units → layers for walls/doors/columns → parallel-line walls → corner union → extrude → GLB. User confirms units, crop, layer map; auto-propose longest outer wall length for one-tap confirm (`$INSUNITS` is often wrong).
3. **DWG**: do not buy ODA for MVP—ask user to Save As DXF. LibreDWG (GPL) only as an isolated server process after legal review.
4. **Fallback**: low confidence → keep plan as underlay; user enters width/height rectangle, or traces polyline outline.
5. **Site photos**: attachments only (materials, height, detail)—not generators.
6. **Blender**: offline LOD/bake/check only; server extrude can use trimesh.

## Conventions (summary)
- Units: meters. XZ floor, Y up, origin top-left of venue, all coords ≥ 0 (Three.js / glTF).
- DXF map: `X=(x−xmin)·m`, `Z=(ymax−y)·m`. trimesh extrude +Z then rotate −90° about X; Blender export +Y Up.
- Defaults: wall 0.2 m; door 0.9/1.2 × 2.1 m; ceiling height default **4.0 m** (event venues), marked as default.
- Site fields in optional `venue.site` (indoor/outdoor, height+source, main entrance openingId, scale source+confidence).
- One entrance source of truth: `venue.site.mainEntrance.openingId`; leave `venue.entrances` empty.
- GLB tree: Venue / Floor / Structure(Walls, Openings, Columns) / Markers(MainEntrance); node names `Type_UUID`; business data in extras.

## Demo copy that MUST appear (image path)
1. Upload: 「拖入场地平面图（PNG / JPG）」
2. Structure only: 「认出墙 / 门 / 柱 · 请确认比例」(no meters yet)
3. Scale (required): 「点一面最长外墙，输入真实长度」→「用第二条已知尺寸复核」
4. Fix list: 「点一下就能改：漏墙 / 误认家具 / 门洞」
5. Extrude: 「层高默认 4 m · 生成可布置的 3D」
6. Fallback: 「识别不准时：底图 + 手填长宽」

Fail review if step 3 is missing or meters are shown before confirm.

## Layout hints (reference only—not certification)
Cite codes as hints: e.g. GB 55037-2022 aisle/exit widths; NFPA 101 aisle clearances. Table/stage sizes have no single authority.

## Frequent pitfalls
- `$INSUNITS` wrong → cross-check `DIMLUNIT` + 3–200 m sanity.
- Browser parsers must expand `INSERT` blocks or floors look empty.
- Xref layers like `xref$0$A-WALL` → recursive decompose + regex map + user override.
- Door gaps break polygonize → parallel-line + buffer corners.
- Tiny closed blocks on wall layers → treat as columns.
- Bitmap CV mistaking furniture edges for walls is the main edit-budget killer.
- Validate DWG magic (`AC10…`) so HTML error pages are not parsed as CAD.

## Privacy / publish rules
- No private OS paths (`E:/…`), personal asset buckets, or real product brand names/logos in demos.
- Use placeholders: `./docs/`, `<ASSET_BASE_URL>`, fictional venue names.
- Sample benchmarks in docs must use public datasets or anonymized diagrams.

## Demo
See `demos/README.md` for the static storyboard acceptance checklist.
