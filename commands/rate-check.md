---
description: What a hotel actually costs for given dates, across every source Google compares
---

Check what a property really costs.

Ask me for the destination or hotel name and the dates if I have not given them. Phrase the search as `hotels in <place>` so Google places it, and add `gl` when the country could be ambiguous.

Then:

1. Call `hasdata_google_travel_hotels_getGoogleHotels` with `q`, `checkInDate` and `checkOutDate`, plus the guest mix if I gave one.
2. Check `gpsCoordinates` on the first result before anything else. If the hotels are not where I asked, say so and stop, because a page of hotels in the wrong country looks exactly like a good answer.
3. Shortlist five: name, `hotelClass`, `overallRating` with `reviews`, and the total rate for the stay. State whether you are quoting `lowest` or `beforeTaxesFees` and use the same one for every row.
4. Call the tool again with the `propertyToken` of the one I pick, keeping the same dates. Report `prices` source by source, `typicalPriceRange` for the stay, and `deal` when Google marks the price as a good one.
5. Read `reviewsBreakdown` and name the two topics reviewers mention most positively and most negatively, with the counts.

Quote the dates next to every figure. A nightly rate without its stay is not a price.
