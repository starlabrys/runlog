# Apple MapKit for display; Routes stored as WGS-84

The map is Apple MapKit. A Route's canonical coordinates are always WGS-84 (raw GPS) — in the local store, in GPX/JSON export, and on the backend. Converting to another datum is strictly a display concern, and the map layer sits behind a swappable `MapProvider` interface.

## Why

In mainland China every legal basemap uses the obfuscated GCJ-02 datum, while GPS returns WGS-84; plotting raw fixes on a GCJ-02 map offsets the Route by 50-500 m. MapKit aligns WGS-84 overlays to its China basemap automatically (pass `CLLocation.coordinate` straight to `MKPolyline`, never pre-converted), and the same code works abroad. It also needs no API key or billing account and is the most battery-efficient option; in China its data is AutoNavi's.

## Considered options

- **Google Maps**: unusable in mainland China (offset needs manual conversion, China map data frozen since ~2016, no watchOS SDK) — not chosen even though the developer's phone can reach it.
- **AutoNavi / 高德 SDK**: best China trail data and the intended upgrade path if MapKit's footpath detail proves insufficient — storage stays WGS-84, converted to GCJ-02 only when drawing the 高德 overlay. Not chosen for Phase 1: needs an account, a compliance SDK, and manual coordinate conversion.
- **Baidu**: rejected — BD-09 adds a second offset on top of GCJ-02.

## Consequences

- Once Routes exist, the canonical datum is baked into all stored and exported data, which is why it is decided up front.
