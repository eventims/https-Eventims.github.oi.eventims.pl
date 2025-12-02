# https-Eventims.github.oi.eventims.pl
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Event¡m Tickets</title>
<script src="https://cdn.jsdelivr.net/npm/ethers@5.7.2/dist/ethers.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/qrcode/build/qrcode.min.js"></script>
<style>
body{font-family:Arial,sans-serif;background:#f5f6f8;margin:0;padding:0;}
header{background:#e60012;color:white;padding:16px;text-align:center;font-size:28px;font-weight:bold;}
.container{max-width:1200px;margin:20px auto;padding:0 16px;}
.filters{display:flex;gap:12px;flex-wrap:wrap;margin-bottom:20px;}
input,select{padding:8px;border-radius:8px;border:1px solid #ccc;}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:16px;}
.event{background:white;border-radius:12px;padding:12px;box-shadow:0 6px 18px rgba(0,0,0,0.1);}
.event img, .event video{width:100%;border-radius:8px;}
.event-title{font-weight:700;margin:8px 0;}
.event-price{color:#e60012;font-weight:700;margin-bottom:8px;}
.btn{padding:8px 12px;border:none;border-radius:8px;cursor:pointer;}
.btn-primary{background:#e60012;color:white;}
.buy-link{display:inline-block;margin-top:8px;color:#e60012;font-weight:bold;text-decoration:none;}
.buy-link:hover{text-decoration:underline;}
.modal, .backdrop{display:none;position:fixed;inset:0;justify-content:center;align-items:center;}
.backdrop{background:rgba(0,0,0,0.5);z-index:1000;}
.modal{background:white;padding:20px;border-radius:12px;max-width:600px;width:90%;z-index:1001;}
.admin{position:fixed;top:80px;right:16px;background:white;padding:16px;border-radius:12px;box-shadow:0 8px 30px rgba(0,0,0,0.12);width:360px;display:none;z-index:1100;}
label{display:block;margin-top:8px;}
.qr-code{margin-top:8px;}
</style>
</head>
<body>

<header>Event¡m</header>

<div class="container">
  <div class="filters">
    <input type="text" id="searchInput" placeholder="Search events...">
    <select id="priceFilter">
      <option value="all">All prices</option>
      <option value="low">180$ - 500$</option>
      <option value="mid">500$ - 1000$</option>
      <option value="high">1000$ - 1500$</option>
    </select>
    <button id="adminBtn" class="btn">Admin Panel</button>
  </div>
  <div class="grid" id="eventsGrid"></div>
</div>

<!-- Event Modal -->
<div class="backdrop" id="modalBackdrop">
  <div class="modal">
    <h2 id="modalTitle">Event</h2>
    <p id="modalDesc"></p>
    <p>Date: <span id="modalDate"></span></p>
    <p>Time: <span id="modalTime"></span></p>
    <p>Venue: <span id="modalVenue"></span></p>
    <p>Price: <span id="modalPrice"></span></p>
    <p>Select Seat / Quantity:
      <select id="modalQty"></select>
    </p>
    <p>
      <button id="connectWalletBtn" class="btn btn-primary">Connect Wallet (Ethereum/BSC)</button>
    </p>
    <p>Select Crypto Payment:</p>
    <select id="cryptoSelect">
      <option value="btc">Bitcoin (BTC)</option>
      <option value="usdt">USDT (TRC‑20)</option>
      <option value="bnb">BNB (BSC)</option>
    </select>
    <p>Wallet Address: <input type="text" id="modalWallet" readonly></p>
    <div id="qrCode" class="qr-code"></div>
    <button id="buyBtn" class="btn btn-primary">Buy with Selected Crypto</button>
    <button id="closeModal" class="btn">Close</button>
  </div>
</div>

<!-- Admin Panel -->
<div class="admin" id="adminPanel">
  <h3>Event¡m Admin Panel</h3>
  <p>Add / Edit Event</p>
  <label>Title</label><input id="adminTitle">
  <label>Description</label><input id="adminDesc">
  <label>Price ($180-$1500)</label><input type="number" id="adminPrice" min="180" max="1500">
  <label>Image/Video URL</label><input id="adminImg">
  <label>Date (YYYY-MM-DD)</label><input type="date" id="adminDate">
  <label>Time (HH:MM)</label><input type="time" id="adminTime">
  <label>Venue</label><input id="adminVenue">
  <label>Ticket Link (URL)</label><input id="adminLink">
  <button id="saveEvent" class="btn btn-primary">Save Event</button>
  <button id="closeAdmin" class="btn">Close</button>
</div>

<script>
// Wallet addresses
const wallets = {
  btc: "3BRZzqBgiW12fMQn3ELqc9UKZSNqgDCpKw",
  usdt: "TK1a6JSeppRZArSmEW1MLr9sr5LGasdQuk",
  bnb: "0xA7c24079ccD5dD20ECE0F8fc5FbB1690bDFA6Ad9"
};

// Web3 smart contract (Ethereum/BSC)
const contractAddress = "0xYourDeployedContractAddress"; // replace with deployed contract
const contractABI = [ /* your contract ABI */ ];

let events = [
  { id:1, title:"SANAH AT THE STADIUMS", desc:"Multiple concerts", img:"https://www.wroclaw.pl/cdn-cgi/image/,f=avif/https://go.wroclaw.pl/api/download/img-3c5bd1aaac110002337e46cc96be7661/wroclaw-sns2026-1080x1080px.png", date:"2026-05-01 — 2026-05-14", time:"-", venue:"Various Stadiums", price:350, link:"https://rozrywka.spidersweb.pl/sanah-na-stadionach-2026-koncerty-gdzie-kiedy-bilety" },
  { id:2, title:"Enrique Iglesias", desc:"Live Concert", video:"https://cdn-p.smehost.net/sites/bbc9ec150d0d4864b8eb7fe0412c1d0e/wp-content/uploads/2025/11/POLAND_EI_HEADER.mp4", date:"2026-05-09", time:"20:00", venue:"Gdańsk", tickets:[{seat:"150$", price:150},{seat:"250$", price:250},{seat:"350$", price:350},{seat:"450$", price:450},{seat:"700$", price:700},{seat:"1000$", price:1000},{seat:"1250$", price:1250},{seat:"1500$", price:1500}] },
  { id:3, title:"BRYAN ADAMS", desc:"Rock Concert", date:"2025-12-15", time:"20:00", venue:"Main Arena", price:450 },
  { id:4, title:"MAT", desc:"Live Concert", date:"2026-05-15", time:"20:00", venue:"PGE National", price:400 }
];

function renderEvents(){
  const grid = document.getElementById("eventsGrid");
  const search = document.getElementById("searchInput").value.toLowerCase();
  const priceFilter = document.getElementById("priceFilter").value;
  grid.innerHTML = "";
  events.filter(e=>{
    let matchSearch = e.title.toLowerCase().includes(search) || e.desc.toLowerCase().includes(search);
    let matchPrice = true;
    if(priceFilter==='low') matchPrice = e.price>=180 && e.price<=500;
    if(priceFilter==='mid') matchPrice = e.price>500 && e.price<=1000;
    if(priceFilter==='high') matchPrice = e.price>1000 && e.price<=1500;
    return matchSearch && matchPrice;
  }).forEach(e=>{
    const div = document.createElement("div"); div.className="event";
    div.innerHTML = `
      ${ e.video ? `<video width="100%" height="140" autoplay muted loop><source src="${e.video}" type="video/mp4"></video>` : `<img src="${e.img||'https://via.placeholder.com/260x140'}">`}
      <div class="event-title">${e.title}</div>
      <div>${e.desc}</div>
      <div>Date: ${e.date} | Time: ${e.time}</div>
      <div>Venue: ${e.venue}</div>
      <div class="event-price">${e.price ? e.price+"$" : ""}</div>
      <button class="btn btn-primary" onclick="openModal(${e.id})">Buy</button>
      ${e.link ? `<a href="${e.link}" target="_blank" class="buy-link">→ Buy Tickets</a>` : ''}
    `;
    grid.appendChild(div);
  });
}

// Connect MetaMask wallet
async function connectWallet() {
  if (window.ethereum) {
    try {
      const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
      const walletAddress = accounts[0];
      document.getElementById("modalWallet").value = walletAddress;
      alert("Wallet connected: " + walletAddress);
    } catch(err){ console.error(err); }
  } else { alert("MetaMask not installed!"); }
}

// Web3 buy (ETH/BNB)
async function buyTicketWeb3(ticketId, price){
  if (!window.ethereum) return alert("Install MetaMask!");
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  const signer = provider.getSigner();
  const contract = new ethers.Contract(contractAddress, contractABI, signer);
  try{
    const tx = await contract.buyTicket(ticketId, { value: ethers.utils.parseEther((price/1000).toString()) });
    await tx.wait();
    alert("Ticket purchased!");
  }catch(err){ console.error(err); alert("Purchase failed."); }
}

function openModal(id){
  const ev = events.find(e=>e.id===id);
  document.getElementById("modalTitle").innerText = ev.title;
  document.getElementById("modalDesc").innerText = ev.desc;
  document.getElementById("modalDate").innerText = ev.date;
  document.getElementById("modalTime").innerText = ev.time;
  document.getElementById("modalVenue").innerText = ev.venue;
  const qtySelect = document.getElementById("modalQty");
  const modalPrice = document.getElementById("modalPrice");

  if(ev.tickets){
    qtySelect.innerHTML = ev.tickets.map(t=>`<option value="${t.price}">${t.seat}</option>`).join('');
    modalPrice.innerText = ev.tickets[0].price+"$";
  } else { qtySelect.innerHTML = `<option>1</option>`; modalPrice.innerText = ev.price+"$"; }

  document.getElementById("modalBackdrop").style.display = "flex";

  qtySelect.onchange = function(){ modalPrice.innerText = this.value+"$"; }

  const cryptoSelect = document.getElementById("cryptoSelect");
  function updateWalletAndQR(){
    const crypto = cryptoSelect.value;
    document.getElementById("modalWallet").value = wallets[crypto];
    QRCode.toCanvas(document.getElementById("qrCode"), wallets[crypto], function (error) { if(error) console.error(error); });
  }
  cryptoSelect.onchange = updateWalletAndQR;
  updateWalletAndQR();

  document.getElementById("connectWalletBtn").onclick = connectWallet;
  document.getElementById("buyBtn").onclick = ()=>{
    const price = qtySelect.value;
    const crypto = cryptoSelect.value;
    if(crypto==='bnb'){ buyTicketWeb3(id, price); } else {
      alert(`Send ${price}$ to ${wallets[crypto]} via ${crypto.toUpperCase()}`);
    }
  }
}

// Modal close
document.getElementById("closeModal").onclick = ()=>{document.getElementById("modalBackdrop").style.display="none";}
document.getElementById("searchInput").oninput = renderEvents;
document.getElementById("priceFilter").onchange = renderEvents;

/* Admin */
document.getElementById("adminBtn").onclick = ()=>{document.getElementById("adminPanel").style.display="block";}
document.getElementById("closeAdmin").onclick = ()=>{document.getElementById("adminPanel").style.display="none";}
document.getElementById("saveEvent").onclick = ()=>{
  const id = events.length+1;
  events.push({
    id:id,
    title:document.getElementById("adminTitle").value,
    desc:document.getElementById("adminDesc").value,
    price:Number(document.getElementById("adminPrice").value),
    img:document.getElementById("adminImg").value,
    date:document.getElementById("adminDate").value,
    time:document.getElementById("adminTime").value,
    venue:document.getElementById("adminVenue").value,
    link:document.getElementById("adminLink").value
  });
  renderEvents();
  document.getElementById("adminPanel").style.display="none";
}

renderEvents();
</script>

</body>
</html>
