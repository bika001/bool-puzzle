<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animal Book-Puzzle | Fix Button</title>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --accent: #e67e22;
            --highlight: #f1c40f;
            --dark: #2c3e50;
            --size: 150px;
        }

        body {
            font-family: 'Montserrat', sans-serif;
            margin: 0;
            background-color: #f0f3f5;
            overflow: hidden;
            height: 100vh;
        }

        .screen { display: none; width: 100%; height: 100vh; flex-direction: column; align-items: center; justify-content: center; position: absolute; top: 0; left: 0; }
        .active { display: flex; }

        /* 1 СТРАНИЦА (HERO) */
        #hero { background: white; z-index: 200; text-align: center; }
        #hero h1 { font-size: clamp(3rem, 10vw, 5rem); color: var(--accent); margin: 0; font-weight: 900; }
        #hero p { max-width: 600px; margin: 25px; font-size: 1.2rem; color: #7f8c8d; }

        /* 2 СТРАНИЦА (МЕНЮ) */
        #menu { background: #f8f9fa; z-index: 100; }
        .grid-menu { display: grid; grid-template-columns: repeat(2, 220px); gap: 20px; margin-top: 30px; }
        .card { background: white; border-radius: 15px; overflow: hidden; cursor: pointer; transition: 0.3s; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .card:hover { transform: translateY(-5px); box-shadow: 0 10px 25px rgba(0,0,0,0.2); }
        .card img { width: 100%; height: 140px; object-fit: cover; pointer-events: none; }
        .card h3 { margin: 10px; text-align: center; text-transform: uppercase; font-size: 1rem; }

        /* GAME UI */
        #game-ui { background: #dce1e4; perspective: 2000px; }
        .instruction { position: absolute; top: 40px; font-size: 2rem; font-weight: 900; text-transform: uppercase; color: var(--dark); text-align: center; width: 100%; }

        /* The Board */
        #puzzle-container { width: 470px; height: 470px; position: relative; transform-style: preserve-3d; transition: transform 1s cubic-bezier(0.4, 0, 0.2, 1); }
        
        #puzzle-front { 
            position: absolute; width: 100%; height: 100%; background: white; padding: 10px; border-radius: 20px; 
            display: grid; grid-template-columns: repeat(3, 1fr); grid-template-rows: repeat(3, 1fr); gap: 2px;
            backface-visibility: hidden; box-shadow: 0 20px 40px rgba(0,0,0,0.1);
        }
        
        .slot { border: 2px dashed #cfd8dc; border-radius: 5px; background: #fafafa; }
        .slot.glow { background: rgba(241, 196, 15, 0.4) !important; border-color: var(--highlight) !important; box-shadow: 0 0 15px var(--highlight); }

        #puzzle-back { 
            position: absolute; width: 100%; height: 100%; background: linear-gradient(135deg, #e67e22, #d35400); 
            color: white; border-radius: 20px; transform: rotateY(180deg); backface-visibility: hidden; 
            display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 40px; text-align: center; box-sizing: border-box;
            pointer-events: auto; /* Важно для кликов */
        }

        /* Puzzle Piece */
        .piece { 
            position: absolute; width: var(--size); height: var(--size); 
            background-size: 450px 450px; cursor: grab; border-radius: 8px; 
            box-shadow: 0 8px 15px rgba(0,0,0,0.2); z-index: 500;
        }

        .is-locked { cursor: default; box-shadow: none; z-index: 10; }

        /* КНОПКИ */
        .btn { 
            padding: 15px 45px; font-size: 1.1rem; font-weight: 800; border: none; border-radius: 50px; 
            cursor: pointer; background: var(--dark); color: white; transition: 0.3s; 
            position: relative; z-index: 999; pointer-events: auto;
        }
        .btn:hover { background: var(--accent); transform: scale(1.05); }

        .btn-play-again {
            background: white;
            color: var(--dark);
            margin-top: 30px;
        }
    </style>
</head>
<body>

    <section id="hero" class="screen active">
        <h1>Book-puzzle</h1>
        <p>Interactive animal journey. Solve the puzzle to unlock nature's secrets!</p>
        <button class="btn" onclick="showScreen('menu')">START JOURNEY</button>
    </section>

    <section id="menu" class="screen">
        <h2 style="font-size: 2.5rem; text-transform: uppercase;">Choose Animal</h2>
        <div class="grid-menu" id="menu-grid"></div>
        <button class="btn" style="margin-top: 30px; background: #95a5a6;" onclick="showScreen('hero')">BACK</button>
    </section>

    <section id="game-ui" class="screen">
        <div class="instruction" id="title">Assemble!</div>
        <div id="puzzle-container">
            <div id="puzzle-front">
                <div class="slot" data-id="0"></div><div class="slot" data-id="1"></div><div class="slot" data-id="2"></div>
                <div class="slot" data-id="3"></div><div class="slot" data-id="4"></div><div class="slot" data-id="5"></div>
                <div class="slot" data-id="6"></div><div class="slot" data-id="7"></div><div class="slot" data-id="8"></div>
            </div>
            <div id="puzzle-back">
                <h2 id="f-title">Facts</h2>
                <p id="f-text" style="margin-bottom: 20px;"></p>
                <button class="btn btn-play-again" id="play-again-btn">PLAY AGAIN</button>
            </div>
        </div>
    </section>

    <script>
        const animals = [
            { name: "Reindeer", img: "https://images.unsplash.com/photo-1484406566174-9da000fda645?w=600", fact: "Reindeer are amazing: their antlers are the fastest-growing bone; they are excellent swimmers; their milk is five times fatter than cow's milk." },
            { name: "Horse", img: "https://images.unsplash.com/photo-1553284965-83fd3e82fa5a?w=600", fact: "Horses have the largest eyes of any land mammal. They can sleep both lying down and standing up. Their brain is smaller than their massive teeth!" },
            { name: "Cat", img: "https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=600", fact: "Cats spend 70% of their lives sleeping. They have 32 muscles in each ear, and their nose print is unique as a human fingerprint!" },
            { name: "Dog", img: "https://images.unsplash.com/photo-1517849845537-4d257902454a?w=600", fact: "A dog's sense of smell is 40,000 times stronger than a human's. They can breathe in and out at the same time." }
        ];

        let solved = 0;
        let piecesRef = [];

        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
        }

        // Кнопка Play Again теперь работает через прямой Event Listener для надежности
        document.getElementById('play-again-btn').addEventListener('click', function(e) {
            e.preventDefault();
            e.stopPropagation();
            backToMenu();
        });

        function backToMenu() {
            // 1. Сброс поворота доски
            document.getElementById('puzzle-container').style.transform = "rotateY(0deg)";
            
            // 2. Удаление всех кусочков пазла с экрана
            piecesRef.forEach(p => {
                if (p && p.parentNode) {
                    p.parentNode.removeChild(p);
                }
            });
            
            // 3. Обнуление данных
            piecesRef = [];
            solved = 0;
            
            // 4. Переход на 2 страницу
            showScreen('menu');
        }

        const grid = document.getElementById('menu-grid');
        animals.forEach(a => {
            const div = document.createElement('div');
            div.className = 'card';
            div.innerHTML = `<img src="${a.img}"><h3>${a.name}</h3>`;
            div.onclick = () => startLevel(a);
            grid.appendChild(div);
        });

        function startLevel(animal) {
            showScreen('game-ui');
            document.getElementById('title').innerText = "Assemble the " + animal.name;
            document.getElementById('f-title').innerText = animal.name + " Facts";
            document.getElementById('f-text').innerText = animal.fact;
            
            piecesRef = [];
            solved = 0;
            for(let i=0; i<9; i++) {
                const p = document.createElement('div');
                p.className = 'piece';
                p.style.backgroundImage = `url('${animal.img}')`;
                const r = Math.floor(i/3), c = i%3;
                p.style.backgroundPosition = `-${c*150}px -${r*150}px`;
                p.style.left = (Math.random() < 0.5 ? 30 : window.innerWidth - 180) + 'px';
                p.style.top = (Math.random() * (window.innerHeight - 250) + 100) + 'px';
                
                initDrag(p, i);
                document.body.appendChild(p);
                piecesRef.push(p);
            }
        }

        function initDrag(el, id) {
            let sx, sy;
            el.onmousedown = (e) => {
                if(el.classList.contains('is-locked')) return;
                sx = e.clientX - el.offsetLeft;
                sy = e.clientY - el.offsetTop;
                el.style.zIndex = 1000;

                document.onmousemove = (e) => {
                    el.style.left = (e.clientX - sx) + 'px';
                    el.style.top = (e.clientY - sy) + 'px';
                    const slot = findSlot(el);
                    document.querySelectorAll('.slot').forEach(s => s.classList.remove('glow'));
                    if(slot && slot.dataset.id == id) slot.classList.add('glow');
                }

                document.onmouseup = () => {
                    document.onmousemove = null;
                    const slot = findSlot(el);
                    if(slot && slot.dataset.id == id) {
                        const sr = slot.getBoundingClientRect();
                        el.style.left = sr.left + 'px';
                        el.style.top = sr.top + 'px';
                        el.classList.add('is-locked');
                        solved++;
                        if(solved === 9) onWin();
                    }
                    el.style.zIndex = 100;
                }
            }
        }

        function findSlot(el) {
            const r = el.getBoundingClientRect();
            const cx = r.left + 75, cy = r.top + 75;
            return [...document.querySelectorAll('.slot')].find(s => {
                const sr = s.getBoundingClientRect();
                return cx > sr.left && cx < sr.right && cy > sr.top && cy < sr.bottom;
            });
        }

        function onWin() {
            setTimeout(() => {
                const container = document.getElementById('puzzle-container');
                const front = document.getElementById('puzzle-front');
                const boardRect = front.getBoundingClientRect();

                piecesRef.forEach(p => {
                    const r = p.getBoundingClientRect();
                    p.style.left = (r.left - boardRect.left) + 'px';
                    p.style.top = (r.top - boardRect.top) + 'px';
                    front.appendChild(p); 
                });

                container.style.transform = "rotateY(180deg)";
            }, 600);
        }

        document.addEventListener('dragstart', e => e.preventDefault());
    </script>
</body>
</html>
