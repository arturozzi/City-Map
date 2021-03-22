## Configuration of requirements
<br>

First of all we need a **Scene**. To call it up we use the three.js library, via the THREE.Scene() function. We will write it inside the main init() function

```javascript
// Main Function
const init = () =>
{
    // Create a Scene
    const scene = new THREE.Scene();
}

init();
```

As next step we need to instantiate a **Camera**. we are usign the **Perspective Camera** that is designed to mimic the way the human eye sees

 As parameters we insert the Field of View(how large the perspective on the image can be in degrees), the Aspect Ratio(the ratio between the camera and the screen) and  the Near Clip and Far Clip (at what minimum and maximum distance from the camera the objects in the scene can be seen)

  can initially set the camera at a predefined distance from the centre as follows:


```javascript
// Create a camera
const camera = new THREE.PerspectiveCamera(45, window.innerWidth/ window.innerHeight, 1, 10000);
    
    
camera.position.set(0.5, 0.5, 5);
```

Now it is time to install the **Renderer** 

The anti-aliasing object is inserted as a parameter. Antialiasing is used to soften lines, smooth edges and improve the image. 

 We need then to set the size of the renderer in our page, set the background colors and call it up in the html page via the JavaScript function getElementbyId


 ```javascript
// renderer
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize( window.innerWidth, window.innerHeight );
renderer.setClearColor('#000000');


document.getElementById('webgl').appendChild(renderer.domElement);
```

To keep the render up to date we create a function called **update()** where we pass as arguments the Renderer, the Scene and the Camera. 
Inside the function we call the render method of the Renderer using the Scene and the Camera as arguments.
Then, still within the update() function, we insert a recursive method, the **requestAnimationFrame**, that **constantly calls the Renderer, Scene and Camera**

```javascript
const update = (renderer, scene, camera) =>
{
    renderer.render(scene, camera);

    requestAnimationFrame(function() {
        update(renderer, scene, camera);
    });
}
```
```javascript
update(renderer, scene, camera);
```

**OrbitControls** is a function provided by Three.js that allows, in a simple and immediate manner, to move the camera around the scene and to zoom in or out on the objects. We need to instanziate it and call as parameters the Camera and the renderer.domElement

```javascript
var controls = new THREE.OrbitControls(camera, renderer.domElement);

update(renderer, scene, camera, controls);
```

remember to add the Orbit Controls also in the parameters of the update function and as an argument within the function

```javascript
const update = (renderer, scene, camera, controls) =>
{
    renderer.render(scene, camera, controls);

    controls.update();

    requestAnimationFrame(function() {
        update(renderer, scene, camera, controls);
    });
}
```

To create a **building**, it is necessary to install its **geometry and material**, join these two objects using the **mesh()** function and then add them to the scene

To make the code cleaner, we can create the buildings in a separate function 

```javascript
// Buildings
const getBox = ( w, h, d ) =>
{
    const boxGeometry = new THREE.BoxBufferGeometry(w, h, d);
    const boxMaterial = new THREE.MeshBasicMaterial({ color: '#e6bb45'});
    const mesh = new THREE.Mesh(boxGeometry, boxMaterial);
    return mesh;        
}
```

**remeber to call it always in the main function an to add it to the scene**
```javascript
 // Add Objects to the Scene and set the position
const box = getBox(1, 1, 1);
scene.add(box);
```
Until the lights are mounted, use the **MeshBasicMaterial**,  which does not need lights to be seen

The geometry of the buildings has three parameters: heigth, width and depth


Now let's use another function to add the **layers**
```javascript
// Soil
const getSoil = ( w, h, d ) =>
{
    const layerGeometry = new THREE.BoxBufferGeometry(w, h, d);
    const layerMaterial = new THREE.MeshBasicMaterial({ color: '#1239a3'}); 
    const mesh = new THREE.Mesh(layerGeometry, layerMaterial);
    return mesh;
}
```

remeber to call it always in the main function an to add it to the scene
```javascript
const soil = getSoil(5, 0.15, 6);
scene.add(soil);
```

The building and the layer must be arranged so that one rests on the other.

We must configure the position on the Y-axis (the one that measures height) and the value must be equal to half its height, because by default it is oriented exactly in the centre of the axis vertexes

```javascript
const box = getBox(1, 1, 1);
    box.position.y = 1 / 2;
    scene.add(box);
```

We must create lights that allow us to see the objects with Standard Material, which is the material who we are looking for

We will create three lights. Two **directional lights** and one **ambient light**.

We compose a function for the ambient light.

```javascript
// AmbientLight
const getAmbientLight = (intensity) =>
{
    const ambientL = new THREE.AmbientLight('#ffffff', intensity);
    return ambientL;
}
```

and two other for the directional Lights

```javascript
// I directionalLightight
const getDirectionalLight = (intensity) =>
{
    const directionalLight = new THREE.DirectionalLight('#ffffff', intensity);
    return directionalLight;
}

// II directionalLightight
const getDirectionalLight2 = (intensity) =>
{
    const directionalLight2 = new THREE.DirectionalLight('#ffffff', intensity);
    return directionalLight2;
}
```

Dont't forget to call it in the main function, init(), and to set it with the followings **intensity**

```javascript
 // Lights
const ambientL = getAmbientLight(0.5);
scene.add(ambientL);

const directionalLight = getDirectionalLight(1);
scene.add(directionalLight);
    
const directionalLight2 = getDirectionalLight2(0.5);
scene.add(directionalLight2);
```
Now we can change Material of the building and layer and look how the **Standard** effect look like

```javascript
const boxMaterial = new THREE.MeshStandardMaterial({ color: '#e6bb45'});

const layerMaterial = new THREE.MeshStandardMaterial({ color: '#1239a3'}); 
```
We have to set the Directionals Lights in the right positions

The Ambient light doesn't need any position


```javascript
directionalLight.position.set(-100, 70, 100);

directionalLight2.position.set(100, 70, 30);
```

at this point we can set some **shadows**

Shadows are activated with three.js in a special way

First of all, they must be **activated in the renderer by setting the ShadowMap** function to true
```javascript
...
renderer.setClearColor('#000000');
renderer.shadowMap.enabled = true;
...
``` 

we must **enable light to cast shadows**
```javascript
...
const directionalLight = new THREE.DirectionalLight('#ffffff', intensity);
directionalLight.castShadow = true;
...
``` 

we need to **enable the building to cast shadows** 
```javascript
...
const mesh = new THREE.Mesh(boxGeometry, boxMaterial);
mesh.castShadow = true;...
``` 

we need to **enable the layer to receive shadows** 
```javascript
...
const mesh = new THREE.Mesh(layerGeometry, layerMaterial);
mesh.receiveShadow = true;
...
``` 

A good way to continue is to set up the **shadow quality** 

The Shadow qualtiy must be set in a value that is a multiple of 2, other ways the renderer could bring to some bugs

```javascript
...
directionalLight.position.set(-100, 70, 100);
directionalLight.shadow.mapSize.width = 2048;
directionalLight.shadow.mapSize.height = 2048;
...
``` 

The real problem with the shadow appear when we want to make them less intense

The opacity of shadows in three.js is unrelated to the material qualities of the objects themselves. In order to achieve the desired degree of transparency it is necessary to create a copy of the elements, layers and buildings, and overlap it. These copies have, like the other elements, geometric characteristics of height, width and depth, but in contrast to these, they do not have the material quality of color, but only the reception and reflection of shadows 

To make the problem of shadows even clearer, we create another building on which we can cast the shadow of the first one

```javascript
const box2 = getBox(1, 1, 1);
box2.position.y = 1 / 2;
box2.position.x = 1.5;
box2.position.z = -1.5;
scene.add(box2);
``` 
It can immediately be seen that the buildings receive no shadows. To do this, enable shadow reception as done on the layer

```javascript
...
mesh.castShadow = true;
mesh.receiveShadow = true;
... 
``` 
If we create a ShadowMaterial on the layer and set the degree of opacity, the shadow will still be visible but the layer will appear to have disappeared.


```javascript
const layerGeometry = new THREE.BoxBufferGeometry(w, h, d);
const layerMaterial = new THREE.ShadowMaterial({ color: '#ffffff'}); 
layerMaterial.opacity = 0.09;
``` 

to be able to continue to see the shadow we change the background color


```javascript
renderer.setClearColor('#f5f5f5');
``` 

if we want to see light obra on buildings as well, they will disappear.

```javascript
const boxMaterial = new THREE.ShadowMaterial({ color: '#e6bb45'});
    boxMaterial.opacity = 0.09;
``` 
here the results

<img width="100%" src="../fotoConfigurationRequirement/layerinMaterialShadow.jpg"/>
<br><br>
<img width="100%" src="../fotoConfigurationRequirement/buildingsandlayerinMaterialShadow.jpg"/>


Let's try creating clones, used only for shadows 
First delete the last steps on building and layers
We need just one type of building that cast and receive shadows

```javascript
// Buildings
const getBox = ( w, h, d ) =>
{
    const boxGeometry = new THREE.BoxBufferGeometry(w, h, d);
    const boxMaterial = new THREE.MeshStandardMaterial({ color: '#e6bb45'});
    const mesh = new THREE.Mesh(boxGeometry, boxMaterial);
    return mesh;        
}

// ShadowBuildings
const getShadowBox = ( w, h, d ) =>
{
    const boxGeometry = new THREE.BoxBufferGeometry(w, h, d);
    const boxMaterial = new THREE.ShadowMaterial();
    boxMaterial.opacity = 0.09;
    const mesh = new THREE.Mesh(boxGeometry, boxMaterial);
    mesh.castShadow = true;
    mesh.receiveShadow = true;
    return mesh;        
}
``` 
```javascript
// buildings in main function 
const shadowBox = getShadowBox(1, 1, 1);
    shadowBox.position.y = 1 / 2;
    shadowBox.position.z = 1;
    scene.add(shadowBox);

    const shadowBox2 = getShadowBox(1, 1, 1);
    shadowBox2.position.y = 1 / 2;
    shadowBox2.position.x = 1.5;
    shadowBox2.position.z = -0.5;
    scene.add(shadowBox2);
```
We need just one type of layer that cast and receive shadows and for the Shadow we can use a **plane geometry**

```javascript
// Plane
const getPlane = ( w, d ) =>
{
    const geometry = new THREE.PlaneBufferGeometry( w, d );
    const material = new THREE.ShadowMaterial();
    material.opacity = 0.09;
    const mesh = new THREE.Mesh(geometry, material);
    mesh.receiveShadow = true;
    mesh.castShadow = true;
    return mesh;
}
// Soil
const getSoil = ( w, h, d ) =>
{
    const layerGeometry = new THREE.BoxBufferGeometry(w, h, d);
    const layerMaterial = new THREE.MeshStandardMaterial({ color: '#ffffff'}); 
    const mesh = new THREE.Mesh(layerGeometry, layerMaterial);
    return mesh;
}
``` 

```javascript
// layers in main function
const soil = getSoil(5, 0.15, 7);
scene.add(soil);

const plane = getPlane(5, 7);
scene.add(plane);
``` 
Now we have to position the new layer in a correct way and mouve just a litle bit downwards the big layer

```javascript
const plane = getPlane(5, 7);
plane.rotation.x = -(Math.PI / 2);
scene.add(plane);
``` 

```javascript
soil.position.set(0, -0.08, 0);
``` 
and here the correct Shadows result

<img width="100%" src="../fotoConfigurationRequirement/correctShadows.jpg"/>

```javascript

``` 
____



<!-- ```javascript

```  -->




### Buildings
The material of the buildings has been defined as Standard Material.

The MeshStandardMaterial is the type of material, provided by the Three.js library, which allows a more accurate result with regard to the interaction of the object with light and shadow.

Has to have the following parameters: metalness = 0.085, roughness = 0.05

The parameters of Metalness and Roughness make the object sensitive to light in different ways depending on the values used, ranging from 0 to 1

### layers

The heigth for each layer has been defined in the value of: 0.15 

The color for each layer has been defined in the value of '#f5f5f5' 

and with the followiong parameters, metalness: 0.225, roughness: 0.05

### camera
Once we have defined the buildings and the method that manages their number, we can add the following function

The Camera position varies in relation to the number of buildings. 

A function external to the main one takes care of the camera position according to the inserted buildings, here are the values used in the function


camera position values of up to 750 buildings: 30, 41, 30 (x, y, z)

camera position values of up to 3200 buildings: 30, 52, 80 (x, y, z)

camera position values of up to 6000 buildings: 80, 70, 90 (x, y, z)

camera position values of up to 10000 buildings: 110, 70, 90 (x, y, z)


```javascript
    
// Camera position
const getCameraPositionX = (a) =>
{
    if (a <= 750)
    {
        return 30;        
    }
    else if (a <= 3200)
    {   
        return 30;        
    }
    else if (a <= 6000)
    {
        return 80;        
    }
    else 
    {
        return 110;    
    }
}

const getCameraPositionY = (a) =>
{
    if (a <= 750)
    {
        return 41;        
    }
    else if (a <= 3200)
    {
        return 52;        
    }
    else 
    {
        return 70;        
    }
}

const getCameraPositionZ = (a) =>
{
    if (a <= 750)
    {
        return 30;        
    }
    else if (a <= 3200)
    {
        return 80;
    }
    else 
    {
        return 90;        
    }
}
```


#### Lights



The ambient light does not need any positioning 

The intensity of the ambient light has been defined in the values of: 0.5 

The intensity of the first directional light has been defined in the value of: 0.55 

The intenstiy of the second directional light has been defined in the value of: 0.5 

<br>

### Shadows


The intensity of the Shadow has been defined in the values of: