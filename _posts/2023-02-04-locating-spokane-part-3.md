---
layout: post
title: "Locating Spokane Part 3"
categories: [osm-finder, beta, tutorial]
---
Next up, let's explore how we can use angles to improve our searches.

Before continuining, it's assumed you've read the [first](https://xetnus.github.io/blog/locating-spokane-part-1/) and [second](https://xetnus.github.io/blog/locating-spokane-part-2/) articles in this series. In this article, we'll use OSM Finder to geolocate the substation (#5) and railway (#6) using more advanced techniques.

![](/blog/images/2023-01-27-highlighted.jpg)

## The Geolocation Process

From the last article, we already know that our search starts in Washington state. If you followed the [installation instructions](https://github.com/Xetnus/osm-finder#installation) on the GitHub page, you should already have the data for Washington state loaded in your database.

### 6. Railway

![](/blog/images/2023-02-04-railway.jpg)

Before we focus on the substation, let's spend some time understanding how certain intersections can be leveraged in investigation and how OpenStreetMap tagging can sometimes surprise and trick you.

In the image above, we can see a railway crossing over a road. Both of these items are represented as a linestring in OSM Finder.

![](/blog/images/2023-02-04-railway-annotated.jpg)

A yellow circle appears at the intersection of the two lines, indicating that we'll be able to enter information about the angle at that intersection at later stages.

**Railway Properties.** After hitting next, we'll select `railway` as the category. Taking a look at the [railway wiki page](https://wiki.openstreetmap.org/wiki/Key:railway), we come across the value 'rail' which is described as "full sized passenger or freight trains in the standard gauge for the country or state." In other words, a normal railway. We can also check out the [taginfo railway key](https://taginfo.openstreetmap.org/keys/railway#values) page to see that 'rail' is the most commonly used value for railways by a wide margin. It seems like a safe bet to use `rail` as the subcategory.

Now for the tags. Since `bridge` worked so well for us when we were searching for the Domino's and hotel in previous articles, let's use it here. After all, the description on the right side of the [bridge wiki page](https://wiki.openstreetmap.org/wiki/Key:bridge) says that "a bridge is an artificial construction that spans features such as roads, railways, waterways or valleys and carries a road, railway or other feature." The bridge we see in the image seems to match that description, so why wouldn't this work? We'll revisit this question in a few minutes.

![](/blog/images/2023-02-04-railway-line1-properties.png)

**Roadway Properties.** Next, we'll enter the properties for the road that travels underneath the railway. We'll select `roadway` as the category and `any` as the subcategory. If you're feeling curious, read through the values on the [highway wiki page](https://wiki.openstreetmap.org/wiki/Key:highway) and take a guess at which subcategory this road will be. We'll leave the tags field empty.

![](/blog/images/2023-02-04-railway-line2-properties.png)

**Relationships.** Finally, we're asked to enter information that describes the relationship between the two lines. Unlike our previous examples, we don't have the option to enter max and min distances here. Since the lines intersect, there is no distance between them. However, we're given the option to enter an angle and angle error.

Just like we learned in the last article, all values should be entered as if we're looking down at a map (i.e., from a top-down perspective). Due to perspective distortion, it may not look like the lines intersect at exactly a 90 degree angle in the image, but we know that they do (or at least, the angle isn't far from 90 degrees). So we'll enter `90` for the angle and `3` for the angle error, just to account for minor variance. We could also try '2' for the error if we wanted to be slightly more restrictive, or '5' if we were feeling less confident.

With these values, we're looking for a railway and roadway that intersect at an angle of 90 ± 3°, or 87° through 93°.

![](/blog/images/2023-02-04-railway-relationships.png)

**Results.** Finally, hit next to generate the query. Your query should look like this.

```
WITH intersections AS
(
  SELECT
    ((ST_DumpPoints(
      ST_Intersection(line1.geom, line2.geom)
    )).geom) AS intersection1,
    line1.geom AS line1_geom,
    line1.osm_id AS line1_id,
    line1.osm_type AS line1_type,
    line2.geom AS line2_geom,
    line2.osm_id AS line2_id,
    line2.osm_type AS line2_type
  FROM linestrings AS line1, linestrings AS line2
  WHERE line1.category = 'railway' AND line1.subcategory = 'rail' AND line1.tags ? 'bridge' AND line2.category = 'roadway' AND ST_Intersects(line1.geom, line2.geom) 
),
buffers AS
(
  SELECT
    intersections.intersection1,
    ST_ExteriorRing(ST_Buffer(intersections.intersection1, 0.5)) AS ring1,
    intersections.line1_geom,
    intersections.line1_id,
    intersections.line1_type,
    intersections.line2_geom,
    intersections.line2_id,
    intersections.line2_type
  FROM intersections
),
points AS
(
  SELECT
    ST_GeometryN
    (
      ST_Intersection(buffers.ring1, buffers.line1_geom)
      , 1
    ) AS ring1_p1,
    ST_GeometryN
    (
      ST_Intersection(buffers.ring1, buffers.line2_geom)
      , 1
    ) AS ring1_p2,
    buffers.intersection1,
    buffers.ring1,
    buffers.line1_geom,
    buffers.line1_id,
    buffers.line1_type,
    buffers.line2_geom,
    buffers.line2_id,
    buffers.line2_type
  FROM buffers
)
SELECT
  replace(replace(points.line1_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || points.line1_id AS line1_url,
  replace(replace(points.line2_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || points.line2_id AS line2_url
FROM points
WHERE
(
  (
    abs(round(degrees(
      ST_Azimuth(points.ring1_p2, points.intersection1)
      -
      ST_Azimuth(points.ring1_p1, points.intersection1)
    ))::decimal % 180.0) BETWEEN 87 AND 93
  ) 
);
```

Running this query, we get 112 total results. At this point, we could search through all of the results, expecting our target to appear somewhere in those results.

But to save some time, I'll cut to the chase. Our target isn't in that list.


### 5. Substation



## Wrapping Up



If you have any feedback to share, please feel free to send me a note using the email link in the footer below.
