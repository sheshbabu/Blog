---
title: Exploring Singapore’s OneMap API
date: 2023-06-29 16:07:54
image: /images/2023-exploring-singapore-onemap-api/2023-exploring-singapore-onemap-api.png
keywords: Singapore, OneMap, Geocoding, Routing, Directions, Postcode
tags:
  - API
---

Lot of people use Google Maps on a regular basis, however when it comes to programmatic access, using Google Maps API can get quite expensive.

If your mapping requirements are restricted to Singapore, you can use the free OneMap API developed by Singapore Land Authority (SLA). Apart from being free, it’s also considered to have the most accurate, detailed and up-to-date information.

![](/images/2023-exploring-singapore-onemap-api/2023-exploring-singapore-onemap-api.png)

## Authentication

Some endpoints such as geocoding don’t require authentication but others like routing require it. The authentication flow is a bit unconventional, here’s how it works:

**Initial registration**

- Register for an account [here](https://developers.onemap.sg/register/)
- Once registered, use the confirmation code sent via email [here](https://developers.onemap.sg/confirm_account/)

**Authenticated requests**

- Hit the `getToken` endpoint to generate new tokens
- This endpoint takes in your email and password set during registration
- The response contains an access token (JWT) with a TTL of 3 days
- It’s not mentioned in the API docs, but this looks similar to the [OAuth ROPC Flow](https://www.rfc-editor.org/rfc/rfc6749#section-1.3.3)

Here's how the `getToken` request and response looks like:

```shell
curl --request POST \
  --url https://developers.onemap.sg/privateapi/auth/post/getToken \
  --header 'Content-Type: application/json' \
  --data '{
  "email": "abc@def.com",
  "password": "hunter2"
}'
```

```json
{
  "access_token": "--token--",
  "expiry_timestamp": "--expiry--"
}
```

If you’re using JavaScript, you can store the token in an in-memory cache, and use something like an [Axios request interceptor](https://axios-http.com/docs/interceptors) to check for token expiry and either use the cache or fetch new token for authenticated requests. Same can be implemented using other libraries like [HTTPX](https://www.python-httpx.org/advanced/#event-hooks)

## Geocoding

[Geocoding](https://en.wikipedia.org/wiki/Address_geocoding) is the process of converting an address into map coordinates like latitude and longitude.

We can use the `search` endpoint to get the coordinates for a location, building or postal code. Here’s a query for “novena mrt”

```shell
curl --request GET \
  --url 'https://developers.onemap.sg/commonapi/search?searchVal=novena%20mrt&returnGeom=Y&getAddrDetails=N'
```

```json
{
  "found": 6,
  "totalNumPages": 1,
  "pageNum": 1,
  "results": [
    {
      "SEARCHVAL": "NOVENA MRT STATION (NS20)",
      "X": "29169.3297925005",
      "Y": "33633.1535391063",
      "LATITUDE": "1.32044079120154",
      "LONGITUDE": "103.843825618748",
      "LONGTITUDE": "103.843825618748"
    },
    ...
  ]
}
```

Notice how we get both X/Y coordinates in addition to the LATITUDE/LONGITUDE values. The X/Y coordinates (SVY21) can provide [more accurate representation](https://app.sla.gov.sg/sirent/About/PlaneCoordinateSystem) of the area compared to the LATITUDE/LONGITUDE values (WGS84).

Keep in mind that this endpoint is not [typo tolerant](https://www.algolia.com/doc/guides/managing-results/optimize-search-results/typo-tolerance/) so using the correct spelling is important.

## Reverse Geocoding

Reverse geocoding is, well, doing the reverse of geocoding - if you provide latitude/longitude, you get the address. We can either pass in the X/Y coordinates (SVY21) or LATITUDE/LONGITUDE coordinates (WGS84)

Using the X, Y values from the above search for “novena mrt”

```shell
curl --request GET \
  --url 'https://developers.onemap.sg/privateapi/commonsvc/revgeocodexy?location=29169.3297925005%2C33633.1535391063&token=--token--'
```

Same for the LATITUDE/LONGITUDE values:

```shell
curl --request GET \
  --url 'https://developers.onemap.sg/privateapi/commonsvc/revgeocode?location=1.32044079120154%2C103.843825618748&token=--token--'
```

```json
{
  "GeocodeInfo": [
    {
      "BUILDINGNAME": "NOVENA SQUARE",
      "BLOCK": "238",
      "ROAD": "THOMSON ROAD",
      "POSTALCODE": "307683",
      "XCOORD": "29172.1136298",
      "YCOORD": "33580.8177133",
      "LATITUDE": "1.319967484392001",
      "LONGITUDE": "103.84385063099295",
      "LONGTITUDE": "103.84385063099295"
    },
    {
      "BUILDINGNAME": "NOVENA SQUARE",
      "BLOCK": "238A",
      "ROAD": "THOMSON ROAD",
      "POSTALCODE": "307684",
      "XCOORD": "29172.1136298",
      "YCOORD": "33580.8177133",
      "LATITUDE": "1.319967484392001",
      "LONGITUDE": "103.84385063099295",
      "LONGTITUDE": "103.84385063099295"
    },
    {
      "BUILDINGNAME": "NOVENA MRT STATION",
      "BLOCK": "250",
      "ROAD": "THOMSON ROAD",
      "POSTALCODE": "307642",
      "XCOORD": "29171.2246894",
      "YCOORD": "33635.0957146",
      "LATITUDE": "1.3204583554775822",
      "LONGITUDE": "103.84384264546134",
      "LONGTITUDE": "103.84384264546134"
    }
  ]
}
```

Makes sense as both the Novena Mall and the MRT station are in the same location.

## Routing

OneMap also provides a way to get directions between two locations. Let’s plan a simple route from Thomson Medical Centre (1.32536064328127, 103.841457941408) to Novena MRT (1.3204583554775822, 103.84384264546134) by driving:

```shell
curl --request GET \
  --url 'https://developers.onemap.sg/privateapi/routingsvc/route?start=1.32536064328127%2C103.841457941408&end=1.3204583554775822%2C103.84384264546134&routeType=drive&token=--token--'
```

```json
{
  "status_message": "Found route between points",
  "route_geometry": "qxaGqqxxRA{@eAA_ABm@Bg@Jc@JKBFGBBb@Kf@Kl@C~@CdA@bBLJ@bAJhBLb@DL@t@FF?\\BtAHj@?J?XAvA_@`@Kd@QJE??DC~@i@f@_@h@a@bBqAz@m@",
  "status": 0,
  "route_instructions": [
    [
      "Left",
      "THOMSON ROAD",
      34,
      "1.32505,103.841685",
      11,
      "34m",
      "East",
      "North",
      "driving",
      "Head East On Thomson Road"
    ],
    [
      "Left",
      "THOMSON ROAD",
      145,
      "1.325062,103.841987",
      17,
      "145m",
      "North",
      "East",
      "driving",
      "Turn Left To Stay On Thomson Road"
    ],
    [
      "Uturn",
      "THOMSON ROAD",
      817,
      "1.326342,103.841838",
      102,
      "817m",
      "North",
      "North",
      "driving",
      "Make A U-turn And Continue On Thomson Road"
    ],
    [
      "Left",
      "THOMSON ROAD",
      0,
      "1.319661,103.843181",
      0,
      "0m",
      "North",
      "South East",
      "driving",
      "You Have Arrived At Your Destination, On The Left"
    ]
  ],
  "route_name": ["THOMSON ROAD"],
  "route_summary": {
    "start_point": "THOMSON ROAD",
    "end_point": "THOMSON ROAD",
    "total_time": 129,
    "total_distance": 995
  },
  "viaRoute": "THOMSON ROAD",
  "subtitle": "Shortest distance"
}
```

Here’s a screenshot from Google Maps, the [instructions](https://goo.gl/maps/Myg3sSVrd4tXpKnv9) are identical!

![](/images/2023-exploring-singapore-onemap-api/2023-exploring-singapore-onemap-api-01.png)

You can set other modes apart from driving like walk, pt, cycle using the `routeType` param. “pt” here stands for public transport which when used unlocks other params like `maxWalkDistance`, `mode` (TRANSIT, BUS, RAIL), etc.

## Conclusion

OneMap is a valuable resource for people building mapping applications in Singapore. There’s a couple of sharp edges like the search not being typo tolerant, unconventional API naming etc, but they pale in comparison to the utility provided.

Thanks for reading! Feel free to follow me in [Twitter](https://twitter.com/sheshbabu) for more posts like this :)
