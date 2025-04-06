<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Order</title>
  <link rel="stylesheet" href="style.css" />
  <link rel="icon" type="image/jpeg" href="imgs/pfppp.jpg" />
</head>

<body>
  <div class="center-wrapper">
    <h1>Order</h1>
    <div class="button-group">
      <button onclick="pullGacha(1)">Single Pull</button>
      <input type="number" id="customPullCount" min="1" max="6" placeholder="1-6" />
      <button onclick="pullCustomAmount()" class="glow">Multiple Pulls</button>
      <button onclick="resetGacha()">Reset Gacha</button>
      <button onclick="toggleHistory()">Show History</button>
    </div>
  </div>

  <div id="slot-wrapper"></div>
  <div class="results" id="results"></div>
  <div class="results" id="history" style="display: none;"></div>
  <div id="burst"></div>

  <audio id="legendarySound" src="sounds/legendary.mp3" preload="auto"></audio>

  <script>
    const items = [
      { name: 'Bookworm', rarity: 'Common', image: 'imgs/Common_Bookworm.png' },
      { name: 'Brute', rarity: 'Common', image: 'imgs/Common_Brute.jpg' },
      { name: 'Hidden', rarity: 'Common', image: 'imgs/Common_hiden.jpg' },
      { name: 'Speedster', rarity: 'Common', image: 'imgs/Common_Speedster.jpg' },
      { name: 'Tanker', rarity: 'Common', image: 'imgs/Common_Tanker.jpg' },

      { name: 'Courage', rarity: 'Epic', image: 'imgs/Epic_Courage.jpg' },
      { name: 'Evil', rarity: 'Epic', image: 'imgs/Epic_Evil.jpg' },
      { name: 'Flicker', rarity: 'Epic', image: 'imgs/Epic_Flicker.jpg' },
      { name: 'Justice', rarity: 'Epic', image: 'imgs/Epic_Justice.jpg' },
      { name: 'Manipulator', rarity: 'Epic', image: 'imgs/Epic_Manipulator.jpg' },

      { name: 'Chemical', rarity: 'Rare', image: 'imgs/Rare_Chemical.png' },
      { name: 'Excellent Aim', rarity: 'Rare', image: 'imgs/Rare_ExcellentAim.jpg' },
      { name: 'Martial Prowess', rarity: 'Rare', image: 'imgs/Rare_MartialProwess.jpg' },
      { name: 'Rusty', rarity: 'Rare', image: 'imgs/Rare_Rusty.png' },
      { name: 'Zero Confidence', rarity: 'Rare', image: 'imgs/Rare_ZeroConfidence.png' },

     

      { name: 'Martial Master', rarity: 'Unique', image: 'imgs/Unique_MartialMaster.jpg' },
      { name: 'Ghost Walk', rarity: 'Unique', image: 'imgs/Unique_GhostWalk.png' },
      { name: 'Imaginary', rarity: 'Unique', image: 'imgs/Unique_Imaginary.jpg' }
    ];

    const rarityChances = {
      Common: 54,
      Rare: 22,
      Epic: 5,
      Unique: 1,
      Legendary: 0.2
    };

    const pool = [];
    Object.entries(rarityChances).forEach(([rarity, chance]) => {
      for (let i = 0; i < chance; i++) {
        pool.push(rarity);
      }
    });

    let history = [];

    function getRandomItem() {
      const rarity = pool[Math.floor(Math.random() * pool.length)];
      const options = items.filter(item => item.rarity === rarity);
      return options[Math.floor(Math.random() * options.length)];
    }

    function createCard(item) {
      return `
        <div class="card">
          <img src="${item.image}" alt="${item.name}" id="${item.rarity}" />
          <div class="rarity ${item.rarity}">${item.name} (${item.rarity})</div>
        </div>
      `;
    }

    function triggerBurstEffect() {
      const burst = document.getElementById('burst');
      burst.classList.add('burst-active');
      setTimeout(() => {
        burst.classList.remove('burst-active');
      }, 800);
    }

    function showSlotMachine(pulled) {
      const wrapper = document.getElementById("slot-wrapper");
      wrapper.innerHTML = '';
      const slotContainer = document.createElement("div");
      slotContainer.className = "slot-container";

      for (let i = 0; i < pulled.length; i++) {
        const reel = document.createElement("div");
        reel.className = "reel";

        const reelTrack = document.createElement("div");
        reelTrack.className = "reel-track";

        for (let j = 0; j < 5; j++) {
          const fake = document.createElement("div");
          fake.className = "reel-item";
          fake.textContent = "Rolling...";
          reelTrack.appendChild(fake);
        }

        const final = document.createElement("div");
        final.className = "reel-item";
        final.innerHTML = `<img src="${pulled[i].image}" style="height: 100px;"><br>${pulled[i].name}`;
        reelTrack.appendChild(final);

        reel.appendChild(reelTrack);
        slotContainer.appendChild(reel);
      }

      wrapper.appendChild(slotContainer);

      setTimeout(() => {
        wrapper.innerHTML = '';
        const results = document.getElementById("results");
        results.innerHTML = pulled.map(createCard).join('');
      }, 2000);
    }

    function pullGacha(times) {
      if (times > 6) times = 6;
      const results = document.getElementById('results');
      const wrapper = document.getElementById('slot-wrapper');
      results.innerHTML = '';
      wrapper.innerHTML = '';
      const pulled = [];

      for (let i = 0; i < times; i++) {
        const item = getRandomItem();
        pulled.push(item);
      }

      const rarityOrder = ['Legendary', 'Unique', 'Epic', 'Rare', 'Common'];
      pulled.sort((a, b) => rarityOrder.indexOf(a.rarity) - rarityOrder.indexOf(b.rarity));

      history.push(...pulled);
      showSlotMachine(pulled);

      if (pulled.some(item => item.rarity === 'Legendary')) {
        triggerBurstEffect();
        document.getElementById("legendarySound").play();
      }
    }

    function pullCustomAmount() {
      const input = document.getElementById('customPullCount');
      let value = input.value.trim();
      let count = value === '' ? 6 : parseInt(value);
      if (isNaN(count) || count < 1) count = 1;
      if (count > 6) count = 6;
      pullGacha(count);
    }

    function resetGacha() {
      document.getElementById('results').innerHTML = '';
      document.getElementById('slot-wrapper').innerHTML = '';
    }

    function toggleHistory() {
      const histDiv = document.getElementById('history');
      if (histDiv.style.display === 'none') {
        histDiv.style.display = 'flex';
        histDiv.innerHTML = history.map(createCard).join('');
      } else {
        histDiv.style.display = 'none';
      }
    }

    document.getElementById('customPullCount').addEventListener('keydown', function (event) {
      if (event.key === 'Enter') {
        pullCustomAmount();
      }
    });
  </script>
</body>
</html>
