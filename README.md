<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Mouridoul Lahi</title>

<style>
body{
    font-family: Arial, sans-serif;
    background:#071018;
    color:#eaf6ff;
    margin:0;
    padding:20px;
}
.container{max-width:1000px;margin:auto;}
header{
    display:flex;
    justify-content:space-between;
    align-items:center;
}
nav a{
    color:#66ffb3;
    margin-left:14px;
    text-decoration:none;
}
.card{
    background:#091221;
    padding:18px;
    border-radius:12px;
    margin-top:14px;
    box-shadow:0 6px 20px rgba(0,0,0,0.6);
}
input, select{
    width:100%;
    padding:10px;
    margin-top:6px;
    background:#08101a;
    border:1px solid #203040;
    border-radius:8px;
    color:white;
}
button{
    background:linear-gradient(90deg,#00e08a,#00b37a);
    border:none;
    padding:12px;
    margin-top:14px;
    border-radius:8px;
    cursor:pointer;
}
table{
    width:100%;
    border-collapse:collapse;
    margin-top:14px;
}
th, td{
    padding:8px;
    border-bottom:1px solid #162532;
}
.hidden{display:none;}
.label{margin-top:10px;display:block;}
.month-box{
    padding:10px;
    background:#062033;
    margin-top:10px;
    border-radius:8px;
}
</style>
</head>

<body>
<div class="container">

<header>
    <h1>Mouridoul Lahi</h1>
    <nav>
        <a onclick="showPage('home')">Accueil</a>
        <a onclick="showPage('addMember')">Ajouter membre</a>
        <a onclick="showPage('members')">Membres</a>
        <a onclick="showPage('summary')">Résumé</a>
    </nav>
</header>

<!-- 🟦 PAGE : ACCUEIL -->
<div id="home" class="card">
    <h2>Tableau de bord</h2>
    <p id="dashboard"></p>
</div>

<!-- 🟦 PAGE : AJOUTER MEMBRE -->
<div id="addMember" class="card hidden">
    <h2>Ajouter un membre</h2>

    <label class="label">Nom :</label>
    <input id="newName">

    <label class="label">Téléphone :</label>
    <input id="newPhone">

    <button onclick="addMember()">Ajouter</button>
</div>

<!-- 🟦 PAGE : LISTE MEMBRES -->
<div id="members" class="card hidden">
    <h2>Liste des membres</h2>
    <table id="membersTable"></table>
</div>

<!-- 🟦 PAGE : DETAILS MEMBRE -->
<div id="memberDetail" class="card hidden">
    <h2 id="memberName"></h2>
    <p id="memberPhone"></p>

    <h3>Ajouter un paiement</h3>

    <label class="label">Mois :</label>
    <select id="payMonth"></select>

    <label class="label">Jour :</label>
    <input type="number" id="payDay" value="1">

    <label class="label">Année :</label>
    <input type="number" id="payYear" value="2025">

    <label class="label">Montant :</label>
    <input type="number" id="payAmount">

    <button onclick="addPayment()">Enregistrer</button>

    <h3>Paiements</h3>
    <table id="paymentsTable"></table>

    <button onclick="showPage('members')">Retour</button>
</div>

<!-- 🟦 PAGE : RÉSUMÉ MENSUEL -->
<div id="summary" class="card hidden">
    <h2>Résumé mensuel</h2>
    <div id="summaryBox"></div>
</div>

</div>

<script>
const MONTHS = ["Janvier","Février","Mars","Avril","Mai","Juin","Juillet","Août","Septembre"];

let members = JSON.parse(localStorage.getItem("members") || "[]");
let payments = JSON.parse(localStorage.getItem("payments") || "[]");
let currentMemberId = null;

function save(){
    localStorage.setItem("members", JSON.stringify(members));
    localStorage.setItem("payments", JSON.stringify(payments));
}

function showPage(page){
    document.querySelectorAll(".card").forEach(c=>c.classList.add("hidden"));
    document.getElementById(page).classList.remove("hidden");
    reloadUI();
}

function reloadUI(){
    // Dashboard
    document.getElementById("dashboard").innerHTML =
        `Membres : <b>${members.length}</b> — Paiements : <b>${payments.length}</b>`;

    // Members table
    let m = "<tr><th>ID</th><th>Nom</th><th>Téléphone</th><th></th></tr>";
    members.forEach((x,i)=>{
        m += `<tr>
                <td>${i+1}</td>
                <td>${x.name}</td>
                <td>${x.phone}</td>
                <td><button onclick="openMember(${i})">Ouvrir</button></td>
              </tr>`;
    });
    document.getElementById("membersTable").innerHTML = m;

    // Summary box
    let s = "";
    MONTHS.forEach(m=>{
        let list = payments.filter(p=>p.month===m);
        let total = list.reduce((a,b)=>a+b.amount,0);
        s += `<div class='month-box'><b>${m}</b> : ${list.length} paiements — total ${total} FCFA</div>`;
    });
    document.getElementById("summaryBox").innerHTML = s;

    // Fill month dropdown
    let opt="";
    MONTHS.forEach(m=>opt+=`<option>${m}</option>`);
    document.getElementById("payMonth").innerHTML=opt;
}

function addMember(){
    let name = document.getElementById("newName").value.trim();
    let phone = document.getElementById("newPhone").value.trim();
    if(!name || !phone){ alert("Remplis tous les champs"); return; }

    members.push({name,phone});
    save();
    showPage('members');
}

function openMember(id){
    currentMemberId = id;
    let m = members[id];

    document.getElementById("memberName").innerText = m.name;
    document.getElementById("memberPhone").innerText = "Téléphone : " + m.phone;

    // Payments
    let list = payments.filter(p=>p.memberId===id);
    let t = "<tr><th>Mois</th><th>Jour</th><th>Année</th><th>Montant</th></tr>";
    list.forEach(p=>{
        t+=`<tr>
                <td>${p.month}</td>
                <td>${p.day}</td>
                <td>${p.year}</td>
                <td>${p.amount}</td>
            </tr>`;
    });
    document.getElementById("paymentsTable").innerHTML = t;

    showPage("memberDetail");
}

function addPayment(){
    let p = {
        memberId: currentMemberId,
        month: document.getElementById("payMonth").value,
        day: parseInt(document.getElementById("payDay").value),
        year: parseInt(document.getElementById("payYear").value),
        amount: parseFloat(document.getElementById("payAmount").value)
    };

    payments.push(p);
    save();
    openMember(currentMemberId);
}

reloadUI();
</script>

</body>
</html># Mouride_pro
