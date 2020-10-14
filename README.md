# Clojure Spatial Analysis/GIS Study Notes
# DCJ 2020-10-14

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
a feature.<br>
Also have features representing US Census blocks and block-groups, with population data.<br>
Currently working to assign/apportion/aggregate Census block population into H3 hexagons.<br>

When I started this task, I came up with something somewhat similar to `ovid.feature`, but not nearly as good or capable.<br>
After I heard @willcohen explain `ovid`, I reworked my code and it is all feature-based now.<br>
Huge improvement!<br>

#### Current comments/thoughts/ideas WRT ovid.feature:

* It would be good to have a `feature?` predicate.

* Given a feature, I want to access its properties, and add new properties.<br>
The current functions `assoc-properties` and `update-properties` are fine, but they are kind of blunt tools.<br>
Let's say I want to assoc in a new key/value into the existing property map, seems like I need to

        (defn add-properties-to-feature
          [f m]
          (feature/update-properties f (partial merge m)))

    and my use of `(partial merge` here is dubious (not always what one would want).<br>
	Should there be a standard set of tools/functions to manipulate and access the properties of a feature?

	Update:  I've already changed this to:

        (defn add-properties-to-feature
          [f m]
          (feature/update-properties f #(merge % m)))

* I've needed to create a feature from a map of properties and a geometry, ATM I just create the hash-map I know it should be.
Should there be a feature constructor function?

* AFAICT `ovid.feature` seems like an obvious candidate for inclusion into `factual/geo` itself, is there a good reason to keep it separate?

#### Hash-map/row-orientation versus feature-orientation

As a Clojure programmer, hash-maps are king, it is the primary datatype used to organize data.<br>
When I read rows from a database, each row is a hash-map, so any geometries in that row/table are included in the hash-map.<br>
With `Feature`s, for some tasks, it now may be preferable to transform a geometry-containing-database-row into a `feature`, with
the non-geometry key/values placed in the feature's properties, and the geometry key's value placed in the feature's geometry.<br>
This feels like a powerful tool for certain tasks.

Perhaps there is an opportunity for tooling to support this use-case, `row->feature`, `feature->row`.<br>
If there is a need to provide fine grained control over these transformations, might that be kept in the (Clojure) metadata?

Random thought:  Is there something useful/cool to be done with `datafy` and `nav` WRT `feature`?

### [ovid.feature.specs](https://github.com/willcohen/ovid#ovidfeaturespecs)

Yes we need this.
Is it currently working?

`clojure.spec` is obviously the top priority, but I have also found great value/use for [Malli](https://github.com/metosin/malli), and in my future work, I anticipate/plan making further use of `malli`.<br>
Is there a role for `malli` schemas in addition to support for `spec` (`ovid.feature.schemas`)?

### [ovid.io](https://github.com/willcohen/ovid#ovidio)

When searching the Internet for regions/boundaries of interest, it is very common to find them published as ESRI shapefiles.<br>
It would be valuable to be able to read (and potentially write?) shapefiles (directly!) in-to/out-of Clojure.

ATM, I do not understand the completeness/thoroughness of `ovid.io`.

Currently I use [ogr2ogr](https://gdal.org/programs/ogr2ogr.html) to convert shapefiles to GeoJSON, and then read that into geo.<br>
If/when the shapefile support in `ovid.io` is feature-complete and robust, then it seems like another obvious candidate for inclusion into `geo.io`

ATM, I have not yet attempted to try/use `ovid.io`

## Aurelius

TL;DR: `aurelius` currently contains several great ideas, and bunch of random tools/helpers that might best be refactored into one or more other libraries.

~~ATM, I haven't actually used this library yet (except for borrowing the idea from `aurelius.db` `ReadableColumn`)~~

~~But, I have some thoughts and opinions anyway :-)~~

Update: Now I've used `aurelius` a bit, see: [Using aurelius.jts and ovid.feature to determine population for H3 hexagon](https://github.com/dcj/Clojure-GeoSpatial-Study-Group#using-aureliusjts-and-ovidfeature-to-determine-population-for-h3-hexagon)

### [aurelius.db](https://github.com/willcohen/aurelius#aureliusdb)

I've always wanted, and wondered how, to do the conversion from `PostGIS` into `geo`, and the `extend-protocol rc/ReadableColumn` does exactly that, thank you, thank you, thank you!

IMHO, the implementation here is overly simplistic, and doesn't handle the other `Postgres` to `Clojure` conversions one might also want.

[See this for an unfinished and incomplete implementation for a number of `Postgres/Clojure` conversions](https://gist.github.com/dcj/af3b723a0f6b81c773936b801087c28d)
(N.B. my current handling of `keyword/enum` converstions is horrific...)

Can we also convert **from** `geo` **to** `PostGIS` via `SettableParameter`?  That would be awesome, and that is on my task list to explore...

The code under the comment/section `Create Database component` has no business being in this library. (I have very similar code myself, the idea is fine, just don't put it here)

I don't yet understand the use for the `def` underneath `Create internal sqlite datasource`. `def`ing a global singleton is a `component` no-no, if some part of `aurelius` needs a `sqlite database`, shouldn't it be passed in explicitly? (within the `context` in `component` speak...)

### [aurelius.jts](https://github.com/willcohen/aurelius#aureliusjts)

~~This seems extremely interesting, and I hope/plan to try this out ASAP as part of my "determine population count for H3 hexagons" task.~~

This is **awesome!**

Once you have drunk the `ovid.feature` Kool-Aid (and why wouldn't you?), AFAICT, there is little reason to use `geo.jts` directly, this namespace provides the JTS functions for/on features.

See: [Using aurelius.jts and ovid.feature to determine population for H3 hexagon](https://github.com/dcj/Clojure-GeoSpatial-Study-Group#using-aureliusjts-and-ovidfeature-to-determine-population-for-h3-hexagon)

### [aurelius.conversions](https://github.com/willcohen/aurelius#aureliusconversions)

OK. Yes there is a need for this, I have namespaces of similar things.

In the future, should there be some sort of unified `geo` units/conversions namespace?

Things like `geo.spatial/earth-mean-circumference` might best be in such a place.

Not sure where to "draw the line" on what should be in such a namespace/library.<br>
For example, I need to convert from/to feet<->meters, nautical-miles<->meters, etc.

### [aurelius.census](https://github.com/willcohen/aurelius#aureliuscensus), [aurelius.sql](https://github.com/willcohen/aurelius#aureliussql), [aurelius.resources](https://github.com/willcohen/aurelius#aureliusresources-dev)

ETL of US Census data is valuable, and I'm doing a tiny bit of this now in my population-of-H3-hexagon task.

That being said, this code might best be re-factored into a separate library.

Brainstorm ideas:

* Can we leverage/use [CitySDK](https://github.com/uscensusbureau/citysdk) (it's written in Clojurescript!)?  Can we use some or all of `CitySDK` from Clojure? (in addition to node/cljs)

* I've done some work to use schema definitions to drive database table definition, and coercion between the database and Clojure.<br>
[Gungnir](https://github.com/kwrooijen/gungnir) seems to be a vastly more interesting/capable take on this.<br>
I'd be interested in applying this to the ETL of Census data into Postgres.

* Seems like there is a lot of useful info in the US Census [TIGER](https://www.census.gov/programs-surveys/geography/guidance/tiger-data-products-guide.html) datasets.<br>
Tools to load TIGER  into `PostGIS` and make it available to us that way might be helpful.<br>
Is this a solved problem (elsewhere), or do we need/want more?

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

And JTS current is `1.17.1`.<br>
When I mistakenly dragged in the newer JTS release into a project, it broke `geo`.<br>
We might want to look at this...

### [geo.io](https://github.com/Factual/geo#geoio)

For the purposes of the discussion below, I define the term `GeoEDN` as being the straightforward conversion of a GeoJSON string into EDN, with no further processing.<br>
Thus, GeoEDN and GeoJSON represent the exact same thing, just the syntax/format is different.<br>
To a Clojure programmer, the only thing GeoJSON is good for is interop with the external world, we can't operate on GeoJSON directly...

Proposed GeoJSON/GeoEDN helper functions:

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
* Reading a string/file containing GeoJSON into GeoEDN (no need for `geo` lib here)
* Transforming a GeoEDN feature into an `ovid/feature` (replacing GeoEDN geometry with `jts` geometry)
* Transforming an `ovid/feature` into a GeoEDN feature (replacing `jts` geometry with GeoEDN geometry)
* Writing GeoEDN to a string/file of (Geo)JSON (no need for `geo` lib here)

My motivations include:
* I might want to manipulate the GeoEDN (read from external) prior to conversion to an `ovid/feature` with `jts` geometry
* To visualize an `ovid/feature` using a Javascript GeoJSON library that has been wrapped/integrated with Clojurescript, the Clojurescript wrapper needs/requires GeoEDN, not GeoJSON.

At the moment, here is how I have to convert an `ovid/feature` into a GeoEDN feature:

```
(defn feature->GeoEDN-feature
  [f]
  {:type "Feature"
   :properties (feature/properties f)
   :geometry (-> f
                 feature/geometry
                 geo.io/to-geojson
                 (json/read-str :key-fn keyword))})
```

Yuck!

We need functions that transform to/from a GeoEDN feature and an `ovid/feature`, and leave the EDN/JSON conversion to other well-established JSON libraries (e.g. `clojure.data.json`, `cheshire`, etc.)<br>
GeoJSON/GeoEDN `Feature`s and `FeatureCollection`s contain a top-level key `type`, should this be transformed into a distinguished namedspaced key (e.g. `:geojson/type`) in the `ovid/feature` properties map?

#### GeoJSON `FeatureCollection` considered harmful

A GeoJSON `FeatureCollection` can be non-optimal/problematic to a Clojure developer, usually we'd prefer a (Clojure) collection of features, where we can bring the full power of Clojure to processing that collection.<br>
We need functions that transform between collection-of-features and GeoJSON/GeoEDN `FeatureCollection`.<br>
There is signicant interop value in `FeatureCollections` that we want to leverage, but within Clojure, it is vastly preferable to have a "collection of features".

#### `geo.io/read-geojson`

When reading a file/string containing a GeoJSON `FeatureCollection` via `read-geojson` it returns a lazyseq of features, that do **not** contain the GeoJSON `{:type “Feature”}`.<br>
This seems like a bug to me.<br>
Thoughts?

### Support for 3D, 4D, and Trajectories:

* Much of my work is with aircraft trajectories, which consist of a series of 4D positions: `[longitude, latitude, altitude, time]`.<br>
Currently `geo` doesn't make it easy to work with altitude and time.<br>
In order to create a 4D `geo/point` I need to create a 4D `geo/coordinate`.

* An important operation in my work is to determine the point-of-closest-approach (`PCA`) of a trajectory to a location-of-interest (`LOI`), and including the distance-of-closest-approach (`DCA`) and time-of-closest-approach (`TCA`),
and I haven't found any `geo` support for these.<br>
PostGIS does provide some helpful (partial) support, via the `ST_3DClosestPoint` function.<br>
I could really use similar support in `geo`

## Visualization

To support geospatial visualization, I've done a bunch of work to wrap/use [deck.gl](https://deck.gl) with Clojurescript, in collaboration with another Clojurian (full discloure: colleague did virtually all the CLJS work...)

The main driver for this work was the development of a full-Clojure-stack web application for viewing both historic and real-time aircraft flight trajectories:
* [Viewing historic daily flight tracks, and sequencing subsequent days](https://dcj.github.io/img/Historic.mp4)
* [Viewing real-time flight tracks](https://dcj.github.io/img/Real-Time.mp4)

We then adapted our Clojurescript-wrapped `deck.gl` to make it accessible from the Clojure REPL, similar to how `Oz` makes `vega` accessible from the Clojure REPL...<br>
We hope/plan to get this code into releaseable condition over the next few months.

Using these tools, here is how we display a collection of `ovid/feature`s, each representing one US Census block, from the Clojure REPL, into a browser window:

```
(dviz/visualize
 browser-context
 (dlayers/geo-json-layer
  {:data      (jts->geojson-edn (features-census-blocks))
   :stroked   true
   :wireframe true
   :lineWidthScale 3
   :getLineColor [255 255 0]
   :getFillColor [100 100 100 0.5]
   :pickable true
   :onHover (fn [event & args]
              (if (and (.-object event)
                       (.-properties (.-object event)))
                (set-hiccup-tooltip
                 [(.-x event) (.-y event)]
                 [:div {:style {:background "tomato"}}
                  (pr-str (.-properties (.-object event)))])
                (set-tooltip [(.-x event) (.-y event)]
                             nil)))}))
```
N.B. The `:onHover` function above is "quick and dirty", and can easily be made both more "Clojure-y", and more aestheticly pleasing.

[A zoomed-in screenshot of the resulting visualization](https://dcj.github.io/img/census-blocks.png)

Some of my visualizations contain geospatial data, but are not themselves geospatial.<br>
[Here is an example chart that superimposes the times-of-closest-approach of aircraft onto the time series sound-level data obtained from a sound-level-monitor](https://dcj.github.io/tca-compare)<br>
(Make a short-in-time brush selection in the lower chart, the upper chart will then display the zoomed-in detail, hover over a vertical black or red mark near a sound peak to see aircraft metadata)<br>
The times-of-closest approach were obtained via a complex PostGIS/Postgres query.<br>
This chart was generated from the Clojure REPL, in a browser, via `vega-lite` (Oz-like)

## Using `aurelius.jts` and `ovid.feature` to determine population for H3 hexagon

I've wanted to obtain the population count of H3 hexagons in order to weight aircraft noise impacts within that hexagon by the number of people impacted.<br>
I didn't have a good idea how to do this, so back-burnered this task.

At the 2020-10-03 SciCloj "Clojure in Geography" meeting, I asked @willcohen about this, and he gave me some valuable suggestions about how to approach it, and
I've been doing just that, and used `ovid.feature` and `aurelius.jts` to do so.

Here is an example of this work:

```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; Assign population count to H3 hexagon
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(defn loi->feature
  [{:keys [lon lat label] :as loi}]
  (-> (jts/point lat lon)
      (feature/to-feature (assoc loi :type :loi :name label))))

;; Turn the LOI (hash-map) into a feature, and add the h3 indexes of interest to its properties
(def loi
  (->> mloi  ;; mloi is a hash-map that contains info about a location-of-interest...
       loi->feature
       (add-h3-indexes-to-point-feature acn-h3-resolutions)))

;; Get the H3 resolution 8 hexagon containing the LOI, as a feature, HOI
(def hoi
  (-> loi
      feature/properties
      :h3
      (get 8) ;; I store the H3 index for a LOI as property [:h3 resolution]
      feature/h3->feature
      (add-properties-to-feature {:type :h3-hexagon})))

;; View the hexagon
(view-feature bctx hoi)
```

![Hexagon-of-interest, HOI](https://dcj.github.io/img/hoi.png)

```
;; Import all the Census blocks as features
(def fcbs (features-census-blocks))

(count fcbs)

; => ~86k

;; Filter the collection of census block features to those that intersect with the H3(8) hexagon containing the LOI
(def hoi-census-blocks
  (let [filter-fn (aurelius.jts/intersects? hoi)]
    (into []
          (filter filter-fn)
          fcbs)))

(count hoi-census-blocks)

; => 73

;; View the subset of census blocks that intersect with HOI
(doseq [f hoi-census-blocks] (view-feature bctx f))
```

![US Census Blocks that intersect with HOI](https://dcj.github.io/img/hoi-with-census-blocks.png)

```
;; What is the population of all these census blocks?
(transduce (map #(-> % feature/properties :population)) + 0 hoi-census-blocks)

; => 4935

(defn fractional-containment-of-feature
  "Questions: do I need the cond below, and is difference the best function to call?"
  [base-feature feature]
  (cond
    (aurelius.jts/covers? base-feature feature) 1.0
    (aurelius.jts/disjoint? base-feature feature) 0.0
    :else (/ (-> feature
                 (aurelius.jts/difference base-feature)
                 aurelius.jts/get-area)
             (aurelius.jts/get-area feature))))

(defn population-contribution-of-feature
  [base-feature feature]
  (long (* (-> feature feature/properties :population)
           (fractional-containment-of-feature base-feature feature))))

(defn hexagon-population
  [hexagon features]
  (transduce (map #(population-contribution-of-feature hexagon %)) + 0 features))

(hexagon-population hoi hoi-census-blocks)

; => 4125
```

IMHO: using `ovid.feature` and `aurelius.jts` made this task easy and pleasurable!
