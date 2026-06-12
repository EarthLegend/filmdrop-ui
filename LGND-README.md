# LGND fork of FilmDrop UI

This is a fork of [Element84/filmdrop-ui](https://github.com/Element84/filmdrop-ui)
maintained at [EarthLegend/filmdrop-ui](https://github.com/EarthLegend/filmdrop-ui).

The `main-lgnd` branch carries LGND-specific features that may not be
contributed back upstream; `main` is a clean upstream mirror. LGND
deployments build directly from the `main-lgnd` branch — pushing here is
what changes the deployed UI.

## Keeping in sync

```bash
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout main-lgnd
git rebase main
# resolve any conflicts, then:
git push origin main-lgnd --force-with-lease
```

## Changes on `main-lgnd` vs upstream

### Tiler-resolved render presets (lgnd-titiler integration)

With lgnd-titiler (LGND's titiler-based tile service) as the scene/mosaic
tiler, render presets resolve **server-side** from the collection's render
extension — the UI passes a named handle (`render=<id>`) instead of
expanding `assets`/`bidx`/`rescale`/`colormap` into every tile URL.
Rendering-param logic then lives in exactly one place (the tiler), not
duplicated per client.

**Config:** set `"TILER_RESOLVES_RENDERS": true`. `autoConfigureRendering`
then stores each collection render as `{ render: <id>, title }` instead of
expanded TiTiler params. Hand-written `visualizations` /
`mosaicTilerParams` entries may equivalently use `{ "render": "<id>" }` —
when present, all other params in the entry are ignored.

**Files changed:**

- `src/utils/configHelper.js` — `autoConfigureRendering` emits render
  handles when `TILER_RESOLVES_RENDERS` is set.
- `src/utils/mapHelper.js` — `constructSceneTilerParams` /
  `constructMosaicTilerParams` short-circuit to `render=<id>` when the
  visualization carries a `render` key; `buildMosaicTileUrl` handles
  lgnd-titiler's mosaic links (templated `{tileMatrixSetId}`, embedded
  `?url=dynamodb://...&collection=...`) alongside e84's bare-path links.

The mosaic flow relies on lgnd-titiler embedding `collection=` in the
register response's tiles/tilejson links, so mosaic tiles resolve the
default preset with zero client params (`render=<id>` to override).

> Known gap: the mosaic **register** request body still uses the e84
> mosaic-titiler contract (`stac_api_root` / `collections` / `max_items`);
> lgnd-titiler's `POST /mosaicjson/mosaics` expects
> `collection` / `bbox` / `datetime` / `limit`. Contract alignment is
> tracked internally.

### MajorTOM grid reference layer

A new `majortom-grid` service type for `LAYER_LIST_SERVICES` that renders the
ESA MajorTOM grid directly in the browser. Grid cells are computed from the
current viewport on every pan/zoom — no tile server required.

**Files changed:**

- `src/utils/majortomGrid.js` — grid geometry math (`getCellsInBbox`) and Leaflet
  layer factory (`createMajorTomGridLayer`). Draws rectangles into a `featureGroup`
  that redraws on `moveend`, with a configurable `minZoom` and a 5 000-cell safety cap.
- `src/utils/configHelper.js` — parses the new `majortom-grid` service type in
  `parseLayerListConfig`. Extracts `distance_meters`, `offset`, `min_zoom`, and
  `style` from config. Also refactors common layer fields (`combinedLayerName`,
  `layerName`, `layerAlias`, `visibility`, `type`) into a shared object.
- `src/utils/mapHelper.js` — new `createReferenceLayerInstance()` factory that
  dispatches on `refLayer.type` (`wms` or `majortom-grid`), replacing the previous
  inline WMS-only construction in `addReferenceLayersToMap` and
  `toggleReferenceLayerVisibility`.
- `CONFIGURATION.md` — documents the new service type and per-layer config fields.

**Configuration example:**

```json
{
  "LAYER_LIST_SERVICES": [
    {
      "name": "MajorTOM",
      "type": "majortom-grid",
      "layers": [
        {
          "name": "2560m",
          "alias": "MajorTOM 2560m grid",
          "default_visibility": false,
          "distance_meters": 2560,
          "offset": 0,
          "min_zoom": 8
        }
      ]
    }
  ]
}
```

## Previously upstreamed (now merged)

These changes originated on this fork and have been merged into upstream `main`.
They no longer appear in the `main-lgnd` diff.

- **ViewSelector geohex aggregation** (PR [#572](https://github.com/Element84/filmdrop-ui/pull/572)) —
  recognize `centroid_geohex_grid_frequency` as a hex-grid aggregation type.
- **Render config bidx extraction** (PR [#575](https://github.com/Element84/filmdrop-ui/pull/575)) —
  extract `bidx` from the STAC render extension in `autoConfigureRendering`.

## Branches

| Branch      | Status               | Purpose                 |
| ----------- | -------------------- | ----------------------- |
| `main`      | synced with upstream | Upstream mirror         |
| `main-lgnd` | active               | LGND integration branch |
