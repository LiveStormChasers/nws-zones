nws-zones
=========

National Weather Service forecast zone geometry, packaged for direct use in a
browser map, with a vector tileset built from it.

NWS publishes zone boundaries as shapefiles. This holds them as GeoJSON that a
map can fetch without a server, plus a PMTiles build of the same geometry for
rendering at scale.

Copyright (c) 2026 Live Storm Chasers Network LLC. See LICENSE.

The zone geometry itself is produced by the National Weather Service and is in
the public domain. This repository asserts nothing over it — see Attribution.


Why
---

A warning references zones by code, not by shape. To draw it you need the
geometry, and the official source is a shapefile, which a browser cannot read.

Two forms are kept here because they answer different questions:

    GeoJSON     look up one zone by code and get its polygon, fetched
                directly, no tile server
    PMTiles     render thousands of zones at once, at any zoom, as a vector
                layer

Fetching a whole GeoJSON to render the whole country is wasteful; querying a
tileset for one polygon by code is awkward. Keeping both avoids doing either.


Files
-----

    zones_main.json                   land forecast zones, 8,010 features
    zones_marine.json                 marine and coastal zones, 761 features
    zones_patches.json                replacement geometry, 271 features
    .github/workflows/
      generate-pmtiles.yml            builds the tileset with tippecanoe and
                                      publishes it as a release artifact


The patches file
----------------

This is the part worth understanding, and the reason the repository exists
rather than pointing at the official download.

NWS zone data lags reality. Zones are reshaped, retired and occasionally added,
and the published file is not always current — a warning can arrive whose zone
is missing or whose outline no longer matches the ground.

`zones_patches.json` is applied over the base files at load, so a fix is a
single commit rather than a rebuild. What it actually contains is worth being
precise about:

    271   zones in the patches file
    269   of which ALREADY EXIST in the base files
      2   which are genuinely new: AMZ284 and ANZ639

So this is almost entirely REPLACEMENT geometry, not missing zones. A patch
entry overrides the base outline for a zone the base already has. The two
additions are the exception rather than the pattern.

That matters for how it is applied: the merge must let a patch win over the
base, or 269 of the 271 entries do nothing at all and the file appears to work
while changing almost nothing.


The tileset
-----------

The workflow runs on push and on manual dispatch. It installs tippecanoe from
source, generates PMTiles from the GeoJSON, and uploads the result as a release
artifact rather than committing it — a tileset is a build product, and keeping
binaries out of the history keeps clones small.

Consumers fetch the tileset from the release URL. That URL is pinned to a
release tag, so regenerating does not silently change what a deployed map is
reading; a consumer moves when it chooses to.


Consumers
---------

A map application fetches the GeoJSON files directly from this repository at
runtime and the tileset from the release. That makes this a live dependency
rather than a build-time one: making this repository private, or deleting the
release, breaks zone rendering in anything pointing at it.

Worth keeping in mind before any housekeeping.


Not done
--------

There is no automated refresh. Updating the base files means downloading the
current NWS shapefiles, converting them and uploading — no workflow does this,
so the data is as current as the last manual update.

The patches file has no schema validation. A malformed entry fails at runtime
in the consumer rather than at commit time here.

There is no check that a zone code in the patches file is absent from the base
files. A patch that duplicates existing geometry would be applied silently.


Attribution
-----------

Forecast zone boundaries are produced by the National Weather Service and are
in the public domain. This repository redistributes that geometry with
corrections; it makes no claim over the underlying data.

    https://www.weather.gov/gis/

Tileset generation uses tippecanoe, developed by Mapbox and now maintained by
Felt, released under BSD 2-Clause. It is installed by the workflow and not
redistributed here.


Licence
-------

See LICENSE. The zone geometry is public domain and unrestricted. The build
workflow and the curation of the patches file are the only parts under the
repository's own licence.
