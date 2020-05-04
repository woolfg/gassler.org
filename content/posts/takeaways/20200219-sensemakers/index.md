---
title: "My 2min take-aways: Digital Twins, cities, neighbourhoods & rail infrastructure"
date: "2020-02-19"
slug: takeaways-sensemakers-20200219
tags: [takeaways,IoT,sensors]
hero: "banner.jpg"
summary: "High quality speakers and presentations at the sensemaker meetup about smart city projects, smart railway infrastructure and predictive maintenance, and new ways of visualizing data in wakable 3D environments."
---

I am not familiar with the IoT community, so I was quite surprised about the high quality of speakers and presented projects at the [sensemakers meetup](https://www.meetup.com/sensemakersams/events/xgqtlrybcdbzb/) with around 100 people. To my surprise, for such a hipster domain of IoT, there weren’t a lot of young people.

[Tanguy Coenen](https://twitter.com/tanguycoenen) presented  [Imec](https://www.imec-int.com/):
* Belgian company with 4000 researchers, designing (not building) sensors, chips, and doing a lot of consulting & research work in the area of smart things and IoT.
* Learned a new buzzword [*digital twin*](https://en.wikipedia.org/wiki/Digital_twin) which is defined as a digital replica of a living or non-living physical entity.
* They do a lot of “smart” city projects (e.g. [Antwerp](https://www.imeccityofthings.be)):
  * [Mounting cheap air quality sensors on parcel delivery cars](https://www.imeccityofthings.be/en/projecten/bel-air) to get more data points in the city. The data quality is lower in comparison to stationary sensors; they evaluate if it is worth it.
  * [Measuring water quality](https://www.imec-int.com/en/articles/internet-of-water-to-tackle-growing-water-risks) using thousands of chip sensors to e.g. prevent salty water in agriculture.
  * Collecting flow data of people and bicyclists in cities and buildings, and subsequently, use the data for simulations, prototyping, and building proofs of concepts.
* They always include ethical considerations and even have a team for it which is involved in all projects. It seems that they have a very holistic approach (and also the funding for it). For example, in a big project with a government, they are also looking into the question if there is real value in IoT/digital twins for the government or if it is just a hype.

Paul Kootwijk & Marcel Gerrits presented the dutch public railway infrastructure company [ProRail](https://www.prorail.nl/) and its innovation lab.
* Railway business is very slow, bureaucracy is a pain.
* They try to use cheap sensors almost everywhere:
  * Vibration sensors on bridges to detect truck collisions and help operators to decide if trains can still run on this bridge or traffic has to be stopped. Test bridge was hit more than 40 times within several weeks.
  * Predictive maintenance and surveillance of railways, point switches, crossings, doors, etc.
  * Train sensors (test on two trains) which can locate the train with a precision of 5cm to detect problems of tracks or catenary.
* Big challenge for them: rollout of projects from proof on concept to production.
* Currently no automated decisions are made but what if....one person in the audience asked about hacking attacks which could then black out public infrastructure. The answer was more like, yes, we heard about security and it seems that we have to think about it ;)


[Tom van Arman](https://twitter.com/tomvanarman) presented showcases how data can be visualized in the future:
* The [Lidar](https://en.wikipedia.org/wiki/Lidar) technology is super expensive and it needs a lot of manual work to create wakable 3D environments.
* Usually 3D videos/flights are pre-rendered and it takes ages to compute it. They used the [Unreal game engine](https://en.wikipedia.org/wiki/Unreal_Engine) to create 3D environments of real objects (e.g. Rijksmuseum) which are walkable in real time.
* He also presented several projects and videos which show how enriched 3D environments can be used to present data. The video below shows an example which combines the 3D point cloud of the [Marineterrein](https://www.marineterrein.nl/) where the [Living Lab](https://www.living-lab.nl/) is located, with additional data and information about the projects of the Living Lab.

{{< youtube yJYPfNaa0sU >}}





















