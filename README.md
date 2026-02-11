# Luna-Aitana
Mi mundo
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Luna Aitana - Mi Universo</title>

<style>
body{margin:0;overflow:hidden;background:black;font-family:Arial;color:white;}
#entrada{
position:fixed;width:100%;height:100%;background:black;
display:flex;flex-direction:column;justify-content:center;align-items:center;
z-index:20;text-align:center;
}
#entrada h1{font-size:26px;margin-bottom:30px;}
#entrada button{
padding:15px 30px;font-size:18px;border:none;border-radius:12px;
background:#ff66b2;color:white;
}
#mensajeCentral{
position:fixed;top:20px;width:100%;text-align:center;
font-size:22px;z-index:5;pointer-events:none;opacity:0;transition:2s;
}
#modal{
position:fixed;width:100%;height:100%;background:rgba(0,0,0,0.95);
display:none;justify-content:center;align-items:center;
flex-direction:column;z-index:30;
}
#modal img,#modal video{
max-width:90%;max-height:80%;border-radius:15px;
}
#cerrar{
margin-top:20px;padding:10px 20px;background:#ff66b2;
border:none;color:white;border-radius:10px;font-size:16px;
}
</style>
</head>

<body>

<div id="entrada">
<h1>¿Quieres conocer mi universo?</h1>
<button onclick="iniciar()">Toca para abrir</button>
</div>

<div id="mensajeCentral">✨ Mi mundo entero ✨</div>

<div id="modal">
<div id="contenidoModal"></div>
<button id="cerrar" onclick="cerrarModal()">Cerrar</button>
</div>

<audio id="musica" loop>
<source src="musica.mp3" type="audio/mp3">
</audio>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/loaders/FontLoader.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/geometries/TextGeometry.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/examples/js/controls/OrbitControls.js"></script>

<script>

let scene,camera,renderer,controls;
let objetos=[];
let raycaster=new THREE.Raycaster();
let mouse=new THREE.Vector2();
let clickable=[];

function iniciar(){
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
camera.position.z=1800;

renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth,window.innerHeight);
document.body.appendChild(renderer.domElement);

controls=new THREE.OrbitControls(camera,renderer.domElement);
controls.enableZoom=true;
controls.enableDamping=true;
controls.enabled=false;

/* Estrellas */
const geo=new THREE.BufferGeometry();
const mat=new THREE.PointsMaterial({color:0xffffff,size:1});
let vertices=[];
for(let i=0;i<20000;i++){
vertices.push(
THREE.MathUtils.randFloatSpread(5000),
THREE.MathUtils.randFloatSpread(5000),
THREE.MathUtils.randFloatSpread(5000)
);
}
geo.setAttribute('position',new THREE.Float32BufferAttribute(vertices,3));
scene.add(new THREE.Points(geo,mat));

/* Crear FOTO */
function planetaFoto(img,size,dist,speedRot){
const tex=new THREE.TextureLoader().load(img);
const mesh=new THREE.Mesh(
new THREE.SphereGeometry(size,32,32),
new THREE.MeshBasicMaterial({map:tex})
);
mesh.userData={tipo:"foto",src:img};
const pivot=new THREE.Object3D();
scene.add(pivot);
pivot.add(mesh);
mesh.position.x=dist;
objetos.push({pivot,speedRot});
clickable.push(mesh);
}

/* Crear VIDEO (no reproduce hasta tocar) */
function planetaVideo(url,size,dist,speedRot){
const tex=new THREE.TextureLoader().load("video-placeholder.jpg"); // opcional miniatura
const mesh=new THREE.Mesh(
new THREE.SphereGeometry(size,32,32),
new THREE.MeshBasicMaterial({map:tex})
);
mesh.userData={tipo:"video",src:url};
const pivot=new THREE.Object3D();
scene.add(pivot);
pivot.add(mesh);
mesh.position.x=dist;
objetos.push({pivot,speedRot});
clickable.push(mesh);
}

/* 9 FOTOS */
planetaFoto("foto1.jpg",12,120,0.006);
planetaFoto("foto2.jpg",12,140,0.006);
planetaFoto("foto3.jpg",12,160,0.006);

planetaFoto("foto4.jpg",14,200,0.004);
planetaFoto("foto5.jpg",12,220,0.004);
planetaFoto("foto6.jpg",12,240,0.004);

planetaFoto("foto7.jpg",14,280,0.002);
planetaFoto("foto8.jpg",12,300,0.002);
planetaFoto("foto9.jpg",12,320,0.002);

/* 4 VIDEOS */
planetaVideo("video1.mp4",18,380,0.0015);
planetaVideo("video2.mp4",18,420,0.0015);
planetaVideo("video3.mp4",18,460,0.0015);
planetaVideo("video4.mp4",18,500,0.0015);

/* Nombre Luna Aitana */
const loader=new THREE.FontLoader();
loader.load('https://threejs.org/examples/fonts/helvetiker_regular.typeface.json',
function(font){
const text=new THREE.Mesh(
new THREE.TextGeometry("Luna Aitana",{font:font,size:10,height:2}),
new THREE.MeshBasicMaterial({color:0xff99cc})
);
const pivot=new THREE.Object3D();
scene.add(pivot);
pivot.add(text);
text.position.x=650;
objetos.push({pivot,speedRot:0.001});
});

/* CLICK */
renderer.domElement.addEventListener("click",function(event){
mouse.x=(event.clientX/window.innerWidth)*2-1;
mouse.y=-(event.clientY/window.innerHeight)*2+1;
raycaster.setFromCamera(mouse,camera);
const intersects=raycaster.intersectObjects(clickable);
if(intersects.length>0){
const obj=intersects[0].object;
if(obj.userData.tipo==="foto"){
abrirModal(`<img src="${obj.userData.src}">`);
}
if(obj.userData.tipo==="video"){
abrirModal(`<video src="${obj.userData.src}" controls autoplay></video>`);
}
}
});

window.addEventListener("resize",()=>{
camera.aspect=window.innerWidth/window.innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(window.innerWidth,window.innerHeight);
});
}

/* MODAL */
function abrirModal(contenido){
document.getElementById("contenidoModal").innerHTML=contenido;
document.getElementById("modal").style.display="flex";
}
function cerrarModal(){
document.getElementById("modal").style.display="none";
document.getElementById("contenidoModal").innerHTML="";
}

/* ANIMACIÓN */
function animate(){
requestAnimationFrame(animate);

if(camera.position.z>350){
camera.position.z-=12; // viaje cinematográfico
}else{
controls.enabled=true;
document.getElementById("mensajeCentral").style.opacity=1;
}

objetos.forEach(o=>{o.pivot.rotation.y+=o.speedRot;});
controls.update();
renderer.render(scene,camera);
}

</script>
</body>
</html>
