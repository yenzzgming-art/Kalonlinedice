<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>KAL DICE</title>

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

body{
background:#0d6b32;
color:white;
text-align:center;
padding:20px;
}

h1{
font-size:50px;
color:#FFD84D;
text-shadow:
3px 3px 0 #000,
0 0 15px gold,
0 0 30px orange;
margin:20px 0;
}

h3{
margin-bottom:20px;
font-size:20px;
}

.colors{
display:flex;
justify-content:center;
flex-wrap:wrap;
gap:8px;
margin-bottom:25px;
}

.colors span{
background:white;
padding:8px 15px;
border-radius:30px;
font-weight:bold;
}

.red{color:red;}
.orange{color:orange;}
.yellow{color:#FFD600;}
.green{color:green;}
.blue{color:#0080ff;}
.purple{color:purple;}

.dice-box{
display:flex;
justify-content:center;
gap:15px;
flex-wrap:wrap;
margin:30px 0;
}

.dice{
width:90px;
height:90px;
border-radius:18px;
border:8px solid white;
background:white;
display:flex;
justify-content:center;
align-items:center;
box-shadow:0 4px 15px rgba(0,0,0,0.3);
transition:.3s ease;
}

/* Titik dadu dibuat transparan/gelap tipis agar tidak merusak warna latar */
.dot{
width:24px;
height:24px;
background:rgba(0, 0, 0, 0.15);
border-radius:50%;
transition: .3s;
}

.rolling{
animation:spin 0.5s linear infinite;
}

@keyframes spin{
100%{
transform:rotate(360deg);
}
}

button{
padding:15px 45px;
font-size:22px;
font-weight:bold;
border:none;
border-radius:18px;
cursor:pointer;
background:linear-gradient(45deg,#4facfe,#0066ff);
color:white;
box-shadow:0 5px 15px rgba(0,102,255,0.4);
transition:.2s;
}

button:hover{
transform:scale(1.05);
}

button:active{
transform:scale(0.95);
}

footer{
margin-top:35px;
font-weight:bold;
font-size:18px;
}
</style>
</head>

<body>

<h1>KAL DICE</h1>

<h3>6 COLOR DICE • SECURE RANDOM ROLLS</h3>

<div class="colors">
<span class="red">RED</span>
<span class="orange">ORANGE</span>
<span class="yellow">YELLOW</span>
<span class="green">GREEN</span>
<span class="blue">BLUE</span>
<span class="purple">PURPLE</span>
</div>

<div class="dice-box">
<div class="dice"><div class="dot"></div></div>
<div class="dice"><div class="dot"></div></div>
<div class="dice"><div class="dot"></div></div>
<div class="dice"><div class="dot"></div></div>
</div>

<button onclick="roll()">🎲 ROLL</button>

<footer>
© 2026 KAL DICE
</footer>

<script>
const colors=[
"red",
"orange",
"yellow",
"green",
"blue",
"purple"
];

function roll(){
const dice = document.querySelectorAll(".dice");

dice.forEach(d => {
d.classList.add("rolling");
d.style.background = "white";
// Sembunyikan titik saat berputar agar efek putaran lebih bersih
d.querySelector('.dot').style.opacity = "0"; 
});

setTimeout(() => {
dice.forEach(d => {
d.classList.remove("rolling");

let color = colors[Math.floor(Math.random() * colors.length)];
d.style.background = color;

// Tampilkan kembali titik dengan kontras warna yang sesuai
const dot = d.querySelector('.dot');
dot.style.opacity = "1";

// Jika warna dadu kuning atau putih, titik dibuat hitam agar kelihatan
if(color === "yellow" || color === "white") {
dot.style.background = "rgba(0,0,0,0.5)";
} else {
dot.style.background = "rgba(255,255,255,0.7)";
}
});
}, 800); // Disamakan temponya dengan kecepatan animasi berputar
}
</script>

</body>
</html>
