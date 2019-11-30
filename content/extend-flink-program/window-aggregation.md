---
title: "Add time windows and aggregation"
chapter: false
weight: 20
---

To localize places in New York that are requesting currently requesting a high number of taxi trip and to generate average trip durations to the airports, we use time windows. Every time window will span an hour according to event time and we can then consider all events within that one hour window to generate the statistics we are interested in for this particular hour.

#### Count trips by pickup locations

To identify popular pickup locations, we'll generate a geo hash with a reduced precision and key by geo hashes. In this way, events in a similar geographic location will have the same geo hash. We then `keyBy` the geo hash and apply a time window of one hour (remember that this is according to event time, not actual processing time) to the resulting stream. In this way, all events with the same geo hash will be collected in the same window and we can easily determine the overall count for that time window. Finally, we actually count the number of events and emit a new `PickupCount` event that contains the resulting count for each geo hash.

Add the following statement after the `print` statement in Line 94.

<!-- "linenos=table,linenostart=99" -->
{{< highlight java>}}
DataStream<PickupCount> pickupCounts = trips
    .map(new TripToGeoHash())
    .keyBy("geoHash")
    .timeWindow(Time.hours(1))
    .apply(new CountByGeoHash());

pickupCounts.print();
{{< / highlight >}}
    


#### Compute average trip durations to airports

To compute the average trip durations, we'll transform the incoming trip events and only emit `TripDuration` events for trips that arrive at one of the airports. We then `keyBy` the pickup location and the destination, so that all trips that started at the same location and ended at the same airport are grouped together. Subsequently, we apply a time window of one hour and aggregate the duration of all `TripDuration` events in that time window that arrive to the same airport into an `AverageTripDuration` event.

Add this statement after the one you have just added in Line 97-103.

{{< highlight java>}}
DataStream<AverageTripDuration> tripDurations = trips
    .flatMap(new TripToTripDuration())
    .keyBy("pickupGeoHash", "airportCode")
    .timeWindow(Time.hours(1))
    .apply(new TripDurationToAverageTripDuration());
{{< / highlight >}}


#### Remove debug output

To verify that the computation is successful, you can comment out the debug output for `trips` in Line 94 and then restart the Flink program to inspect the output.

<!--"linenos=table,linenostart=96-->
{{< highlight java>}}
// trips.print();
{{< / highlight >}}
