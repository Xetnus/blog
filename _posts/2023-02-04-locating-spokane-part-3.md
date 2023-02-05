---
layout: post
title: "Locating Spokane Part 3"
categories: [osm-finder, beta, tutorial]
---
To wrap up our quick investigation into Spokane, we'll explore how to use angles to improve the efficiency of our investigations.

Before continuining, it's assumed you've read the [first](https://xetnus.github.io/blog/locating-spokane-part-1/) and [second](https://xetnus.github.io/blog/locating-spokane-part-2/) articles in the series. In this article, we'll use [OSM Finder](https://osm-finder.netlify.app/) to geolocate the substation (#5) and railway (#6) using more advanced techniques.

![](/blog/images/2023-01-27-highlighted.jpg)

## The Geolocation Process

From the last article, we already know that our search starts in Washington state. If you followed the [installation instructions](https://github.com/Xetnus/osm-finder#installation) on the GitHub page, you should already have the data for Washington state loaded in your database.

Before searching for the substation, let's discuss how intersections can be leveraged in investigations and how OpenStreetMap's tagging feature can be both crucial and dangerous to an investigation.

### 6. Railway

![](/blog/images/2023-02-04-railway.jpg)

In this cropped image, we can see a railway suspended and crossing over a road. Both a railway and road are represented as a linestring in OSM Finder.

![](/blog/images/2023-02-04-railway-annotated.jpg)

In OSM Finder, a yellow circle is placed at the intersection of the two lines, indicating that we'll be able to enter information about the angle at that intersection in later stages.

**Railway Properties.** After hitting next, we'll select `railway` as the category. Taking a look at the [railway wiki page](https://wiki.openstreetmap.org/wiki/Key:railway), we come across the value 'rail', which can be used to describe "full sized passenger or freight trains in the standard gauge for the country or state." In other words, a normal railway. We can also check out the [taginfo railway page](https://taginfo.openstreetmap.org/keys/railway#values) to see that 'rail' is the most commonly used value for railways by a wide margin. It seems like a safe bet to use `rail` as the subcategory.

Now for the tags. Since `bridge` worked so well for us when we were searching for the Domino's and hotel in previous articles, let's use it here. After all, the description of the [bridge wiki page](https://wiki.openstreetmap.org/wiki/Key:bridge) says that "a bridge is an artificial construction that spans features such as roads, railways, waterways or valleys and carries a road, railway or other feature." The portion of the railway that passes over the road in the image seems to match that description, so why wouldn't this work? We'll revisit this question shortly.

![](/blog/images/2023-02-04-railway-line1-properties.png)

**Roadway Properties.** Next, we'll enter the properties for the road that travels underneath the railway. We'll select `roadway` as the category and `any` as the subcategory. If you're feeling adventurous, read through the values on the [highway wiki page](https://wiki.openstreetmap.org/wiki/Key:highway) and take a guess at which subcategory this road will be. It's a good skill to be able to identify the type of a highway quickly and accurately. For the tags, we have nothing to add right now, so we'll leave the field empty.

![](/blog/images/2023-02-04-railway-line2-properties.png)

**Relationships.** Finally, we're asked to enter information that describes the relationship between the two lines. Unlike our previous examples, we don't have the option to enter maximum and minimum distances here. Since the lines intersect, there is no distance between them. However, we're given the option to enter an angle and angle error.

Just like in the last article, all values should be entered as if the scene is viewed from directly above. Due to perspective distortion, it may not look like the lines intersect at exactly a 90 degree angle in the image, but we know that they do (or at least, the angle shouldn't be far from 90 degrees). So we'll enter `90` for the angle and `3` for the angle error, just to account for minor variance. We could also try, for instance, '2' for the error if we wanted to be slightly more restrictive, or '5' if we're feeling less confident.

With these values, we're looking for a railway and roadway that intersect at the angle range of 90 ± 3°, or 87° through 93°.

![](/blog/images/2023-02-04-railway-relationships.png)

**Results.** After hitting next, our query looks like this.

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

Running this query, we get 112 total results. At this point, we *could* search through all of them, expecting our target to appear somewhere in those results.

But to save some time, I'll cut to the chase. Our target isn't in that list.

**Bridges vs. Tunnels.** According to the [bridge wiki page](https://wiki.openstreetmap.org/wiki/Key:bridge), when "the lower feature \[roadway] is surrounded by earth then the lower feature \[roadway] should probably be tagged using tunnel" instead of the upper feature (i.e., railway) being tagged as a bridge. This description makes sense, as typically when we think of a tunnel it's surrounded by earth. But in our image, the roadway isn't surrounded by earth. So what gives?

Unfortunately, the wiki goes on to say that it's sometimes a matter of personal judgment whether a situation is tagged using the bridge or tunnel key. Therefore, the individual who drew the road and railway in OpenStreetMap was responsible for choosing whether the railway was tagged as a bridge or the road as a tunnel.

Since bridge didn't work, let's go back and remove the `bridge` tag from the railway and add a `tunnel=yes` tag to the roadway. We're using the 'yes' value since the [tunnel wiki page](https://wiki.openstreetmap.org/wiki/Key:tunnel) gives only six possible options, and 'yes' is the only one that seems to fit.

**Results (again).** Finally, after hitting next, our query should look essentially the same as before, but with some minor changes to the WHERE clause.

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
  WHERE line1.category = 'railway' AND line1.subcategory = 'rail' AND line2.category = 'roadway' AND line2.tags ? 'tunnel' AND ST_Intersects(line1.geom, line2.geom) 
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

Running this query, we get 62 total results, which is slightly more manageable. Diligently searching through the small haystack, we find the needle: [446607561](https://www.openstreetmap.org/way/446607561). Clicking on that ID, we see that this roadway is labelled as a [tertiary highway](https://wiki.openstreetmap.org/wiki/Tag:highway%3Dtertiary), which is described as "a road linking small settlements, or the local centres of a large town or city." Makes sense.

![](/blog/images/2023-02-04-railway-results.png)

The roadway is found and highlighted four times in the screenshot above. The reason for this is simple. Multiple parallel railways cross that same small section of road in OpenStreetMap, which can also be observed in the original photo of Spokane.

### 5. Substation

![](/blog/images/2023-02-04-substation.jpg)

Out of the 1,209 substations labelled in OpenStreetMap within Washington state, we're looking to find just one. To do this, we'll need to use other elements in the image to narrow our search down even more. Since we already know how to search for the intersection of the road and railway, let's use this as a starting point.

Just like buildings and other closed polygons, OSM Finder classifies substations as nodes, using the same centroid() method that was covered in [Part 2](https://xetnus.github.io/blog/locating-spokane-part-2/).

![](/blog/images/2023-02-04-substation-annotation.jpg)

**Node Properties.** Before jumping in, let's cover the basics. How do we know that it's a substation? Looking through the values listed on the [power wiki page](https://wiki.openstreetmap.org/wiki/Key:power), there are quite a few options. Using the short descriptions and pictures given for all of them, we can quickly narrow the values down to where only 'substation' is left. Understandably, this might not be intuitive to some, which is perfectly fine. In fact, even if you don't deduce that it's a substation and only label it with the category 'power', you'll return nearly the same results, it'll just take a bit longer for the query to run.

With that said, use the `power` category and `substation` subcategory (or don't provide a subcategory at all, your choice). We'll leave the tags empty.

**Linestring Properties.** Use the same properties we used above to describe the railway and roadway, including `tunnel=yes` for the road and no tags for the railway.

**Relationships.** When prompted to enter the details for the relationship between the two linestrings (i.e., the road and railway), use the same parameters we entered before. When prompted to describe the relationship between the node and the intersection, follow the instructions below.

When a node and intersection are both present in an OSM Finder instance, it's possible to input all four relationship parameters: max distance, min distance, angle, and angle error. This is also true for two non-intersecting linestrings. However, when an intersection is present, distances and angles will be defined in relation to that intersection point.

With this in mind, let's enter the parameters. The maximum distance will be, let's say, `100` meters. A minimum distance of `5` meters is a safe estimate. 

The angle is a little harder to estimate. Cognizant of the fact that the angle should be entered as if the scene is viewed from above, a `45` degree angle doesn't seem too ridiculous. Just to be safe, we'll enter an angle error of `10` degrees.

![](/blog/images/2023-02-04-substation-relationship.jpg)

**Results.** 
Clicking next, the program outputs a long query that looks like this.

```
WITH intersections AS
(
  SELECT
    ((ST_DumpPoints(
      ST_Intersection(line2.geom, line1.geom)
    )).geom) AS intersection1,
    line1.geom AS line1_geom,
    line1.osm_id AS line1_id,
    line1.osm_type AS line1_type,
    line2.geom AS line2_geom,
    line2.osm_id AS line2_id,
    line2.osm_type AS line2_type,
    node1.geom AS node1_geom,
    node1.osm_id AS node1_id,
    node1.osm_type AS node1_type
  FROM linestrings AS line1, linestrings AS line2, nodes AS node1
  WHERE line1.category = 'railway' AND line1.subcategory = 'rail' AND line2.category = 'roadway' AND line2.tags ? 'tunnel' AND node1.category = 'power' AND node1.subcategory = 'substation' AND ST_Intersects(line2.geom, line1.geom) 
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
    intersections.line2_type,
    intersections.node1_geom,
    intersections.node1_id,
    intersections.node1_type
  FROM intersections
  WHERE ST_DWithin(intersections.intersection1, intersections.node1_geom, 100) AND ST_Distance(intersections.intersection1, intersections.node1_geom) > 5
),
points AS
(
  SELECT
    ST_GeometryN
    (
      ST_Intersection(buffers.ring1, buffers.line2_geom)
      , 1
    ) AS ring1_p1,
    ST_GeometryN
    (
      ST_Intersection(buffers.ring1, buffers.line1_geom)
      , 1
    ) AS ring1_p2,
    buffers.intersection1,
    buffers.ring1,
    buffers.line1_geom,
    buffers.line1_id,
    buffers.line1_type,
    buffers.line2_geom,
    buffers.line2_id,
    buffers.line2_type,
    buffers.node1_geom,
    buffers.node1_id,
    buffers.node1_type
  FROM buffers
)
SELECT
  replace(replace(points.line1_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || points.line1_id AS line1_url,
  replace(replace(points.line2_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || points.line2_id AS line2_url,
  replace(replace(points.node1_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || points.node1_id AS node1_url
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
)
AND 
(
  (
    abs(round(degrees(
      ST_Azimuth(points.ring1_p1, points.intersection1)
      -
      ST_Azimuth(points.intersection1, node1_geom)
    ))::decimal % 180.0) BETWEEN 35 AND 55
  ) 
  OR
  (
    abs(round(degrees(
      ST_Azimuth(points.ring1_p1, points.intersection1)
      -
      ST_Azimuth(points.intersection1, node1_geom)
    ))::decimal % 180.0) BETWEEN 125 AND 145
  ) 
  OR
  (
    abs(round(degrees(
      ST_Azimuth(points.ring1_p2, points.intersection1)
      -
      ST_Azimuth(points.intersection1, node1_geom)
    ))::decimal % 180.0) BETWEEN 35 AND 55
  ) 
  OR
  (
    abs(round(degrees(
      ST_Azimuth(points.ring1_p2, points.intersection1)
      -
      ST_Azimuth(points.intersection1, node1_geom)
    ))::decimal % 180.0) BETWEEN 125 AND 145
  ) 
);
```

Running the query in the terminal, we get exactly two results which both correctly identify the substation: [289826522](https://www.openstreetmap.org/way/289826522). Mission accomplished.

![](/blog/images/2023-02-04-substation-results.png)

Interestingly, if we didn't specify that the power station was a substation and we used 'any' as the subcategory, we still would have only received four results to the query.

## Wrapping Up

Although this article was a little longer than the previous ones, we covered some crucial concepts in OSM Finder. Leveraging intersections and angles can dramatically improve your geolocation abilities, but you need to be cautious when trusting your own biases about how you think elements should be tagged. An image which shows a bridge to you may appear as a tunnel to someone else.

This wraps up our case study on Spokane, unless I come back to visit the T-shaped building (#4) later once I've figured out how to efficiently compare shapes. As always, if you have any feedback to share, please feel free to send me a note using the email link in the footer below.
