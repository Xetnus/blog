---
layout: post
title: Introducing OSM Finder Beta
categories: [osm-finder, beta, log]
---

After four months of part-time development, OSM Finder is ready to enter the beta phase. OSM Finder has transitioned from a hardly functioning prototype that (by some miracle) won [Bellingcat's Digital Investigations Hackathon](https://www.bellingcat.com/resources/2022/10/06/automated-map-searches-scam-busting-tools-and-twitter-search-translations-here-are-the-results-of-bellingcats-second-hackathon/) to a functional and powerful geolocation tool.

## Updates (v0.3.1)

- A revamped user interface with fancy new colors, buttons, and other gadgets to play with, based on the [Quasar Framework](https://quasar.dev/).
- A "downsampling" capability that allows certain OpenStreetMap `way` types, like buildings, to be queried as nodes.
- Support for many more linestring and node types.
- A new default photo, which will be the basis for some instructional blog posts in the near future.
- And finally, this blog is live! I plan to use this blog for dev and update logs, as well as demonstrating real-world examples using OSM Finder.

## Future Work

With all that said, there's still a lot of work to do. 

The application would benefit from various UX improvements, including implementing a better image downscaler to reduce aliasing artifacts and adding a time estimate meter that provides a rough estimate as to how long a given query is expected to take to complete with the given parameters. These tasks are relatively straightforward.

A long-term and more difficult goal is to add support for querying closed and open polygonal shapes. This is no small feat, particularly due to image perspective distortion and the difficulty in comparing shapes of unknown dimensions/units. If you have experience in this and want to collaborate, please reach out using the email contact button in the footer below.

I'm also open to suggestions. If you have an idea for how to improve OSM Finder, please open an [issue on GitHub](https://github.com/Xetnus/osm-finder/issues) or submit a [pull request](https://github.com/Xetnus/osm-finder/pulls).
