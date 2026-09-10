<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Declaraciones ❤️</title>

<!-- Firebase -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/12.1.0/firebase-app.js";
import {
  getAuth,
  signInWithEmailAndPassword,
  signOut,
  onAuthStateChanged
} from "https://www.gstatic.com/firebasejs/12.1.0/firebase-auth.js";

import {
  getFirestore,
  collection,
  addDoc,
  deleteDoc,
  doc,
  query,
  orderBy,
  onSnapshot,
  serverTimestamp
} from "https://www.gstatic.com/firebasejs/12.1.0/firebase-firestore.js";


/* =================================
   PEGÁ ACÁ TU CONFIGURACIÓN FIREBASE
================================= */

const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROYECTO.firebaseapp.com",
  projectId: "TU_PROJECT_ID",
  storageBucket: "TU_STORAGE_BUCKET",
  messagingSenderId: "TU_SENDER_ID",
  appId: "TU_APP_ID"
};


/* =================================
   INICIAR FIREBASE
================================= */

const app = initializeApp(firebaseConfig);

const auth = getAuth(app);

const db = getFirestore(app);


/* =================================
   MOSTRAR DECLARACIONES
================================= */

const declaracionesRef =
collection(db,"declaraciones");

const q =
query(
  declaracionesRef,
  orderBy("createdAt","desc")
);


onSnapshot(q,(snapshot)=>{

  const cont =
  document.getElementById("declaraciones");

  const vacio =
  document.getElementById("vacio");

  cont.innerHTML="";

  if(snapshot.empty){

    vacio.style.display="block";

    vacio.textContent=
    "Todavía no hay declaraciones 💌";

    return;
  }

  vacio.style.display="none";


  snapshot.forEach((item)=>{

    const x=item.data();

    let hora="";

    if(x.createdAt){

      const fecha=
      x.createdAt.toDate();

      hora=
      fecha.getHours()
      .toString()
      .padStart(2,"0")
      +":"
      +
      fecha.getMinutes()
      .toString()
      .padStart(2,"0");

    }


    cont.innerHTML+=`

    <div class="declaracion">

      <div class="nombre">

        💌 ${escapar(x.nombre)}

        <span class="curso">

          · ${escapar(x.curso || "")}

        </span>

      </div>

      <div class="texto">

        ${escapar(x.texto)}

      </div>

      <div class="info">

        ${hora} ✓✓

      </div>

    </div>

    `;

  });

});


/* =================================
   ESCAPAR TEXTO
================================= */

function escapar(texto){

  const div=
  document.createElement("div");

  div.textContent=
  texto || "";

  return div.innerHTML;

}


/* =================================
   ABRIR PANEL
================================= */

window.abrirAdmin=function(){

  const panel=
  document.getElementById("panel");

  panel.style.display=
  panel.style.display==="block"
  ?"none"
  :"block";

};


/* =================================
   LOGIN
================================= */

window.login=async function(){

  const email=
  document.getElementById("email").value.trim();

  const password=
  document.getElementById("password").value;

  const error=
  document.getElementById("error");

  error.textContent="";

  try{

    await signInWithEmailAndPassword(
      auth,
      email,
      password
    );

  }catch(e){

    error.textContent=
    "❌ Correo o contraseña incorrectos.";

  }

};


/* =================================
   ESTADO DEL ADMINISTRADOR
================================= */

onAuthStateChanged(auth,(user)=>{

  const loginBox=
  document.getElementById("login");

  const admin=
  document.getElementById("administracion");

  if(user){

    loginBox.style.display="none";

    admin.style.display="block";

    mostrarAdmin();

  }else{

    loginBox.style.display="block";

    admin.style.display="none";

  }

});


/* =================================
   PUBLICAR
================================= */

window.publicar=async function(){

  const user=auth.currentUser;

  if(!user){

    alert("Tenés que iniciar sesión.");

    return;

  }


  let nombre=
  document.getElementById("nombre").value.trim();

  const curso=
  document.getElementById("curso").value.trim();

  const texto=
  document.getElementById("texto").value.trim();


  if(!nombre){

    nombre="Anónimo";

  }


  if(!texto){

    alert("Escribí una declaración.");

    return;

  }


  try{

    await addDoc(
      declaracionesRef,
      {
        nombre:nombre,
        curso:curso,
        texto:texto,
        createdAt:serverTimestamp(),
        autor:user.uid
      }
    );


    document.getElementById("nombre").value="";

    document.getElementById("curso").value="";

    document.getElementById("texto").value="";


    alert(
      "❤️ Declaración publicada correctamente."
    );


  }catch(e){

    console.error(e);

    alert(
      "❌ No se pudo publicar."
    );

  }

};


/* =================================
   ADMINISTRAR
================================= */

function mostrarAdmin(){

  const lista=
  document.getElementById("listaAdmin");


  onSnapshot(q,(snapshot)=>{

    lista.innerHTML="";


    snapshot.forEach((item)=>{

      const x=item.data();


      lista.innerHTML+=`

      <div class="admin-item">

        <strong>

          ${escapar(x.nombre)}

        </strong>

        <small>

          · ${escapar(x.curso || "")}

        </small>

        <p>

          ${escapar(x.texto)}

        </p>

        <button
          class="eliminar"
          onclick="eliminar('${item.id}')">

          🗑️ Eliminar

        </button>

      </div>

      `;

    });

  });

}


/* =================================
   ELIMINAR
================================= */

window.eliminar=async function(id){

  if(!auth.currentUser){

    alert("No tenés permiso.");

    return;

  }


  if(
    !confirm(
      "¿Eliminar esta declaración?"
    )
  ){

    return;

  }


  try{

    await deleteDoc(
      doc(db,"declaraciones",id)
    );

  }catch(e){

    console.error(e);

    alert(
      "❌ No se pudo eliminar."
    );

  }

};


/* =================================
   CERRAR SESIÓN
================================= */

window.cerrarSesion=async function(){

  await signOut(auth);

  location.reload();

};

</script>


<style>

*{
box-sizing:border-box;
}

body{
margin:0;
font-family:Arial,sans-serif;
background:#641526;
color:white;
min-height:100vh;
overflow-x:hidden;
}


/* CORAZONES */

.corazon{
position:fixed;
color:#ffb0c0;
opacity:.15;
font-size:30px;
z-index:0;
animation:flotar 8s infinite linear;
pointer-events:none;
}

.c1{left:5%;top:20%}
.c2{left:20%;top:70%}
.c3{left:40%;top:30%}
.c4{left:60%;top:80%}
.c5{left:80%;top:25%}
.c6{left:92%;top:60%}


@keyframes flotar{

0%{
transform:translateY(40px);
opacity:0;
}

30%{
opacity:.2;
}

100%{
transform:translateY(-120px);
opacity:0;
}

}


/* ENCABEZADO */

header{
text-align:center;
padding:45px 15px 25px;
position:relative;
z-index:1;
}

header h1{
font-size:38px;
margin:0 0 8px;
}

header p{
opacity:.85;
}


/* CONTENEDOR */

.contenedor{
width:94%;
max-width:650px;
margin:auto;
position:relative;
z-index:1;
}


/* ADMIN */

.admin-btn{
display:block;
margin:30px auto;
padding:12px 20px;
border:0;
border-radius:10px;
background:#40101b;
color:white;
font-size:15px;
cursor:pointer;
}


/* PANEL */

#panel{
display:none;
background:white;
color:#351018;
border-radius:18px;
padding:22px;
margin:20px auto;
box-shadow:0 5px 20px #0005;
}

#administracion{
display:none;
}


input,
textarea{
width:100%;
padding:12px;
margin:7px 0 14px;
border:1px solid #ccc;
border-radius:9px;
font-size:16px;
font-family:Arial,sans-serif;
}

textarea{
min-height:110px;
resize:vertical;
}


button{
border:0;
border-radius:9px;
padding:11px 16px;
cursor:pointer;
}


.publicar{
width:100%;
background:#741d32;
color:white;
font-size:16px;
}


#error{
color:red;
font-weight:bold;
}


/* ADMIN */

.admin-item{
border:1px solid #ddd;
border-radius:10px;
padding:12px;
margin-top:12px;
}

.admin-item strong{
color:#741d32;
}


.eliminar{
background:#eee;
color:#a00020;
margin-top:8px;
}


/* TITULO */

.tituloDeclaraciones{
text-align:center;
margin-top:40px;
margin-bottom:20px;
}


/* WHATSAPP */

.declaracion{
background:#efe7de;
color:#222;
padding:10px 12px 7px;
margin:13px 0;
border-radius:9px;
box-shadow:0 2px 5px #0005;
position:relative;
max-width:90%;
margin-left:auto;
}


.declaracion:after{
content:"";
position:absolute;
right:-7px;
top:0;
border-width:0 0 12px 12px;
border-style:solid;
border-color:transparent transparent transparent #efe7de;
}


/* NOMBRE */

.nombre{
font-weight:bold;
color:#8b2140;
font-size:15px;
margin-bottom:5px;
}


.curso{
font-weight:normal;
color:#777;
font-size:12px;
display:inline;
margin-left:5px;
}


/* TEXTO */

.texto{
font-size:16px;
line-height:1.4;
word-wrap:break-word;
}


/* HORA */

.info{
text-align:right;
font-size:11px;
color:#777;
margin-top:4px;
}


/* VACÍO */

.vacio{
text-align:center;
padding:40px 10px;
opacity:.8;
}


/* FOOTER */

footer{
text-align:center;
padding:30px;
opacity:.7;
}


/* CELULAR */

@media(max-width:500px){

header h1{
font-size:31px;
}

.declaracion{
max-width:94%;
}

}

</style>
</head>


<body>


<!-- CORAZONES -->

<div class="corazon c1">♥</div>
<div class="corazon c2">♡</div>
<div class="corazon c3">♥</div>
<div class="corazon c4">♡</div>
<div class="corazon c5">♥</div>
<div class="corazon c6">♡</div>


<!-- ENCABEZADO -->

<header>

<h1>💌 Declaraciones</h1>

<p>
Las declaraciones de nuestra comunidad escolar ❤️
</p>

</header>


<div class="contenedor">


<!-- BOTÓN ADMIN -->

<button
class="admin-btn"
onclick="abrirAdmin()">

🔐 Panel de administrador

</button>


<!-- PANEL -->

<div id="panel">


<!-- LOGIN -->

<div id="login">

<h2>🔐 Administrador</h2>

<p>
Ingresá con tu cuenta de administrador.
</p>


<input
type="email"
id="email"
placeholder="Correo">


<input
type="password"
id="password"
placeholder="Contraseña">


<button
class="publicar"
onclick="login()">

Entrar

</button>


<p id="error"></p>

</div>


<!-- ADMINISTRACIÓN -->

<div id="administracion">

<h2>💌 Nueva declaración</h2>


<label>
Nombre
</label>

<input
id="nombre"
placeholder="Anónimo">


<label>
Curso
</label>

<input
id="curso"
placeholder="Ej: 4°B">


<label>
Declaración
</label>

<textarea
id="texto"
placeholder="Escribí la declaración...">
</textarea>


<button
class="publicar"
onclick="publicar()">

❤️ Publicar

</button>


<button
class="publicar"
onclick="cerrarSesion()"
style="margin-top:8px;background:#555">

Cerrar sesión

</button>


<h3>
Administrar declaraciones
</h3>


<div id="listaAdmin"></div>

</div>

</div>


<!-- DECLARACIONES -->

<h2 class="tituloDeclaraciones">

💌 Declaraciones publicadas

</h2>


<div id="declaraciones"></div>


<div
id="vacio"
class="vacio">

Cargando declaraciones...

</div>


</div>


<footer>

Hecho con ❤️ para el colegio

</footer>

</body>
</html>
