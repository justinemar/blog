---
title: easelist dev log | [rendering markers on the map based on geojson data with mapgl]
date: 2019-09-25 22:32:18
tags:
---

### Inspiration
I am working on a property listing website as a side project and while browsing similar applications i found a cool feature that i wanted to build here's the final output of that feature

![final](/images/final.gif)

### Goal
Render markers with dynamic data on an interactive map.
(I built this feature with a backend service for easelist but for simplicity sake we'll be using mock datas.)

### Prerequisite
Knows React and how to set it up.
Knows MapBox and how to set it up.

### GeoJSON

In order to display dynamic data on the map we need a geospatial data, in this case we'll be working with GeoJSON.
GeoJSON follows a strict format so before we proceed to rendering data to our map lets take a look at GeoJSON first,
Though we'll use `Point` for for the entirety of this writing it is good to know that there are seven Geometry Object types 
`Point`, `MultiPoint`, `LineString`, `MultiLineString`, `Polygon`, `MultiPolygon`, and `GeometryCollection`.


The Geometry object represents points, curves, and surfaces in coordinate space. A single Geometry Object is a valid GeoJSON Data, if you copy the Geometry Object below and paste it on [geojson.io](http://geojson.io/) The map viewport will change based on the coordinates.

```
   {
        "type": "Point",
        "coordinates": [120.9842, 14.5995]
   }
```

Here's another valid GeoJSON Data a Feature Object which has `properties` and `geometry` field. The value of geometry field shall either be a type of Geometry object as defined above or null, the properties value can be any JSON object or a null

```
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [
      120.9842,
      14.5995
    ]
  },
  "properties": {
    "name": "Manila Philippines"
  }
}
```

Can also be 

```

{
  "type": "Feature",
  "geometry": null,
  "properties": {
    "name": null
  }
}


```

FeatureCollection object is another valid GeoJSON which has a field `features` and the value of this field is an array object each consists of the Feature Object as defined above. The value of features can be empty.

```
{
   "type": "FeatureCollection",
   "features": [
      {
         "type": "Feature",
         "geometry": {
            "type": "Point",
            "coordinates": [
               120.9842,
               14.5995
            ]
         },
         "properties": null
      }
   ]
}
```

Can also be

```
{
   "type": "FeatureCollection",
   "features": []
}

```


### React and MapBox

Now we have basic idea of the structure of GeoJSON let's get to work!
We'll be using Uber's react-map-gl library it exposes some of MapBox features [react-map-gl](http://uber.github.io/react-map-gl) and d3-ease for transition effects

```
import React from "react";
import ReactMapGL, {
  Marker,
  LinearInterpolator,
  FlyToInterpolator,
  Popup
} from "react-map-gl";
import { easeCubic } from "d3-ease";


class MapGLRenderer extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      viewport: {
        latitude: 13.1162,
        longitude: 121.0794,
        width: "100%",
        height: "100vh" /*100% */,
        zoom: 12,
        center: [13.1162, 121.0794],
        transitionDuration: 5000,
        transitionInterpolator: new FlyToInterpolator(),
        transitionEasing: easeCubic,
        maxZoom: 12
      },
      token:
        "pk.eyda5ghIjoiYmVyYFAFiwiYSI6ImNrMG90cnpzNTA5YzUzbmtyMjFlano1ZDYifQ.pBd7NWQF3lCn2FADZVFS"
    };
  }

  handleViewPortChange = (viewport) => {
    this.setState({
      viewport
    });
  }

  render() {
    const { viewport, token } = this.state;
    return (
    <ReactMapGL {...viewport} mapboxApiAccessToken={token}
        onViewportChange={(viewport) => this.handleViewPortChange(viewport)}>

    </ReactMapGL>
    );

  }
}

export default MapGLRenderer;
```
We created a basic map above, In order to show our markers we have to style it.
You can use the following
```
.marker {
  width: 10px;
  height: 10px;
  outline: none;
  cursor: pointer;
  border: none;
  border-radius: 100px;
  background: #448fff;
}
```

Now this is where our data comes in..
Using this mock Feature Object we can add a marker to our map by providing its coordinates
```
const feature = {
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [
      120.9842,
      14.5995
    ]
  },
  "properties": null
}



<ReactMapGL {...viewport} mapboxApiAccessToken={token}
    onViewportChange={(viewport) => this.handleViewPortChange(viewport)}>
    <Marker
        latitude={feature.geometry.coordinates[1]}
        longitude={feature.geometry.coordinates[0]}
    >
        <button className="marker"></button>
    </Marker>
</ReactMapGL>

```
if we want more than one we can build a Feature Collection and map it.
```
const FeatureCollection = {
    type: 'FeatureCollection',
    features: [
        {
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [
            120.9842,
            14.5995
            ]
        },
        "properties": null
        },
        {
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [
                121.0437,
                14.6760
            ]
        },
        "properties": null
        }

    ]
}

 <ReactMapGL {...viewport} mapboxApiAccessToken={token}
        onViewportChange={(viewport) => this.handleViewPortChange(viewport)}>
        {FeatureCollection.features.map(feature => (
          <Marker
            latitude={feature.geometry.coordinates[1]}
            longitude={feature.geometry.coordinates[0]}
          >
            <button className="marker"></button>
          </Marker>

        ))}
      </ReactMapGL>

```

Now we have our markers we can interact with them but first populate the properties field for each Feature.
Then add a state named `selectedFeature` default to null and create function that will handle this state.


```
// Function that handles our selectedFeature
 setSelectedFeature(feature) {
    this.setState({
      selectedFeature: feature
    })
  }

// Populate properties
const FeatureCollection = {
    type: 'FeatureCollection',
    features: [
        {
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [
            120.9842,
            14.5995
            ]
        },
        "properties": {
            name: 'Manila',
            description: 'Ok Place'
        }
        },
        {
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [
                121.0437,
                14.6760
            ]
        },
        "properties": {
                name: 'Quezon City',
                description: 'Good place'
            }
        }

    ]
}
```

We have added an onClick handler to our marker and import in Popup component from react-map-gl passed in the selected feature coordinates and render its properties.

```
<ReactMapGL {...viewport} mapboxApiAccessToken={token}
    onViewportChange={(viewport) => this.handleViewPortChange(viewport)}>
    <Marker
        latitude={feature.geometry.coordinates[1]}
        longitude={feature.geometry.coordinates[0]}
    >
        <button className="marker" onClick={e => {
                e.preventDefault();
                this.setSelectedFeature(item);
              }}></button>
    </Marker>

      {selectedFeature ?
          <Popup
            latitude={selectedFeature.geometry.coordinates[1]
            }
            longitude={selectedFeature.geometry.coordinates[0]
            }
            onClose={() => {
              this.setSelectedFeature(null);
            }}
          >
            <div className="property-popup-container">

              <strong>
                <p className="is-size-6">
                  {selectedFeature.properties.name}
                </p>
                <p className="is-size-6">
                  {selectedFeature.properties.description}
                </p>
              </strong>
              \
              </div>
          </Popup>
          : null}
</ReactMapGL>
```

That's it! We're done! If you want to try implement this with a backend service here is the Model i used
for this particular feature of easelist

```
const FeatureSchema = new Schema({
  type: {
    type: String,
    default: 'Feature'
  },
  geometry: {
    type: {
      type: String,
      default: 'Point'
    },
    coordinates: {
      type: [Number],
      index: '2dsphere'
    }
  },
  property: {
    type: Schema.Types.ObjectId,
    ref: 'List'
  }
})


const AddonSchema = new Schema({
  property: {
    type: Schema.Types.ObjectId,
    ref: 'List'
  },
  community_features: {
    type: Array
  },
  policy: {
    dogs: String,
    cats: String,
    smoking: String,
    detail: String
  },
  amenities: {
    outdoor: {
      type: Array
    },
    indoor: {
      type: Array
    }
  },
  flooring: [String],
  parking: {
    dedicated: String,
    covered: String,
    garage: String
  },
  extra_loc: {
    type: Array
  }
})

const ListSchema = new Schema({
  status: {
    type: String,
    default: 'Pending'
  },
  publisher: {
    type: Schema.Types.ObjectId,
    ref: 'Publisher'
  },
  extras: {
    type: Schema.Types.ObjectId,
    ref: 'Addons'
  },
  lease: {
    terms: [String],
    detail: String
  },
  facts: {
    title: String,
    description: String,
    numOfBed: String,
    numOfBath: String,
    squareFeet: String,
    pricing: {
      type: Array
    },
  },
  location: {
    zip: Number,
    city: String,
    province: String,
    address: String
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  feature: {
    type: Schema.Types.ObjectId,
    ref: 'Feature'
  }
})
```