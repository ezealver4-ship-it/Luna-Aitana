# Luna-Aitana
Mi mundo
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Luna Aitana</title>
<style>
body{margin:0;overflow:hidden;background:black;font-family:Arial;color:white}
#entrada{
position:fixed;width:100%;height:100%;
background:black;display:flex;
flex-direction:column;justify-content:center;
align-items:center;z-index:10;text-align:center;
}
#entrada h1{font-size:24px;margin-bottom:25px}
#entrada button{
padding:15px 30px;font-size:18px;
border:none;border-radius:10px;
background:#ff66b2;color:white
}
#mensaje{
position:fixed;top:20px;width:100%;
text-align:center;font-size:22px;
opacity:0;transition:2s;
pointer-events:none
}
#modal{
position:fixed;width:100%;height:100%;
background:rgba(0,0,0,0.95);
display:none;justify-content:center;
align-items:center;flex-direction:column;
z-index:20
}
#modal img,#modal video{
max-width:90%;max-height:80%;
border-radius:15px
}
#cerrar{
margin-top:20px;padding:10px 20px;
background:#ff66b2;border:none;
color:white;border-radius:10px
}
</style>
</head>
<body>

<div id="entrada">
<h1>¿Quieres conocer mi universo?</h1>
<button onclick="start()">Toca para abrir</button>
</div>

<div id="mensaje">✨ Mi mundo entero ✨</div>

<div id="modal">
<div id="contenido"></div>
<button id="cerrar" onclick="cerrar()">Cerrar</button>
</div>

<audio id="musica" loop src="musica.mp3"></audio>

<script type="module">

import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.module.js";
import { OrbitControls } from "https://cdn.jsdelivr.net/npm/three@0.158.0/examples/jsm/controls/OrbitControls.js";

let scene,camera,renderer,controls;
let objetos=[];
let raycaster=new THREE.Raycaster();
let mouse=new THREE.Vector2();
let clickable=[];

window.start=function(){
document.getElementById("entrada").style.display="none";
document.getElementById("musica").play();
init();
animate();
}

function init(){

scene=new THREE.Scene();

camera=new THREE.PerspectiveCamera(
75,window.innerWidth/window.innerHeight,0.1,5000
);
camera.position.z=1500;

renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth,window.innerHeight);
document.body.appendChild(renderer.domElement);

controls=new OrbitControls(camera,renderer.domElement);
controls.enableZoom=true;
controls.enableDamping=true;
controls.enabled=false;

/* estrellas */
const geo=new THREE.BufferGeometry();
let vertices=[];
for(let i=0;i<15000;i++){
vertices.push(
THREE.MathUtils.randFloatSpread(5000),
THREE.MathUtils.randFloatSpread(5000),
THREE.MathUtils.randFloatSpread(5000)
);
}
geo.setAttribute("position",new THREE.Float32BufferAttribute(vertices,3));
const stars=new THREE.Points(geo,new THREE.PointsMaterial({color:0xffffff,size:1}));
scene.add(stars);

/* función foto */
function addFoto(src,size,dist,vel){
const tex=new THREE.TextureLoader().load(src);
const mesh=new THREE.Mesh(
new THREE.SphereGeometry(size,32,32),
new THREE.MeshBasicMaterial({map:tex})
);
mesh.userData={tipo:"foto",src};
const pivot=new THREE.Object3D();
scene.add(pivot);
pivot.add(mesh);
mesh.position.x=dist;
objetos.push({pivot,vel});
clickable.push(mesh);
}

/* función video */
function addVideo(src,size,dist,vel){
const tex=new THREE.TextureLoader().load("foto1.jpg");
const mesh=new THREE.Mesh(
new THREE.SphereGeometry(size,32,32),
new THREE.MeshBasicMaterial({map:tex})
);
mesh.userData={tipo:"video",src};
const pivot=new THREE.Object3D();
scene.add(pivot);
pivot.add(mesh);
mesh.position.x=dist;
objetos.push({pivot,vel});
clickable.push(mesh);
}

/* 9 fotos */
for(let i=1;i<=9;i++){
addFoto(`foto${i}.jpg`,12,100+i*40,0.002+(i*0.0002));
}

/* 4 videos */
for(let i=1;i<=4;i++){
addVideo(`video${i}.mp4`,18,500+i*60,0.001);
}

/* click */
renderer.domElement.addEventListener("click",e=>{
mouse.x=(e.clientX/window.innerWidth)*2-1;
mouse.y=-(e.clientY/window.innerHeight)*2+1;
raycaster.setFromCamera(mouse,camera);
let inter=raycaster.intersectObjects(clickable);
if(inter.length>0){
let obj=inter[0].object;
if(obj.userData.tipo==="foto"){
abrir(`<img src="${obj.userData.src}">`);
}
if(obj.userData.tipo==="video"){
abrir(`<video src="${obj.userData.src}" controls autoplay></video>`);
}
}
});

window.addEventListener("resize",()=>{
camera.aspect=window.innerWidth/window.innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(window.innerWidth,window.innerHeight);
});
}

function abrir(html){
document.getElementById("contenido").innerHTML=html;
document.getElementById("modal").style.display="flex";
}
window.cerrar=function(){
document.getElementById("modal").style.display="none";
document.getElementById("contenido").innerHTML="";
}

function animate(){
requestAnimationFrame(animate);

if(camera.position.z>350){
camera.position.z-=8;
}else{
controls.enabled=true;
document.getElementById("mensaje").style.opacity=1;
}

objetos.forEach(o=>o.pivot.rotation.y+=o.vel);
controls.update();
renderer.render(scene,camera);
}

</script>
</body>
</html>
