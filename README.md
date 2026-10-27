[index.html](https://github.com/user-attachments/files/34001739/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>MECH STRIKE — 3D Robot Combat</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#080e18;color:white;font-family:Arial,sans-serif}
canvas{display:block}
#hud{position:fixed;inset:0;pointer-events:none}
.panel{background:#071421d9;border:1px solid #24c8df;padding:12px;clip-path:polygon(0 0,100% 0,100% 85%,90% 100%,0 100%)}
#top{position:absolute;top:16px;left:16px;right:16px;display:flex;justify-content:space-between;gap:12px}
#title{font-weight:900;color:#42e9ff;letter-spacing:3px;font-size:21px}
#stats{font-size:13px;line-height:1.7}
#healthBox{position:absolute;bottom:24px;left:24px;width:260px}
#healthText{font-weight:bold;margin-bottom:7px}
.bar{height:13px;background:#263746;border:1px solid #6c8295}
#health{height:100%;width:100%;background:linear-gradient(90deg,#1ce8a1,#a6ffcf);transition:width .15s}
#weapon{margin-top:12px;color:#5ceaff;font-weight:bold}
#crosshair{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);color:#53f5ff;font-size:28px;text-shadow:0 0 8px #00dfff}
#message{position:absolute;left:50%;top:23%;transform:translateX(-50%);text-align:center;font-weight:900;font-size:30px;color:#66f6ff;text-shadow:0 0 18px #00dfff}
#help{position:absolute;bottom:20px;right:20px;max-width:250px;font-size:12px;line-height:1.8;color:#c4d6e4}
button{pointer-events:auto;border:1px solid #48eaff;background:#062637;color:white;padding:11px 16px;font-weight:bold;cursor:pointer}
#start{position:absolute;inset:0;background:radial-gradient(ellipse at center,#173650ed,#030812f5);display:flex;align-items:center;justify-content:center;pointer-events:auto;text-align:center}
#start h1{font-size:clamp(32px,7vw,68px);letter-spacing:5px;color:#56edff;margin:0 0 10px}
#start p{color:#c0d8e7;line-height:1.8}
#start button{font-size:18px;padding:16px 35px;margin-top:12px}
#mobile{display:none;position:absolute;bottom:100px;left:12px;right:12px;justify-content:space-between;pointer-events:auto}
.pad{display:grid;grid-template-columns:repeat(3,48px);gap:5px}
.pad button{padding:10px 0;opacity:.8}
#fire{width:76px;height:76px;border-radius:50%;background:#7b1727d9;border-color:#ff6c81}
@media(max-width:700px){
 #top{top:8px;left:8px;right:8px}
 #title{font-size:16px}
 #healthBox{left:12px;bottom:12px;width:200px}
 #help{display:none}
 #mobile{display:flex}
}
</style>
</head>
<body>
<div id="hud">
 <div id="top">
  <div class="panel"><div id="title">MECH STRIKE</div><div id="stats">ELIMINATIONS: <span id="kills">0</span><br>HOSTILES: <span id="remaining">5</span></div></div>
  <div class="panel" style="text-align:right">COMBAT ZONE: 07<br><span style="color:#ff637a">● LIVE BATTLE</span><br><span id="status">SYSTEM READY</span></div>
 </div>
 <div id="crosshair">+</div>
 <div id="message"></div>
 <div id="healthBox" class="panel">
  <div id="healthText">TITAN-01 // ARMOR <span id="hpText">100%</span></div>
  <div class="bar"><div id="health"></div></div>
  <div id="weapon">⚡ PLASMA CANNON &nbsp; [LMB]</div>
 </div>
 <div id="help" class="panel">
  <b>COMBAT CONTROLS</b><br>
  W A S D — Move<br>
  MOUSE — Aim / Look<br>
  LEFT CLICK — Fire<br>
  SHIFT — Boost<br>
  R — Restart battle<br>
  Destroy every hostile mech.
 </div>
 <div id="mobile">
  <div class="pad">
   <span></span><button data-key="w">▲</button><span></span>
   <button data-key="a">◀</button><button data-key="s">▼</button><button data-key="d">▶</button>
  </div>
  <button id="fire">FIRE</button>
 </div>
</div>
<div id="start">
 <div>
  <h1>MECH STRIKE</h1>
  <p>THIRD-PERSON ROBOT COMBAT SIMULATION<br>SECTOR 07 // HOSTILE UNITS DETECTED</p>
  <p>Deploy your mech. Eliminate all five enemy robots.</p>
  <button id="play">▶ DEPLOY MECH</button>
  <p style="font-size:12px">Desktop: WASD + mouse + left click<br>Mobile: directional buttons + FIRE</p>
 </div>
</div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js';

const $=id=>document.getElementById(id);
const scene=new THREE.Scene();
scene.background=new THREE.Color(0x081421);
scene.fog=new THREE.Fog(0x081421,35,100);

const camera=new THREE.PerspectiveCamera(65,innerWidth/innerHeight,.1,250);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.outputColorSpace=THREE.SRGBColorSpace;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.2;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0x91c9ff,0x18202c,2));
const sun=new THREE.DirectionalLight(0xffffff,3);
sun.position.set(-15,30,15);sun.castShadow=true;
sun.shadow.mapSize.set(2048,2048);
sun.shadow.camera.left=-40;sun.shadow.camera.right=40;
sun.shadow.camera.top=40;sun.shadow.camera.bottom=-40;
scene.add(sun);

const mat=(color,metalness=.3,roughness=.55)=>new THREE.MeshStandardMaterial({color,metalness,roughness});
const floorMat=mat(0x263746,.25,.85);
const dark=mat(0x101d2b,.75,.35);
const steel=mat(0x465b70,.8,.3);
const cyan=mat(0x14dff5,.35,.25);
const red=mat(0xe94753,.65,.3);
const white=mat(0xcbd8e5,.65,.28);

const floor=new THREE.Mesh(new THREE.PlaneGeometry(180,180),floorMat);
floor.rotation.x=-Math.PI/2;floor.receiveShadow=true;scene.add(floor);

const grid=new THREE.GridHelper(180,90,0x2b7288,0x304552);
grid.position.y=.025;scene.add(grid);

const obstacles=[];
function box(x,y,z,sx,sy,sz,material,collide=true){
 const mesh=new THREE.Mesh(new THREE.BoxGeometry(sx,sy,sz),material);
 mesh.position.set(x,y,z);mesh.castShadow=true;mesh.receiveShadow=true;scene.add(mesh);
 if(collide)obstacles.push({x,z,hx:sx/2+.7,hz:sz/2+.7});
 return mesh;
}

// Industrial buildings and defensive barriers.
for(let i=0;i<22;i++){
 const angle=i*2.399;
 const radius=12+(i%4)*6;
 const x=Math.cos(angle)*radius,z=Math.sin(angle)*radius;
 const w=2+(i%3),h=1.8+(i%4)*.8,d=2+(i%2);
 box(x,h/2,z,w,h,d,i%3===0?steel:dark);
 box(x,h+.06,z,w+.1,.12,d+.1,i%2?cyan:red,false);
}
for(let i=-1;i<=1;i++){
 box(i*12,1.2,-9,5,2.4,1.2,steel);
 box(i*12,1.2,9,5,2.4,1.2,steel);
}
for(let i=0;i<4;i++){
 box(-27+i*18,.8,0,2,1.6,3,red);
}

// Decorative pylons.
for(let i=0;i<12;i++){
 const a=i*Math.PI/6,r=33;
 const p=new THREE.Mesh(new THREE.CylinderGeometry(.35,.6,5,6),steel);
 p.position.set(Math.cos(a)*r,2.5,Math.sin(a)*r);p.castShadow=true;scene.add(p);
 const lamp=new THREE.PointLight(0x20dfff,4,7);
 lamp.position.set(p.position.x,4.7,p.position.z);scene.add(lamp);
}

// Mech construction: original geometric robot models.
function makeMech(enemy=false){
 const root=new THREE.Group();
 const armor=enemy?red:mat(0x237a9d,.8,.3);
 const eye=enemy?mat(0xffd0a3,.3,.2):cyan;
 function part(parent,m,x,y,z,sx,sy,sz){
  const o=new THREE.Mesh(new THREE.BoxGeometry(sx,sy,sz),m);
  o.position.set(x,y,z);o.castShadow=true;o.receiveShadow=true;parent.add(o);return o;
 }
 // Legs, hips and armored torso.
 part(root,dark,0,1.05,0,1.05,.42,.62);
 part(root,armor,0,1.85,0,1.12,.95,.72);
 part(root,steel,0,2.35,-.05,.75,.3,.62);
 part(root,eye,0,2.04,.375,.55,.13,.06);
 part(root,dark,0,2.57,0,.5,.23,.45);
 for(const s of [-1,1]){
  part(root,armor,s*.68,1.92,0,.35,.7,.42);
  part(root,steel,s*.7,1.52,0,.3,.36,.35);
  part(root,dark,s*.38,.7,0,.38,.85,.42);
  part(root,armor,s*.4,.3,.08,.48,.32,.62);
  part(root,steel,s*.85,1.25,.12,.25,.45,.3);
 }
 // Twin shoulder cannon barrels.
 for(const s of [-1,1]){
  const gun=new THREE.Group();gun.position.set(s*.7,2.18,.15);root.add(gun);
  part(gun,dark,0,.12,.15,.3,.34,.48);
  part(gun,armor,0,.08,.48,.24,.24,.5);
  part(gun,eye,0,.08,.74,.12,.12,.08);
 }
 root.userData.eye=eye;
 scene.add(root);
 return root;
}

const player=makeMech(false);
player.position.set(0,0,0);

const enemies=[];
const enemyPositions=[[-13,0,-12],[14,0,-14],[-17,0,12],[17,0,13],[0,0,-23]];
for(const [x,y,z] of enemyPositions){
 const mech=makeMech(true);mech.position.set(x,y,z);
 mech.userData={hp:100,maxHp:100,alive:true,fireTimer:1+Math.random()*2,phase:Math.random()*6.28,hitFlash:0};
 enemies.push(mech);
}

// Floating health bars for enemies.
for(const e of enemies){
 const canvas=document.createElement('canvas');canvas.width=256;canvas.height=40;
 const ctx=canvas.getContext('2d');
 ctx.fillStyle='#121d2a';ctx.fillRect(0,0,256,40);
 ctx.fillStyle='#ff475b';ctx.fillRect(4,4,248,32);
 const tex=new THREE.CanvasTexture(canvas);
 const sprite=new THREE.Sprite(new THREE.SpriteMaterial({map:tex,transparent:true,depthTest:false}));
 sprite.scale.set(2.5,.39,1);sprite.position.y=3.5;e.add(sprite);
 e.userData.bar=sprite;e.userData.barCanvas=canvas;
}
const projectiles=[];
const particles=[];
let running=false,health=100,kills=0,gameOver=false;
let yaw=0,pitch=.12,shooting=false,boost=false,lastShot=0;
const keys={};
const clock=new THREE.Clock();
const raycaster=new THREE.Raycaster();
const mouse=new THREE.Vector2(0,0);

function updateBar(e){
 const c=e.userData.barCanvas,ctx=c.getContext('2d');
 ctx.clearRect(0,0,256,40);
 ctx.fillStyle='#111923';ctx.fillRect(0,0,256,40);
 ctx.fillStyle='#ff465a';ctx.fillRect(4,4,248*Math.max(0,e.userData.hp/100),32);
 ctx.strokeStyle='#fff';ctx.strokeRect(3,3,250,34);
 e.userData.bar.material.map.needsUpdate=true;
}

function projectile(start,end,owner,damage=12){
 const direction=new THREE.Vector3().subVectors(end,start).normalize();
 const mesh=new THREE.Mesh(
  new THREE.SphereGeometry(owner==='player'?.13:.16,8,8),
  new THREE.MeshBasicMaterial({color:owner==='player'?0x38f6ff:0xff4b59})
 );
 mesh.position.copy(start);scene.add(mesh);
 const light=new THREE.PointLight(owner==='player'?0x28dfff:0xff354a,2,4);
 mesh.add(light);
 projectiles.push({mesh,dir:direction,owner,damage,life:2});
}

function muzzle(){
 const start=player.position.clone().add(new THREE.Vector3(0,2,.7).applyAxisAngle(new THREE.Vector3(0,1,0),yaw));
 const dir=new THREE.Vector3(Math.sin(yaw),0,Math.cos(yaw));
 const end=start.clone().add(dir.multiplyScalar(65));
 raycaster.set(start,dir);raycaster.far=65;
 const targets=enemies.filter(e=>e.userData.alive);
 const hits=raycaster.intersectObjects(targets,true);
 if(hits.length){
  const e=targets.find(e=>{let o=hits[0].object;while(o){if(o===e)return true;o=o.parent}return false});
  if(e){
   e.userData.hp-=24;updateBar(e);
   burst(hits[0].point,0x42edff,7);
   if(e.userData.hp<=0){
    e.userData.alive=false;kills++;$('kills').textContent=kills;
    e.visible=false;burst(e.position.clone().add(new THREE.Vector3(0,1,0)),0xff6a43,28);
    $('remaining').textContent=enemies.filter(x=>x.userData.alive).length;
    if(kills===enemies.length){gameOver=true;$('message').textContent='VICTORY';}
   }
  }
 }else projectile(start,end,'player',24);
 // Always produce a visible muzzle flash.
 burst(start,0x3eeaff,3,.09);
}

function burst(pos,color,count=8,speed=.15){
 for(let i=0;i<count;i++){
  const m=new THREE.Mesh(new THREE.BoxGeometry(.07,.07,.07),new THREE.MeshBasicMaterial({color}));
  m.position.copy(pos);scene.add(m);
  particles.push({mesh:m,vel:new THREE.Vector3((Math.random()-.5)*speed*2,Math.random()*speed,(Math.random()-.5)*speed*2),life:.35+Math.random()*.45});
 }
}

function hurt(amount){
 health=Math.max(0,health-amount);
 $('health').style.width=health+'%';$('hpText').textContent=Math.ceil(health)+'%';
 if(health<=0){gameOver=true;$('message').textContent='MECH DESTROYED';}
}

function reset(){
 for(const p of projectiles)scene.remove(p.mesh);
 projectiles.length=0;
 for(const p of particles)scene.remove(p.mesh);
 particles.length=0;
 health=100;kills=0;gameOver=false;running=true;yaw=0;pitch=.12;
 player.position.set(0,0,0);player.rotation.y=0;
 for(let i=0;i<enemies.length;i++){
  const e=enemies[i];e.visible=true;e.userData.hp=100;e.userData.alive=true;
  e.userData.fireTimer=1+Math.random()*2;
  e.position.set(...enemyPositions[i]);updateBar(e);
 }
 $('health').style.width='100%';$('hpText').textContent='100%';
 $('kills').textContent='0';$('remaining').textContent=enemies.length;
 $('message').textContent='';$('status').textContent='ENGAGED';
 $('start').style.display='none';
}

$('play').addEventListener('click',()=>{
 reset();
 if(innerWidth>700)renderer.domElement.requestPointerLock?.();
});
document.addEventListener('keydown',e=>{
 keys[e.key.toLowerCase()]=true;
 if(e.key==='Shift')boost=true;
 if(e.key.toLowerCase()==='r'&&running)reset();
});
document.addEventListener('keyup',e=>{
 keys[e.key.toLowerCase()]=false;
 if(e.key==='Shift')boost=false;
});
document.addEventListener('mousemove',e=>{
 if(document.pointerLockElement===renderer.domElement){
  yaw-=e.movementX*.0025;
  pitch=THREE.MathUtils.clamp(pitch-e.movementY*.0015,-.15,.65);
 }
});
renderer.domElement.addEventListener('click',()=>{
 if(running&&innerWidth>700&&document.pointerLockElement!==renderer.domElement)
  renderer.domElement.requestPointerLock?.();
});
document.addEventListener('mousedown',e=>{if(e.button===0&&running)shooting=true});
document.addEventListener('mouseup',()=>shooting=false);
document.addEventListener('contextmenu',e=>e.preventDefault());

document.querySelectorAll('[data-key]').forEach(b=>{
 const k=b.dataset.key;
 b.addEventListener('pointerdown',e=>{e.preventDefault();keys[k]=true});
 for(const ev of ['pointerup','pointercancel','pointerleave'])
  b.addEventListener(ev,()=>keys[k]=false);
});
$('fire').addEventListener('pointerdown',e=>{e.preventDefault();shooting=true});
['pointerup','pointercancel','pointerleave'].forEach(ev=>$('fire').addEventListener(ev,()=>shooting=false));

function movePlayer(dt){
 const speed=(boost?10:6)*dt;
 const forward=new THREE.Vector3(Math.sin(yaw),0,Math.cos(yaw));
 const right=new THREE.Vector3(forward.z,0,-forward.x);
 const move=new THREE.Vector3();
 if(keys.w)move.add(forward);
 if(keys.s)move.sub(forward);
 if(keys.d)move.add(right);
 if(keys.a)move.sub(right);
 if(move.lengthSq())move.normalize().multiplyScalar(speed);
 const next=player.position.clone().add(move);
 next.x=THREE.MathUtils.clamp(next.x,-38,38);
 next.z=THREE.MathUtils.clamp(next.z,-38,38);
 const blocked=obstacles.some(o=>Math.abs(next.x-o.x)<o.hx&&Math.abs(next.z-o.z)<o.hz);
 if(!blocked)player.position.copy(next);
 player.rotation.y=yaw;
}

function updateEnemies(dt,time){
 for(const e of enemies){
  if(!e.userData.alive)continue;
  const toPlayer=new THREE.Vector3().subVectors(player.position,e.position);
  const dist=toPlayer.length();
  const targetYaw=Math.atan2(toPlayer.x,toPlayer.z);
  e.rotation.y=targetYaw;
  e.userData.phase+=dt;
  if(dist>7){
   const step=Math.min(dt*2.3,dist-6);
   const nx=e.position.x+Math.sin(targetYaw)*step;
   const nz=e.position.z+Math.cos(targetYaw)*step;
   if(!obstacles.some(o=>Math.abs(nx-o.x)<o.hx&&Math.abs(nz-o.z)<o.hz)){
    e.position.x=nx;e.position.z=nz;
   }else{
    e.rotation.y+=Math.sin(time+e.userData.phase)*.7;
   }
  }
  e.position.y=Math.sin(time*2+e.userData.phase)*.035;
  e.userData.fireTimer-=dt;
  if(dist<27&&e.userData.fireTimer<=0&&!gameOver){
   const start=e.position.clone().add(new THREE.Vector3(0,2,.7).applyAxisAngle(new THREE.Vector3(0,1,0),targetYaw));
   const end=player.position.clone().add(new THREE.Vector3(0,1,0));
   projectile(start,end,'enemy',8);
   e.userData.fireTimer=1.2+Math.random()*1.2;
  }
  e.userData.bar.lookAt(camera.position);
 }
}

function updateProjectiles(dt){
 for(let i=projectiles.length-1;i>=0;i--){
  const p=projectiles[i];p.life-=dt;
  p.mesh.position.addScaledVector(p.dir,dt*32);
  let remove=p.life<=0;
  if(p.owner==='enemy'&&p.mesh.position.distanceTo(player.position)<1.15){
   hurt(p.damage);burst(p.mesh.position,0xff5365,5);remove=true;
  }
  if(p.owner==='player'){
   for(const e of enemies){
    if(e.userData.alive&&p.mesh.position.distanceTo(e.position.clone().add(new THREE.Vector3(0,1.5,0)))<1){
     e.userData.hp-=p.damage;updateBar(e);remove=true;
     if(e.userData.hp<=0){
      e.userData.alive=false;e.visible=false;kills++;
      $('kills').textContent=kills;
      $('remaining').textContent=enemies.filter(x=>x.userData.alive).length;
      burst(e.position.clone().add(new THREE.Vector3(0,1,0)),0xff6444,24);
      if(kills===enemies.length){gameOver=true;$('message').textContent='VICTORY';}
     }
     break;
    }
   }
  }
  if(remove){scene.remove(p.mesh);projectiles.splice(i,1)}
 }
}

function updateParticles(dt){
 for(let i=particles.length-1;i>=0;i--){
  const p=particles[i];p.life-=dt;
  p.mesh.position.addScaledVector(p.vel,dt*8);
  p.mesh.scale.multiplyScalar(.97);
  if(p.life<=0){scene.remove(p.mesh);particles.splice(i,1)}
 }
}

function updateCamera(){
 const offset=new THREE.Vector3(0,5.2,-9).applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
 camera.position.copy(player.position).add(offset);
 const look=player.position.clone().add(new THREE.Vector3(0,1.7,0));
 look.add(new THREE.Vector3(Math.sin(yaw)*5, pitch*8,Math.cos(yaw)*5));
 camera.lookAt(look);
}

function animate(){
 requestAnimationFrame(animate);
 const dt=Math.min(clock.getDelta(),.04),time=clock.elapsedTime;
 if(running&&!gameOver){
  movePlayer(dt);
  if(shooting&&time-lastShot>.24){muzzle();lastShot=time}
  updateEnemies(dt,time);
  updateProjectiles(dt);
  updateParticles(dt);
 }
 updateCamera();
 renderer.render(scene,camera);
}
animate();

addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
