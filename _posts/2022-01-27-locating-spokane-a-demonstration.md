---
layout: post
title: "Locating Spokane: A Demonstration Part 1"
categories: [osm-finder, beta, tutorial]
---

Sometimes you need to see a tool in action to understand how you can apply it to your own investigations. That's why I took a trip to Spokane, WA, USA to capture this photo.

![](/blog/images/2022-01-27-Spokane_WA_US.jpg)

If you've opened [OSM Finder](https://osm-finder.netlify.app/) recently, you might recognize it as the site's new placeholder image. This image is full of information that would normally make it trivial to geolocate using traditional techniques. Still, it's nearly a perfect image to demonstrate the capabilities of OSM Finder.

There are several specific items of interest that we'll be covering in this and future articles.

![](/blog/images/2022-01-27-highlighted.jpg)

We'll cover item 1 in this quick, introductory article. Items 2 and 3 will be covered in an article being released very soon. We'll hopefully go over item 4 at some point in the distant future, but searching for shapes is a difficult problem that [hasn't yet been implemented](https://xetnus.github.io/blog/introducing-osm-finder-beta/#future-work). Items 5 and 6 will be demonstrated in another article coming out within the next few days. Finally, item 7 is what kicks off this investigation. It's a billboard that, very conveniently, tells us which state to start our search in.

![](/blog/images/2022-01-27-billboard.jpg)

## The Geolocation Process

If you followed the most recent version of the [installation instructions](https://github.com/Xetnus/osm-finder#installation) on the GitHub page exactly, you should already have the data for Washington state loaded in your database. Now, it's just a matter of creating a query that'll sufficiently narrow our search area down.

### 1. Domino's pizza

In the very top left of the original image, we see a building with a highway passing directly next to it. Let's draw this relationship in OSM Finder. As of this beta release, buildings are all depicted using nodes in OSM Finder. Likewise, highways are all depicted as linestrings.

![](/blog/images/2022-01-27-dominos-annotated.jpg)

#### Properties

After clicking on next at the bottom of the page, you'll be asked to input the properties for the items you drew. Depending on the order you drew them, you'll either enter the properties for the node or linestring first. The exact order doesn't matter.

**Node.** The category of the node we drew should be `Building`. Based on the values shown on the [building Wiki page](https://wiki.openstreetmap.org/wiki/Key:building), there could be multiple subcategories that apply to this building, such as `yes`, `commercial`, and `retail`. To play it safe, we'll just keep it at the default selection of `Any`.

Some people will instantly recognize the sign towering above the restaurant as the [Domino's](https://en.wikipedia.org/wiki/Domino%27s) logo. Others may need to zoom in to see it. Still others may need to look up a list of restaurant logos using their preferred search engine before they come to the same conclusion. Regardless of how you found it, there's an OpenStreetMap key to specify the [brand](https://wiki.openstreetmap.org/wiki/Key:brand) associated with a place. Given this, we'll input the tag `brand=Domino's`.

![](/blog/images/2022-01-27-dominos-node-properties.jpg)

**Linestring.** Clicking on next allows us to enter the properties for the linestring. We'll select `Roadway` for the category. To select the subcategory, feel free to look through the dozens of potential values on the [highway Wiki page](https://wiki.openstreetmap.org/wiki/Key:highway). However, to save myself the time, I'm going to leave it at `Any`.

The highway looks to be a bridge of some sort. If we go to the [Wiki page](https://wiki.openstreetmap.org/wiki/Key:bridge) for the bridge key, we'll see that there are several possible values that we could use, like cantilever, trestle, and viaduct. There's also a 'yes' option, which is a non-specific description for a bridge. To keep it simple, we're not going to provide a value; we'll just provide a tag using the key `bridge`.

![](/blog/images/2022-01-27-dominos-linestring-properties.jpg)

Those familiar with OpenStreetMap know that we're making quite a few daring assumptions here. At the most basic level, we're assuming that the highway and building have been added to OpenStreetMap. This isn't always the case when using OpenStreetMap data. On top of that, we make the assumption that the OpenStreetMap author of the building tagged it with `brand=Domino's`. We also assume that the highway has been tagged as `bridge`. If any of these assumptions don't hold true when compared to the current state of the OpenStreetMap data, our search will fall flat and we'll be forced to widen our search using less restrictive parameters and tags. 

#### Relationships

Moving to the next stage, we're asked to enter the details of the relationship between the node and line. If we eyeball the distance between the highway and restaurant, a conservative estimate suggests that they come within 200 meters of each other. This will be the maximum distance. We could also enter a minimum distance if we felt comfortable doing so, but I'll leave it blank. The angle parameters are disabled because this type of relationship doesn't have any angles supported by OSM Finder.

![](/blog/images/2022-01-27-dominos-relationship.jpg)

#### Queries

Finally, hit next to generate the query. Once the query pops up, you can click on the copy icon at the top right to copy the query to the clipboard.

![](/blog/images/2022-01-27-dominos-query.jpg)

Pasting that query into your PostgreSQL interactive terminal should turn up 10 total results, which we'll need to sift through.

![](/blog/images/2022-01-27-dominos-results.png)

Fortunately for us, we don't have to sift for long, because the very first result is what we're looking for: www.openstreetmap.org/way/224606443


