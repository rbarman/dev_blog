Title: How to create clickable entities with AR.js
Date: 2024-05-14
Category: Augmented Reality
Tags: AR.js

How to create clickable entities with AR.js

# 1) Setup

We first start with a simple AR.js application that shows the camera.

Deploy the application and open it in a mobile chrome browser. You should see a camera view.

```html
<!DOCTYPE html>
<html>
<head>
<title>AR.js A-Frame Location-based</title>
<script src="https://aframe.io/releases/1.3.0/aframe.min.js"></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/three.js/build/ar-threex-location-only.js'></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js'></script>

<body>
    <a-scene 
        vr-mode-ui='enabled: false' 
        arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false' 
        renderer='antialias: true; alpha: true'
    >
        <a-camera gps-new-camera='gpsMinDistance: 5'></a-camera>
    </a-scene>
</body>
</html>
```

# 2) Add an Entity

Let's add an entity to the scene. Use the `gps-new-entity-place` attribute to specify the latitude and longitude of where to place the entity. Replace `YOUR_LATITUDE` and `YOUR_LONGITUDE` in the below code with actual latitude and longitude values near you.

Now when you run the application, you should see a red box near you. The entity might not be directly in front of you, so you might need to look around you in multiple directions.

```html
<a-entity 
    material='color: red' 
    geometry='primitive: box' 
    gps-new-entity-place="latitude: YOUR_LATITUDE; longitude: YOUR_LONGITUDE" 
    scale="200 200 200" 
></a-entity>
```

# 3) Make the Entity clickable

Update the `<a-scene>` to include a `cursor`. The cursor is how to interact with entities

```html
<a-scene 
    vr-mode-ui='enabled: false' 
    arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false' 
    renderer='antialias: true; alpha: true'
    cursor='rayOrigin: mouse'
    raycaster='near: 0; far: 50000'
>
```

Add a new attribute to the entity called "clickable". "Clickable" is not a reserved or special word, you can use any name.

```html
<a-entity 
    material='color: red' 
    geometry='primitive: box' 
    gps-new-entity-place="latitude: YOUR_LATITUDE; longitude: YOUR_LONGITUDE" 
    scale="200 200 200"
    clickable 
></a-entity>
```

Finally we need to add an event listener for click events on the entity. Create an inline script within the head tags.

```html
<script>
    AFRAME.registerComponent('clicker', {
        init: function() {
            this.el.addEventListener('click', () => {
                alert("clicked");
            });
        }
    });
</script>
```
Now when you run the application and click on the red box, you will see a popup.

# Full Code:

```html
<!DOCTYPE html>
<html>
<head>
<title>AR.js A-Frame Location-based</title>
<script src="https://aframe.io/releases/1.3.0/aframe.min.js"></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/three.js/build/ar-threex-location-only.js'></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js'></script>
<script>
    AFRAME.registerComponent('clicker', {
        init: function() {
            this.el.addEventListener('click', () => {
                alert("clicked");
            });
        }
    });
</script>
</head>
<body>
    <a-scene 
        vr-mode-ui='enabled: false' 
        arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false' 
        renderer='antialias: true; alpha: true'
        cursor='rayOrigin: mouse'
        raycaster='near: 0; far: 50000'
    >
        <a-camera gps-new-camera='gpsMinDistance: 5'></a-camera>
        <a-entity 
            material='color: red' 
            geometry='primitive: box' 
            gps-new-entity-place="latitude: YOUR_LATITUDE; longitude: YOUR_LONGITUDE" 
            scale="200 200 200" 
            clicker
        ></a-entity>
    </a-scene>
</body>
</html>
```
# Optional: Dynamically generate Entity location

Instead of hardcoding the entity's location, we can get the lat and long coordinates from the `gps-new-camera`. 

The below code listens for `gps-camera-update-position` event to get the current coordinates and then dynamically adds an entity to the scene.

```html
<!DOCTYPE html>
<html>
<head>
<title>AR.js A-Frame Location-based</title>
<script src="https://aframe.io/releases/1.3.0/aframe.min.js"></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/three.js/build/ar-threex-location-only.js'></script>
<script type='text/javascript' src='https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js'></script>
<script>
    AFRAME.registerComponent('clicker', {
        init: function() {
            this.el.addEventListener('click', () => {
                alert("clicked");
            });
        }
    });
</script>
</head>
<body>
    <a-scene 
        vr-mode-ui='enabled: false' 
        arjs='sourceType: webcam; videoTexture: true; debugUIEnabled: false' 
        renderer='antialias: true; alpha: true'
        cursor='rayOrigin: mouse'
        raycaster='near: 0; far: 50000'
    >
        <a-camera gps-new-camera='gpsMinDistance: 5'></a-camera>
    </a-scene>
</body>
<script>
    // Add an entity when the camera updates its GPS position
    let added_entity = false;
    const el = document.querySelector("[gps-new-camera]");
    el.addEventListener("gps-camera-update-position", e => {
        // only add entity once
        if(!added_entity) {
            const entity = document.createElement("a-box");
            entity.setAttribute('material', { color: 'red' } );
            entity.setAttribute('clicker', '');
            entity.setAttribute("scale", "100 100 100");
            entity.setAttribute('gps-new-entity-place', {
                latitude: e.detail.position.latitude,
                longitude: e.detail.position.longitude
            });
            document.querySelector("a-scene").appendChild(entity);
            added_entity = true;
        }
    })
</script>
</html>

```