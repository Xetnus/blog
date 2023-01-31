---
layout: post
title: "Locating Spokane Part 2"
categories: [osm-finder, beta, tutorial]
---
Let's take another quick look at Spokane before we move onto more advanced techniques.

Before continuining, I'm assuming you've read the [first article](https://xetnus.github.io/blog/locating-spokane-part-1/) in this series, where we located the Domino's pizza (#1) in the image below. Now, we'll cover the hotel (#2) and chimneys (#3) using similar techniques. 

![](/blog/images/2022-01-27-highlighted.jpg)

## The Geolocation Process

From the last article, we already know to start our search in Washington state. If you followed the most recent version of the [installation instructions](https://github.com/Xetnus/osm-finder#installation) on the GitHub page, you should have the data for Washington state loaded in your database.

### 2. Hotel

Let's take a closer look at the hotel.

![](/blog/images/2022-01-30-hotel.jpg)

We can clearly see what looks to be a hotel next to a highway. In OSM Finder, the highway would be represented by a linestring and the hotel would be drawn as a node.

Once the linestring and node are drawn, click on the next button to move to the next stage. Depending on the order in which you drew the hotel and highway, you'll either enter the properties for the node or linestring first. The order doesn't matter.

**Node Properties.** The category of the hotel should be `Building`. To determine the subcategory, we can take a look at the values shown on the [building Wiki page](https://wiki.openstreetmap.org/wiki/Key:building). Fortunately, there's a value named 'hotel' that we can use as our subcategory. Using 'hotel' as the subcategory could be risky, because there's no guarantee that it's been marked correctly in the OpenStreetMap data. However, we can always come back and remove the subcategory later if we hit a dead end in our search. Go ahead and select `hotel` as the subcategory, or type it out and hit enter.

We have a few options for the tags field. We could try scouring the Internet to find out which hotel brand uses white and blue signs or orange and white paint. If we were successful in our search, we could try our luck at entering the `brand` tag again like we did with Domino's. Or, we could try something different, like using the [building:levels](https://wiki.openstreetmap.org/wiki/Key:building:levels) key to specify how many levels are in the hotel. There are four visible levels to this hotel, so let's use the `building:levels=4` tag.

![](/blog/images/2022-01-30-hotel-properties.png)

**Linestring Properties.** Next, we'll enter the properties for the highway. Just like in the last article, we'll use `Roadway` as the category and leave the subcategory at `Any`. We'll also tag the highway as `bridge` without a value.

**Relationships.** Finally, we're asked to enter information for the relationship between the hotel and highway. Playing it safe, we'll enter a maximum distance of 200m. If we want, we can also enter a minimum distance of roughly 5m, but it's not required.

**Results.** Finally, hit next to generate the query. Your query should look like this.

```
SELECT
  replace(replace(line1.osm_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || line1.osm_id AS line1_url, 
  replace(replace(node1.osm_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || node1.osm_id AS node1_url
FROM linestrings AS line1, nodes AS node1
WHERE line1.category = 'roadway' AND line1.tags ? 'bridge' AND node1.category = 'building' AND node1.subcategory = 'hotel' AND node1.tags->>'building:levels' = '4' AND ST_DWithin(line1.geom, node1.geom, 200) AND ST_Distance(line1.geom, node1.geom) > 5;
```

Pasting that query into your PostgreSQL interactive terminal should turn up 4 total results. Luckily for us, they all refer to the same hotel: [Baymont Inn & Suites](https://www.openstreetmap.org/way/404682854)

![](/blog/images/2022-01-30-hotel-results.png)

**Technical aside.** Buildings and other closed polygons can be queried as nodes due to osm2pgsql's [centroid()](https://osm2pgsql.org/doc/manual.html#geometry-objects-in-lua) method. What this means for the investigator is that buildings are stored in the database as a single point. Specifically, the point that's stored is the object's center of mass. To visualize this, I've drawn a not-to-scale diagram of the hotel we just located.

![](/blog/images/2022-01-30-hotel-openstreetmap.jpg)

Looking at this diagram, the minimum and maximum distances we entered are compared to the center of mass of the hotel, not the outside edges of the building as one might expect. For larger buildings, it's important to take this into consideration when you're deciding on the min and max values.

### 3. Chimneys

## Wrapping Up

If you have any feedback to share, please feel free to send me a note using the email link in the footer below.
