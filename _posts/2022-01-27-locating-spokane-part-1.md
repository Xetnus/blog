---
layout: post
title: "Locating Spokane Part 1"
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

**Node Properties.** The category of the node we drew should be `Building`. To determine the subcategory, we can take a look at the values shown on the [building Wiki page](https://wiki.openstreetmap.org/wiki/Key:building). Based on those values, it looks like there could be multiple subcategories that apply to this building, such as `yes`, `commercial`, and `retail`. To play it safe, we'll just keep it at the default selection of `Any`.

Some people will instantly recognize the sign towering above the restaurant as the [Domino's](https://en.wikipedia.org/wiki/Domino%27s) logo. Others may need to zoom in to see it. Still others may need to look up a list of restaurant logos using their preferred search engine before they come to the same conclusion. Regardless of how you found it, there's an OpenStreetMap key to specify the [brand](https://wiki.openstreetmap.org/wiki/Key:brand) associated with a place. Given this, we'll input the tag `brand=Domino's`.

![](/blog/images/2022-01-27-dominos-node-properties.jpg)

**Linestring Properties.** Clicking on next allows us to enter the properties for the linestring. We'll select `Roadway` for the category. To select the subcategory, feel free to look through the dozens of potential values on the [highway Wiki page](https://wiki.openstreetmap.org/wiki/Key:highway). However, to save myself the time, I'm going to leave it at `Any`.

The highway looks to be a bridge of some sort. If we go to the [bridge Wiki page](https://wiki.openstreetmap.org/wiki/Key:bridge), we'll see that there are several possible values that we could use, like cantilever, trestle, and viaduct. There's also a 'yes' option, which is a non-specific description for a bridge. To keep it simple, we're not going to provide a value for this key; we'll just use the tag `bridge`.

![](/blog/images/2022-01-27-dominos-linestring-properties.jpg)


#### Relationships

Moving to the next stage, we're asked to enter the details of the relationship between the node and line. If we eyeball the distance between the highway and restaurant, a conservative estimate suggests that they come within 200 meters of each other. This will be the maximum distance. We could also enter a minimum distance if we feel comfortable doing so, but I'll leave it blank. The angle parameters are disabled because this type of relationship doesn't have any angles supported by OSM Finder.

![](/blog/images/2022-01-27-dominos-relationship.jpg)


#### Query and Results

Finally, hit next to generate the query. Once the query pops up, you can click on the copy icon at the top right to copy the query to the clipboard.

![](/blog/images/2022-01-27-dominos-query.jpg)

Pasting that query into your PostgreSQL interactive terminal should turn up 10 total results within a few seconds, which we'll need to sift through.

![](/blog/images/2022-01-27-dominos-results.png)

Fortunately for us, we don't have to sift for long, because the very first node is what we're looking for: [www.openstreetmap.org/way/224606443](https://www.openstreetmap.org/way/224606443)

Now that we know the location of the Domino's and highway depicted in the original image, it shouldn't take long to find exactly where the picture was taken. Taking in the totality of the image, we can determine that the picture was taken from the [Davenport Hotel Tower](https://goo.gl/maps/deQy3dGURyfpVjed8). Specifically, the picture was taken looking south, as shown below.

![](/blog/images/2022-01-27-dominos-geolocation.jpg)


### Wrapping Up

Those familiar with OpenStreetMap know that we made quite a few daring assumptions here. At the most basic level, we assumed that the highway and building had been added to OpenStreetMap. This isn't always the case when using OpenStreetMap data. On top of that, we made the assumption that the OpenStreetMap author of the building tagged it with `brand=Domino's`. We also assumed that the highway had been tagged as `bridge`. If any of these assumptions didn't hold true when compared to the current state of the OpenStreetMap data, our search would have fallen flat and we'd have been forced to widen our search using less restrictive parameters and tags. These are the inherent risks when using open-source data.

This article served as an introduction to using OSM Finder. In future articles, I'll cover more complex examples. Although I tried to make OSM Finder as intuitive as I could, I won't deny that the tool can be cumbersome to use. My goal for this blog is to make it easier for beginners to understand how to use the tool and to help everyone avoid common pitfalls while using the system.

If you have any feedback to share, please feel free to send me a note using the email link in the footer below.
