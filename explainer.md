<h2>Geofencing Explained</h2>

## What's All This About?

This API will allow web applications to react to changes in the users location without having to poll/watch the position constantly (allowing the device to be more energy efficient). Essentially it allows web applications to do things based on physical proximity of the device to specific locations.

## Registering A Geofence

Registering for a geofence is straightforward:

```html
<!DOCTYPE html>
<!-- https://app.example.com/index.html -->
<html>
  <head>
    <script>
      navigator.serviceWorker.register("/sw.js");

      navigator.serviceWorker.whenReady().then(function(sw) {
        navigator.geolocation.registerRegion(new CircularRegion(
          {
            id: "string id of region",
            latitude: 37.421999,        // degrees
            longitude: -122.084015,     // degrees
            radius: 100.0               // meters
          }
        )).then(function() { // Success
                  // No resolved value
                  // Success, region is now registered
                },
                function() { // Failure
                });
      });
    </script>
  </head>
  <body> ... </body>
</html>
```

Other than circular regions, other types of regions can be allowed as well, such as bluetooth LE based beacons, and maybe a kind of region that pops up UI for the user to pick his own geographic region. This would look something like this:
```js
navigator.geolocation.registerRegion(new BluetoothLowEnergyRegion(
  {
    id: "string id of the region",
    uuids: [],                      // list of BLE UUIDs (strings)
    radius: 1.0                     // (optional) some kind of radius when to trigger the region
  }
).then(...);
```

TODO(mek): Maybe pass the ID as a separate parameter, instead of it being part of the region. Other APIs seem to do it that way.

## Handling Geofence Events

Location notification happens from the Service Worker context via the new `regionenter` and `regionexit` events.

```js
// sw.js
self.onregionenter = function(event) {
  if (event.region.id == "string id of region") {
    // entered the region, do something useful
  } else {
    // garbage collect unknown regions (perhaps from older pages).
    navigator.geolocation.deregisterRegion(event.region.id);
};

self.onregionexit = function(event) {
  if (event.region.id == "string id of region") {
    // left the region, do something useful
  } else {
    // garbage collect unknown regions (perhaps from older pages).
    navigator.geolocation.deregisterRegion(event.region.id);
};
```


## Removing Geofences
```js
navigator.geolocation.deregisterRegion("string id of region to remove").then(
  function() {
    // removed successfully
  },
  function() {
    // removal failed, possibly because there was no region registered with the id
  });
```

TODO(mek): Other APIs seem to use unregister instead of deregister

## Enumerating registered Geofences
```js
navigator.geolocation.getRegisteredRegions().then(
  function(regions) {
    // regions is an array of all currently registered regions
  },
  function() {
    // failed to get list of regions.
    // simply no regions being registered is not considered a failure.
  });
```

## Notes

  * Since Service Workers are a requirement for Geofencing, and since Service Workers are limited to HTTPS origins, sites served without encryption will always fail to register for fences.
  * SW event handlers aren't allowed to run forever.

TODO(mek): Should Service Workers be a requirement, or should it be possible to use geofencing on a normal page as well (as long as the page remains loaded)?
