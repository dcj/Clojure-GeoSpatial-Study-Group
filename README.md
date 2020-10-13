# Clojure Spatial Analysis/GIS Study Notes - DCJ 2020-10-12

## Goals/Objectives:

* Study/review the new [ovid](https://github.com/willcohen/ovid) and [aurelius](https://github.com/willcohen/aurelius) libraries.

* Contemplate the relationships of ovid and aurelius to [geo](https://github.com/Factual/geo)

* Generate ideas/proposals for potential future/addtional work in these areas.

## Ovid

### [ovid.feature](https://github.com/willcohen/ovid#ovidfeature)

This is awesome!

For my aircraft noise analysis project, I'm using [H3](https://h3geo.org) to grid a portion of the San Fransisco Bay Area of interest.

[Example visualization showing the number of aircraft that flew over H3 hexagons (resolution 7) in one day](https://dcj.github.io/img/h3-aircraft-counts.png)

I want to determine the population count for each H3 hexagon.

I've found various data sources, boundaries for: counties, cities, US congressional districts, used `ovid.feature` to turn each such region/boundary into
a feature.
Also have features representing US Census blocks and block-groups, with population data.
Currently working to assign/apportion/aggregate Census block population into H3 hexagons.

When I started this task, I came up with something somewhat similar to `ovid.feature`, but not nearly as good or capable.
Once @willcohen explained `ovid`, I reworked my code and it is all feature-based now.
Huge improvement!

#### Current comments/thoughts/ideas WRT ovid.feature:

Seems like it would be good to have a `feature?` predicate.

Given a feature, I want to access its properties, and add new properties.  The current functions `assoc-properties` and `update-properties` are fine, but they are kind of blunt tools. Let's say I want to assoc in a new key/value into the existing property map, seems like I need to

```
(defn add-properties-to-feature
  [f m]
  (feature/update-properties f (partial merge m)))
```

and my use of `(partial merge` here is dubious (not always what one would want).
I wonder if there should be a standard set of tools/functions to manipulate and access the properties of a feature...

I've needed to create a feature from a map of properties and a geometry, ATM I just create the hash-map I know it should be.  Should there be some sort of feature constructor?

AFAICT `ovid.feature` seems like an obvious candidate for inclusion into `factual/geo` itself, is there a good reason to keep it separate?

### [ovid.feature.specs](https://github.com/willcohen/ovid#ovidfeaturespecs)

Yes we need this.
Is it currently working?

`clojure.spec` is obviously the top priority, but I have also found great value/use for [Malli](https://github.com/metosin/malli), and in my future work, I anticipate/plan making further use of `malli`.  Is there a role for `malli` schemas in addition to support for `spec` (`ovid.feature.schemas`)?

### [ovid.io](https://github.com/willcohen/ovid#ovidio)

When searching the Internet for regions/boundaries of interest, it is very common to find them published as ESRI shapefiles.
It would be extremely valuable to be able to read (and potentially write?) shapefiles (directly!) in-to/out-of Clojure.

ATM, I do not understand the completeness/thoroughness of `ovid.io`.
Currently I use [ogr2ogr](https://gdal.org/programs/ogr2ogr.html) to convert shapefiles to GeoJSON, and then read that into geo.
If/when the shapefile support in `ovid.io` is feature-complete and robust, then it seems like another obvious candidate for inclusion into `geo.io`

ATM, I have not yet attempted to try/use `ovid.io`

## Aurelius

ATM, I haven't actually used this library yet (except for borrowing the idea from `aurelius.db` `ReadableColumn`)

But, I have some thoughts and opinions anyway :-)

TL;DR: `aurelius` currently contains several great ideas, and bunch of random tools/helpers that might best be refactored into one or more other libraries.

### [aurelius.db](https://github.com/willcohen/aurelius#aureliusdb)

I've always wanted, and wondered how, to do the conversion from `PostGIS` into `geo`, and the `extend-protocol rc/ReadableColumn` does exactly that, thank you, thank you, thank you!

IMHO, the implementation here is overly simplistic, and doesn't handle the other `Postgres` to `Clojure` conversions one might also want.

[See this for an unfinished and incomplete implementation for a number of `Postgres/Clojure` conversions](https://gist.github.com/dcj/af3b723a0f6b81c773936b801087c28d)
(N.B. my current handling of `keyword/enum` converstions is horrific...)

Can we also convert *from* `geo` *to* `PostGIS` via `SettableParameter`?  That would be awesome, and that is on my task list to explore...

The code under the comment/section `Create Database component` has no business being in this library. (I have very similar code myself, the idea is fine, just don't put it here)

I don't yet understand the use for the `def` underneath `Create internal sqlite datasource`. `def`ing a global singleton is a `component` no-no, if some part of `aurelius` needs a `sqlite database`, shouldn't it be passed in explicitly? (within the `context` in `component` speak...)

### [aurelius.jts](https://github.com/willcohen/aurelius#aureliusjts)

This seems extremely interesting, and I hope/plan to try this out ASAP as part of my "determine population count for H3 hexagons" task.

### [aurelius.conversions](https://github.com/willcohen/aurelius#aureliusconversions)

OK. Yes there is a need for this, I have namespaces of similar things.

In the future, should there be some sort of unified `geo` units/conversions namespace?

Things like `geo.spatial/earth-mean-circumference` might best be in such a place.

Not sure where this begins/ends.  I need to convert from/to feet<->meters, miles<->meters, etc.

### [aurelius.census](https://github.com/willcohen/aurelius#aureliuscensus), [aurelius.sql](https://github.com/willcohen/aurelius#aureliussql), [aurelius.resources](https://github.com/willcohen/aurelius#aureliusresources-dev)

ETL of US Census data is extremely valuable, and I'm doing a tiny bit of this now in my population-of-H3-hexagon task.

That being said, this code might best be re-factored into a different library.

Brainstorm ideas:

* Can we leverage/use [CitySDK](https://github.com/uscensusbureau/citysdk) (it's written in Clojurescript!)?  Can we use some or all of `CitySDK` from Clojure, instead of node/cljs?

* I've done some work to use schema definitions to drive database table definition, and coercion between the database and Clojure.  [Gungnir](https://github.com/kwrooijen/gungnir) seems to have done a much more interesting take on this.  I'd be interested in applying this to Census data.

* Seems like there is a lot of useful info in the US Census [TIGER](https://www.census.gov/programs-surveys/geography/guidance/tiger-data-products-guide.html) datasets.  Tools to load TIGER  into `PostGIS` and make it available to us that way might be helpful.  Is this a solved problem (elsewhere), or do we need/want more?

### [aurelius.util](https://github.com/willcohen/aurelius#aureliusutil)

Not sure if this is generally useful, and if so, where it should live.

## [Geo](https://github.com/Factual/geo)

### [geo.h3](https://github.com/Factual/geo#geoh3)

Use it, love it!

The core H3 library is now at [Release v3.7.1](https://github.com/uber/h3/releases/tag/v3.7.1), and the corresponding support in [h3-java](https://github.com/uber/h3-java) is merged into tip-of-tree, but not currently in a tagged release. At some point soon-ish, I'll need a version of `geo` that supports these updates.

### [geo.jts](https://github.com/Factual/geo#geojts)

Currently uses

```
[org.locationtech.jts/jts-core "1.16.1"]
[org.locationtech.jts.io/jts-io-common "1.16.1"]
```

And current is `1.17.1`.
When I mistakenly dragged in the newer JTS release into a project, it broke `geo`.
Someday we might want to look at this.

### [geo.io](https://github.com/Factual/geo#geoio)

For the purposes of the discussion below, I define the term `geoEDN` as being the straightforward conversion of a geoJSON string into EDN, with no further processing.
Thus, geoEDN and geoJSON can represent the exact same thing, just the format is different.
To a Clojure programmer, the only thing geoJSON is good for is interop with the external world, we can't operate on geoJSON directly...

Example geoJSON/geoEDN helper functions:

```
(defn geojson-type
  [{:keys [type] :as geoedn}]
  type)

(defn geojson-feature-collection?
  [geoedn]
  (-> geoedn
      geojson-type
      (= "FeatureCollection")))

(defn geojson-feature?
  [geoedn]
  (-> geoedn
      geojson-type
      (= "Feature")))
```

#### GeoJSON function complection

I have some issues with the current `geo.ip/read-geojson` and `geo.io/to-geojson` and their variants....

IMHO, these functions complect:
* conversion between JSON and EDN
* translating between `geo`/`jts` `geometry` and GeoJSON `geometry`

I need to invoke the following operations/transformations independently:
* Reading a string/file containing geoJSON into geoEDN (no need for `geo` here)
* Transforming a geoEDN feature into an `ovid/feature` (replacing geoEDN geometry with `jts` geometry
* Transforming an `ovid/feature` into a geoEDN feature (replacing `jts` geometry with geoEDN geometry
* Writing geoEDN to a string/file of (geo)JSON (no need for `geo` here)

My motivations include:
* I might want to manipulate the geoEDN (read from external) prior to conversion to an `ovid/feature` with `jts` geometry
* To visualize an `ovid/feature` using a Javascript GeoJSON library that has been wrapped/integrated with Clojurescript, the Clojurescript wrapper needs/requires geoEDN, not geoJSON.

At the moment, here is how I have to convert an `ovid/feature` into a geoEDN feature:

```
(defn feature->geoEDN-feature
  [f]
  {:type "Feature"
   :properties (feature/properties f)
   :geometry (-> f
                 feature/geometry
				 geo.io/to-geojson
				 (json/read-str :key-fn keyword))})
```

Yuck!

A GeoJSON `FeatureCollection` can be non-optimal/problematic to a Clojure developer, usually we'd prefer a (Clojure) collection of features, where we can bring the full power of Clojure to processing that collection. Maybe we need a function that converts a collection-of-features to a geoEDN `FeatureCollection`, if we need/want a FeatureCollection (can be needed for interop...

#### `geo.io/read-geojson`

When reading a file/string containing a GeoJSON `FeatureCollection` via `read-geojson` it returns a lazyseq of features, that do *not* contain the GeoJSON `{:type “Feature”}`.
This seems like a bug to me.
Thoughts?

### Support for 3D and 4D:

Much of my work is with aircraft trajectories, which consist of a series of 4D positions: `[longitude, latitude, altitude, time]`.
In order to use `geo/point` I need to create a 4D `geo/coordinate`.
I need to take a fresh/new look at how `geo` helps/fails to support my needs here, stay tuned....

## Visualization

To support geospatial visualization, I've done a bunch of work to wrap/use [deck.gl](https://deck.gl) with Clojurescript, in collaboration with another Clojurian.

[Here is a short screencast showing a web application for displaying and animating aircraft trajectories](https://dcj.github.io/img/Historic.mp4)

We then adapted our Clojurescript-wrapped deck.gl to make it accessible from the Clojure REPL, similar to how `Oz` makes `vega` accessible from the Clojure REPL...
We hope/plan to get this code into releaseable condition over the next few months.
