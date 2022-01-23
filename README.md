
exploring the basics of 3js - exploring scenes, cameras, rendering, and page dynamics. 
# My first-3js-scene

Exploring the basics of 3js - exploring scenes, cameras, rendering, and page dynamics. 

## To run project

clone the github repo
run "npm install" from the parent directory to install dependencies.
run "npm run dev" - Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

### Notes and walk through on how this project was set up

Add canvas into index.html 
Add styling for canvas with position=fixed, top=0, left=0 to pin in top left corner 



Import three into main.js

### We will always need 3 things:
Scene
Camera
renderer


#### A scene is like a container that holds your objects, cameras and lights:
const scene = new THREE.Scene(); 


#### Next need a camera, and there are many in three js. Most common is perspective camera that #mimics what human eye balls see
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000);
→ Args (field of view, aspect ration (use browser to calc), and view frustrum)


#### Next, render == draw
const renderer = new THREE.WebGLRenderer({
  canvas: document.querySelector('#bg'),
})

#### set pixel ratio to device pixel ration
renderer.setPixelRatio( window.devicePixelRatio );
#### set render size to device browser size
renderer.setSize( window.innerWidth, window.innerHeight );
#camera starts in middle so this moves it along the z axis
camera.position.setZ(30);
#call render method passing out scene and camera 
renderer.render( scene, camera )


#### now we just have blank space, so lets add an object 

#### geometry (set of vectors - x,y,z)
const geometry = new THREE.TorusGeometry( 10, 3, 16, 100 );
#### material, color or texture, - can write custom shaders with webgl
#### -most materials require light, but we use a basic that does not 
const material = new THREE.MeshBasicMaterial( { color: 0xFF6347, wireframe: true } );
#### mesh - combine geoterey with material 
const torus = new THREE.Mesh( geometry, material );

scene.add(torus)

#### We dont want to call render over and over again, so best to set a recursive function in an         # infinite loop that calls render automatically (also called game loop)
function animate() {
  requestAnimationFrame( animate );
  renderer.render( scene, camera );
}

animate()

#### now we can see a static torus in web page 

#### now we want it to move 
#Every shape we make has different properties like position, rotation and scale 
function animate() {
  requestAnimationFrame( animate );

  torus.rotation.x += 0.01;
  torus.rotation.y += 0.005;
  torus.rotation.z += 0.01;

  renderer.render( scene, camera );
}

#### next lets look at lighting

#### change material from basic to standard  - this reacts to light 
const material = new THREE.MeshStandardMaterial( { color: 0xFF6347 } );

#### many types of light, but add all direction white basic point light
const pointLight = new THREE.PointLight(0xffffff)
#### set position to remove from center 
pointLight.position.set(20,20,20)


#### add ambient light to see general env light 
const pointLight = new THREE.PointLight(0xffffff)
pointLight.position.set(20,20,20)
 
const ambientLight = new THREE.AmbientLight(0xffffff)
scene.add(pointLight, ambientLight)


#### can use lighthelper to show position and direction of light source <3 and grid helper to show  
#### 2D grid along scene
const lightHelper = new THREE.PointLightHelper(pointLight)
const gridHelper = new THREE.GridHelper(200,50);
 
scene.add(lightHelper, gridHelper)

#### make scene more interactive by adding orbit controls
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
 

#### then
const controls = new OrbitControls(camera, renderer.domElements);
#### this listens to dom events on mouse and updates camera position accordingly. 

#### add controls.update to loop
function animate() {
 requestAnimationFrame( animate );
 
 torus.rotation.x += 0.01;
 torus.rotation.y += 0.005;
 torus.rotation.z += 0.01;
 
 controls.update();
 
 renderer.render( scene, camera );
}


#### now you can use mouse to navigate in browser

#### next use math to generate a large amount of objects - stars in our case
#### we will create star function, make a sphere with .25 radius
function addStar() {
 const geometry = new THREE.SphereGeometry(0.25);
 const material = new THREE.MeshStandardMaterial( { color: 0xffffff } )
 const star = new THREE.Mesh( geometry, material );
}

#### but now how to we randomly position
#### filling array with 3 values and map each value to three js helper function randFloatSpread
function addStar() {
 const geometry = new THREE.SphereGeometry(0.25);
 const material = new THREE.MeshStandardMaterial( { color: 0xffffff } )
 const star = new THREE.Mesh( geometry, material );
 
 const [x,y,z] = Array(3).fill().map(() => THREE.MathUtils.randFloatSpread( 100 ) );
 
 star.position.set(x, y, z);
 scene.add(star)
}

#### now just a matter of deciding how many stars we want to add to the scene 
#### make 200 stars - create array of 200 values and for each value, call addStar function
Array(200).fill().forEach(addStar)


#### next lets add background texture 
// can also use a callback function - useful if wana show a loading bar if scene relies on a lot of static assets
const spaceTexture = new THREE.TextureLoader().load('space.jpeg');
scene.background = spaceTexture;


#### you can also add textures using texture mapping - simple example - 
#### 3D cube with image mapped to all 6 side 
// avatar
const avatarTexture = new THREE.TextureLoader().load('jeff.png');
 
const avatar = new THREE.Mesh(
 new THREE.BoxGeometry(3,3,3),
 new THREE.MeshBasicMaterial( { map: avatarTexture } )
);
 
scene.add(avatar)

#### now we can do more cool
#### add normal map to give texture appears of dips and grooves and have light bounce off
// moon
const moonTexture = new THREE.TextureLoader().load('moon.jpeg');
const normalTexture = new THREE.TextureLoader().load('normal.jpeg');
 
const moon = new THREE.Mesh(
 new THREE.SphereGeometry(3, 32, 32),
 new THREE.MeshStandardMaterial( {
   map: moonTexture,
   normalMap: normalTexture
 } )
);
#### appearance of depth, a normal map can make an object feel much more realistic 
#### with little effort

#### next, lets start adding some scroll animation 
#### first added markup in index.html - point is for text to take up a lot of vertical space
#### add position = absolute for the main element in styles

#### move moon down page
// both these position set suyntax styles do same thing
moon.position.z = 30;
moon.position.setX(-10);

#### create moveCamera function and set that to the document.body.onscroll event
function moveCamera() {
 
}
 
#### fires function every time user scrolls
document.body.onscroll = moveCamera



#### now inside the function 
#### caloc where user is currently scroller to - top property shows us how far we are from top 
#### rotate moon and avatar
#### most important is change position of camera - top value will always be neg so * by neg
function moveCamera() {
 
 const t = document.body.getBoundingClientRect().top;
 moon.rotation.x += 0.05;
 moon.rotation.y += 0.075;
 moon.rotation.z += 0.05;
 
 avatar.rotation.y += 0.01;
 avatar.rotation.z += 0.01;
 
 camera.position.z = t * -0.01;
 camera.position.x = t * -0.0002;
 camera.position.y = t * -0.0002;
 
}
 
document.body.onscroll = moveCamera



#### finish up with using CSS grid in styles to make it easy 
