---
layout: post
title: "Locating Spokane Part 2"
categories: [osm-finder, beta, tutorial]
---
Now that we've found Domino's, let's take another look at basic entity relationships before we move onto more advanced geolocation techniques.

Before continuining, I'm assuming you've read the [first article](https://xetnus.github.io/blog/locating-spokane-part-1/) in this series, where we located Domino's pizza (#1) in the image below. In this article, we'll cover the hotel (#2) and chimneys (#3) using similar techniques. 

![](/blog/images/2022-01-27-highlighted.jpg)

## The Geolocation Process

From the last article, we already know that our search starts in Washington state. If you followed the [installation instructions](https://github.com/Xetnus/osm-finder#installation) on the GitHub page, you should already have the data for Washington state loaded in your database.

### 2. Hotel

![](/blog/images/2022-01-30-hotel.jpg)

We can clearly see what looks to be a hotel next to a highway. In OSM Finder, the highway is represented by a linestring and the hotel is drawn as a node.

Once the linestring and node are drawn, click on the next button to move to the next stage. Depending on the order in which you drew the hotel and highway, you'll either enter the properties for the node or linestring first. The order doesn't matter.

**Node Properties.** The category of the hotel should be `Building`. To determine the subcategory, we can take a look at the values shown on the [building Wiki page](https://wiki.openstreetmap.org/wiki/Key:building). Fortunately, there's a value named 'hotel' that we can use as our subcategory. Using 'hotel' as the subcategory could be risky, because there's no guarantee that it's been marked correctly in the OpenStreetMap data. However, we can always come back and remove the subcategory later if we hit a dead end in our search. Go ahead and select `hotel` as the subcategory, or type 'hotel' and hit enter.

We have a few options for the tags field. We could try scouring the Internet to figure out which hotel brand uses white and blue signs or orange and white paint. If we were successful in our search, we could try our luck at entering the `brand` tag again like we did with Domino's. Or, we could try something different, like using the [building:levels](https://wiki.openstreetmap.org/wiki/Key:building:levels) key to specify how many levels are in the hotel. There are four visible levels to this hotel, so let's use the `building:levels=4` tag.

![](/blog/images/2022-01-30-hotel-properties.png)

**Linestring Properties.** Next, we'll enter the properties for the highway. Since it's the same highway we saw in the last article, we'll use the same properties: `roadway` as the category and `any` as the subcategory. We'll also use the tag `bridge`.

**Relationships.** Finally, we're asked to enter information for the relationship between the hotel and highway. Playing it safe, we'll enter a maximum distance of 200m. If we want, we can also enter a minimum distance of roughly 5m, but it's not required.

**Results.** Finally, hit next to generate the query. Your query should look like this.

```
SELECT
  replace(replace(line1.osm_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || line1.osm_id AS line1_url, 
  replace(replace(node1.osm_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || node1.osm_id AS node1_url
FROM linestrings AS line1, nodes AS node1
WHERE line1.category = 'roadway' AND line1.tags ? 'bridge' AND node1.category = 'building' AND node1.subcategory = 'hotel' AND node1.tags->>'building:levels' = '4' AND ST_DWithin(line1.geom, node1.geom, 200) AND ST_Distance(line1.geom, node1.geom) > 5;
```

Pasting that query into your PostgreSQL interactive terminal should turn up 4 total results.

![](/blog/images/2022-01-30-hotel-results.png)

Luckily for us, they all refer to the same hotel: [Baymont Inn & Suites](https://www.openstreetmap.org/way/404682854).

### Technical Aside
Buildings and other closed polygons can be queried as nodes due to osm2pgsql's [centroid()](https://osm2pgsql.org/doc/manual.html#geometry-objects-in-lua) method. What this means for the investigator is that buildings are stored in the database as a single point. More accurately, the object's center of mass is stored. To visualize this, I've drawn a not-to-scale diagram of the hotel we just located.

![](/blog/images/2022-01-30-hotel-openstreetmap.jpg)

Looking at this diagram, the minimum and maximum distances we entered are compared to the center of mass of the hotel, not the outside edges of the building as one might expect. For larger buildings, it's important to take this into consideration when you're determining the min and max properties.

### 3. Chimneys

You probably have the hang of these basic techniques by now, but let's consider the situation where you only have two nodes. In this case, two chimneys.

![](/blog/images/2022-01-30-chimney.png)

It may not be obvious at first glace that there's a second chimney partially visible on the right side of the image, but it's there.

Go ahead and drop two nodes at the base of the chimneys in the image. While it's not strictly necessary that you drop them at the base, it's a good habit to draw nodes as near to the ground as possible. Doing so will save you some sanity once we get into more advanced techniques, like angle analysis.

**Node Properties.** Since we only have two nodes, we don't have to worry about entering properties for linestrings. We'll enter the same properties for both nodes.

Chimneys are a part of OpenStreetMap's [man_made](https://wiki.openstreetmap.org/wiki/Key:man_made) key, so we'll choose `man made` as the category. If you search through the list of values on that Wiki page, you'll come across the `chimney` value, which we'll use as our subcategory. Chimneys tend to have fewer tags in OpenStreetMap than some other categories and subcategories, so we'll skip the tags field. 

**Relationships.** For the relationship between the two nodes, enter what you think is appropriate. I'd recommend a maximum distance of no smaller than 30m and a minimum distance of no greater than 10m.

**Results.** Clicking next should generate the following query.

```
SELECT
  replace(replace(node1.osm_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || node1.osm_id AS node1_url, 
  replace(replace(node2.osm_type, 'N', 'www.openstreetmap.org/node/'), 'W', 'www.openstreetmap.org/way/') || node2.osm_id AS node2_url
FROM nodes AS node1, nodes AS node2
WHERE node1.category = 'man_made' AND node1.subcategory = 'chimney' AND node2.category = 'man_made' AND node2.subcategory = 'chimney' AND node2.osm_id != node1.osm_id AND ST_DWithin(node1.geom, node2.geom, 30) AND ST_Distance(node1.geom, node2.geom) > 10;
```

Running that in the terminal, we get 18 results. 

![](/blog/images/2022-01-30-chimney-results.png)

This time, the last two rows are what we're looking for: [node1](https://www.openstreetmap.org/way/366611954) and [node2](https://www.openstreetmap.org/way/366611953).

## Wrapping Up

In this article, we found the hotel (#2) and the chimneys (#3). We also covered how buildings and other closed polygons are converted into one center-of-mass point. Now that we've beaten these basic operations into the ground, we'll be covering more advanced techniques in the next article. More specifically, we'll figure out how to use angle analysis to geolocate the substation (#5) and railway (#6). Eventually, once I've figured out how to implement shape comparison algorithms, I'll write an article about finding the T-shaped roof (#4).

If you have any feedback to share, please feel free to send me a note using the email link in the footer below.
