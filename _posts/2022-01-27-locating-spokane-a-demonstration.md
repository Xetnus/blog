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

We'll cover items 1 through 3 in this introductory article. We'll hopefully go over item 4 at some point in the distant future, but searching for shapes is a difficult problem that [hasn't yet been implemented](https://xetnus.github.io/blog/introducing-osm-finder-beta/#future-work). Items 5 and 6 will be demonstrated in another article coming out within the next few days. Finally, item 7 is what kicks off this investigation. It's a billboard that, very conveniently, tells us which state to start our search in.

![](/blog/images/2022-01-27-billboard.jpg)

## The Geolocation Process

If you followed the most recent version of the [installation instructions](https://github.com/Xetnus/osm-finder#installation) on the GitHub page exactly, you should already have the data for Washington state loaded in your database. Now, it's just a matter of creating a query that'll sufficiently narrow our search area down.

### 1. Domino's pizza

![](/blog/images/2022-01-27-dominos-sign.jpg)

In the far top left of the original image, we see a building with a sign rising just above the highway directly next to it, shown above. Let's draw this relationship in OSM Finder. As of this beta release, buildings are all depicted using nodes in OSM Finder. Likewise, highways are all depicted as linestrings.

![](/blog/images/2022-01-27-dominos-annotated.jpg)

After you click on next, you'll be asked to input the properties for the items you drew. Depending on the order you drew them, you'll either enter the properties for the node or linestring first. The exact order doesn't matter.

Some people will instantly recognize the sign towering above the restaurant as the [Domino's](https://en.wikipedia.org/wiki/Domino%27s) logo. Others may need to zoom in to see it. Still others may need to look up a list of restaurant logos using their preferred search engine before they come to the same conclusion. Regardless of how you found it, there's an OpenStreetMap key to specify the [brand](https://wiki.openstreetmap.org/wiki/Key:brand) associated with a place. Given this, we'll input the tag `brand=Domino's`.

![](/blog/images/2022-01-27-dominos-node-properties.jpg)

A highway can be seen passing directly by the restaurant. If we eyeball the distance between the highway and restaurant, a conservative estimate suggests that they come within 200 meters of each other. The highway also looks to be a bridge of some sort. If we go to the [Wiki page](https://wiki.openstreetmap.org/wiki/Key:bridge) for the bridge key, we'll see that there are several possible values that we could use, like cantilever, trestle, and viaduct. There's also a 'yes' option, which is a non-specific description for a bridge. To keep it simple, we're not going to provide a value; we'll just provide a tag using the key `bridge`.

![](/blog/images/2022-01-27-dominos-linestring-properties.jpg)

![](/blog/images/2022-01-27-dominos-relationship.jpg)

Those familiar with OpenStreetMap know that we're making quite a few daring assumptions here. At the most basic level, we're assuming that the highway and building have been added to OpenStreetMap. You'll occasionally find that this isn't true when using OpenStreetMap data. On top of that, if in fact they have been added, we make the assumption that the OpenStreetMap author of the building tagged it with `brand=Domino's`. We also assume that the highway has been tagged as `bridge`. If any of these assumptions don't hold true when compared to the current state of the OpenStreetMap data, our search will fall flat and we'll be forced to widen our search using less restrictive parameters and tags. 

Finally, hit next to generate the query.

![](/blog/images/2022-01-27-dominos-query.jpg)

### 2. Hotel

### 3. Chimneys
