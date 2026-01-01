# belajar-3
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Studio Kolase Pro + AI Magic</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/interact.js/1.10.11/interact.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&family=Playfair+Display:wght@700&family=Dancing+Script:wght@700&family=Montserrat:wght@700&family=Pacifico&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; overflow: hidden; }
        
        /* --- CANVAS & LAYOUT --- */
        .collage-container {
            transition: background 0.3s ease, padding 0.3s ease;
            position: relative;
            background-size: 20px 20px;
            box-shadow: 0 10px 40px -10px rgba(0,0,0,0.2);
            overflow: hidden; /* Keep stickers inside */
        }

        .img-item {
            position: relative;
            overflow: hidden;
            background-color: #f3f4f6;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: border-radius 0.3s ease;
            user-select: none;
        }

        .img-item img {
            width: 100%;
            height: 100%;
            object-fit: cover;
            object-position: 50% 50%; /* Center default */
            cursor: grab;
            pointer-events: auto;
        }
        
        .img-item img:active {
            cursor: grabbing;
        }

        .img-item.dragging-over {
            filter: brightness(0.8);
            border: 2px dashed #6366f1;
        }

        /* --- DRAGGABLE ELEMENTS (Text & Stickers) --- */
        .draggable-element {
            position: absolute;
            cursor: move;
            user-select: none;
            z-index: 50;
            padding: 4px;
            border: 2px solid transparent;
            transition: transform 0.1s;
        }
        
        .draggable-element:hover, .draggable-element.is-selected {
            border: 2px dashed #6366f1;
            background-color: rgba(255, 255, 255, 0.2);
        }

        .delete-element-btn {
            position: absolute;
            top: -12px;
            right: -12px;
            width: 20px;
            height: 20px;
            background: red;
            color: white;
            border-radius: 50%;
            font-size: 10px;
            display: none;
            align-items: center;
            justify-content: center;
            cursor: pointer;
        }

        .draggable-element:hover .delete-element-btn {
            display: flex;
        }

        /* --- LAYOUT GRIDS --- */
        .layout-grid-2x2 { display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr; }
        .layout-vertical { display: flex; flex-direction: column; }
        .layout-horizontal { display: flex; flex-direction: row; }
        .layout-focus-left { display: grid; grid-template-columns: 2fr 1fr; grid-template-rows: 1fr 1fr 1fr; }
        .layout-focus-left .img-item:first-child { grid-row: span 3; }
        .layout-focus-top { display: grid; grid-template-columns: 1fr 1fr 1fr; grid-template-rows: 2fr 1fr; }
        .layout-focus-top .img-item:first-child { grid-column: span 3; }
        .layout-mosaic { display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr; }
        .layout-mosaic .img-item:nth-child(1) { grid-row: span 2; }
        .layout-mosaic .img-item:nth-child(4) { grid-column: span 1; }

        /* --- FILTERS --- */
        .filter-grayscale { filter: grayscale(100%); }
        .filter-sepia { filter: sepia(60%) contrast(110%); }
        .filter-vintage { filter: sepia(40%) hue-rotate(-20deg) saturate(140%) contrast(90%); }
        .filter-contrast { filter: contrast(130%) brightness(105%); }
        .filter-brightness { filter: brightness(120%); }
        
        /* --- PATTERNS --- */
        .bg-pattern-dots { background-image: radial-gradient(#94a3b8 2px, transparent 2px); }
        .bg-pattern-grid { background-image: linear-gradient(#94a3b8 1px, transparent 1px), linear-gradient(90deg, #94a3b8 1px, transparent 1px); }
        
        /* Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #c7c7c7; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #a0a0a0; }

        /* AI Loading Animation */
        .ai-loading {
            animation: pulse-purple 2s infinite;
        }
        @keyframes pulse-purple {
            0% { box-shadow: 0 0 0 0 rgba(147, 51, 234, 0.7); }
            70% { box-shadow: 0 0 0 10px rgba(147, 51, 234, 0); }
            100% { box-shadow: 0 0 0 0 rgba(147, 51, 234, 0); }
        }
    </style>
</head>
<body class="bg-gray-100 h-screen flex flex-col md:flex-row">

    <!-- SIDEBAR -->
    <div class="w-full md:w-80 bg-white shadow-2xl z-20 flex flex-col h-full border-r border-gray-200">
        <!-- Header -->
        <div class="p-4 border-b border-gray-100 bg-gradient-to-r from-purple-600 to-indigo-600 text-white">
            <h1 class="text-lg font-bold flex items-center gap-2">
                <i class="fas fa-palette"></i> Studio Kolase
            </h1>
        </div>

        <!-- Scrollable Tools -->
        <div class="flex-1 overflow-y-auto p-4 space-y-6">
            
            <!-- 1. Media -->
            <div class="space-y-3">
                <label class="text-xs font-bold text-gray-500 uppercase tracking-wider flex items-center justify-between">
                    <span>1. Upload Foto</span>
                    <span class="text-[10px] bg-gray-100 px-2 py-0.5 rounded text-gray-500">Geser utk tukar</span>
                </label>
                <div class="relative group">
                    <input type="file" id="file-input" multiple accept="image/*" class="absolute inset-0 w-full h-full opacity-0 cursor-pointer z-10">
                    <div class="border-2 border-dashed border-indigo-200 rounded-xl p-3 text-center bg-indigo-50 group-hover:bg-indigo-100 transition">
                        <i class="fas fa-cloud-upload-alt text-xl text-indigo-500 mb-1"></i>
                        <p class="text-xs text-indigo-700 font-medium">Pilih Foto</p>
                    </div>
                </div>
            </div>

            <!-- 2. Layouts -->
            <div class="space-y-3">
                <label class="text-xs font-bold text-gray-500 uppercase tracking-wider">2. Tata Letak</label>
                <div class="grid grid-cols-4 gap-2">
                    <button onclick="setLayout('grid-2x2')" class="p-2 border rounded hover:border-indigo-500 hover:bg-indigo-50"><i class="fas fa-th-large"></i></button>
                    <button onclick="setLayout('focus-left')" class="p-2 border rounded hover:border-indigo-500 hover:bg-indigo-50"><i class="fas fa-columns"></i></button>
                    <button onclick="setLayout('focus-top')" class="p-2 border rounded hover:border-indigo-500 hover:bg-indigo-50"><i class="fas fa-window-maximize transform rotate-180"></i></button>
                    <button onclick="setLayout('mosaic')" class="p-2 border rounded hover:border-indigo-500 hover:bg-indigo-50"><i class="fas fa-th"></i></button>
                    <button onclick="setLayout('vertical')" class="p-2 border rounded hover:border-indigo-500 hover:bg-indigo-50"><i class="fas fa-grip-lines-vertical"></i></button>
                    <button onclick="setLayout('horizontal')" class="p-2 border rounded hover:border-indigo-500 hover:bg-indigo-50"><i class="fas fa-grip-lines"></i></button>
                </div>
            </div>

            <!-- 3. Styles -->
            <div class="space-y-3">
                <label class="text-xs font-bold text-gray-500 uppercase tracking-wider">3. Gaya & Filter</label>
                
                <div class="grid grid-cols-2 gap-2">
                    <select onchange="setGlobalFilter(this.value)" class="text-xs border p-2 rounded w-full">
                        <option value="">Normal</option>
                        <option value="filter-vintage">Vintage</option>
                        <option value="filter-sepia">Sepia</option>
                        <option value="filter-grayscale">B&W</option>
                        <option value="filter-contrast">Vivid</option>
                    </select>
                    <div class="flex items-center border rounded px-2 bg-white">
                        <input type="color" class="w-6 h-6 border-none bg-transparent cursor-pointer" onchange="setBgColor(this.value)" value="#ffffff">
                        <span class="text-xs ml-2 text-gray-500">Bg</span>
                    </div>
                </div>

                <!-- Sliders -->
                <div class="space-y-3 pt-2 bg-gray-50 p-3 rounded-lg">
                    <div>
                        <div class="flex justify-between text-[10px] text-gray-500"><span>Jarak</span><span id="gap-val">4</span></div>
                        <input type="range" id="gap-input" max="40" value="4" class="w-full h-1 accent-indigo-600">
                    </div>
                    <div>
                        <div class="flex justify-between text-[10px] text-gray-500"><span>Pinggir</span><span id="padding-val">16</span></div>
                        <input type="range" id="padding-input" max="60" value="16" class="w-full h-1 accent-indigo-600">
                    </div>
                    <div>
                        <div class="flex justify-between text-[10px] text-gray-500"><span>Bulat</span><span id="radius-val">0</span></div>
                        <input type="range" id="radius-input" max="50" value="0" class="w-full h-1 accent-indigo-600">
                    </div>
                </div>
            </div>

            <!-- 4. Text & Stickers -->
            <div class="space-y-3 border-t pt-4">
                <label class="text-xs font-bold text-gray-500 uppercase tracking-wider">4. Tambahan</label>
                
                <!-- Text Tool -->
                <div class="flex gap-2">
                    <input type="text" id="new-text-input" placeholder="Teks baru..." class="text-xs border p-2 rounded flex-1">
                    <button onclick="addText()" class="bg-indigo-600 text-white text-xs px-3 rounded hover:bg-indigo-700">
                        <i class="fas fa-plus"></i> Teks
                    </button>
                </div>

                <!-- Stickers -->
                <div class="grid grid-cols-5 gap-2 text-xl text-center cursor-pointer select-none">
                    <div onclick="addSticker('❤️')" class="hover:scale-125 transition">❤️</div>
                    <div onclick="addSticker('✨')" class="hover:scale-125 transition">✨</div>
                    <div onclick="addSticker('🔥')" class="hover:scale-125 transition">🔥</div>
                    <div onclick="addSticker('🌿')" class="hover:scale-125 transition">🌿</div>
                    <div onclick="addSticker('📸')" class="hover:scale-125 transition">📸</div>
                    <div onclick="addSticker('🎂')" class="hover:scale-125 transition">🎂</div>
                    <div onclick="addSticker('🎉')" class="hover:scale-125 transition">🎉</div>
                    <div onclick="addSticker('☀️')" class="hover:scale-125 transition">☀️</div>
                    <div onclick="addSticker('✈️')" class="hover:scale-125 transition">✈️</div>
                    <div onclick="addSticker('☕')" class="hover:scale-125 transition">☕</div>
                </div>
            </div>

             <!-- 5. AI Magic (GEMINI API) -->
             <div class="space-y-3 border-t pt-4 bg-gradient-to-b from-transparent to-purple-50 -mx-4 px-4 pb-2">
                <label class="text-xs font-bold text-purple-600 uppercase tracking-wider flex items-center gap-1">
                    <i class="fas fa-sparkles"></i> 5. AI Magic Assistant
                </label>
                
                <div class="space-y-2">
                    <input type="text" id="ai-theme-input" placeholder="Tema (cth: Pantai, Ultah, Kopi)" class="w-full text-xs border border-purple-200 p-2 rounded focus:ring-2 focus:ring-purple-200 outline-none">
                    
                    <div class="grid grid-cols-2 gap-2">
                        <button onclick="generateAICaption()" id="btn-ai-caption" class="bg-purple-600 text-white text-xs py-2 px-1 rounded hover:bg-purple-700 transition flex items-center justify-center gap-1">
                            <i class="fas fa-quote-left"></i> Buat Caption
                        </button>
                        <button onclick="generateAIStickers()" id="btn-ai-stickers" class="bg-white border border-purple-300 text-purple-700 text-xs py-2 px-1 rounded hover:bg-purple-50 transition flex items-center justify-center gap-1">
                            <i class="far fa-smile"></i> Saran Stiker
                        </button>
                    </div>
                </div>
            </div>

        </div>

        <!-- Download -->
        <div class="p-4 border-t border-gray-100">
            <button onclick="downloadCollage()" class="w-full bg-gray-900 text-white py-3 rounded-xl font-bold hover:bg-gray-800 transition flex items-center justify-center gap-2">
                <i class="fas fa-download"></i> Simpan
            </button>
        </div>
    </div>

    <!-- MAIN CANVAS AREA -->
    <div class="flex-1 bg-gray-200 flex items-center justify-center overflow-auto p-8 relative">
        
        <!-- Canvas Wrapper -->
        <div id="collage-board" class="collage-container bg-white" style="width: 600px; min-height: 600px; padding: 16px;">
            <!-- Grid Container -->
            <div id="collage-grid" class="layout-grid-2x2 w-full h-full" style="gap: 4px; min-height: 568px;">
                <!-- Initial Placeholders -->
                <div class="img-item rounded-sm border-2 border-dashed border-gray-300">
                    <div class="text-gray-300 text-4xl"><i class="fas fa-image"></i></div>
                </div>
                <div class="img-item rounded-sm border-2 border-dashed border-gray-300">
                    <div class="text-gray-300 text-4xl"><i class="fas fa-image"></i></div>
                </div>
                <div class="img-item rounded-sm border-2 border-dashed border-gray-300">
                    <div class="text-gray-300 text-4xl"><i class="fas fa-image"></i></div>
                </div>
                <div class="img-item rounded-sm border-2 border-dashed border-gray-300">
                    <div class="text-gray-300 text-4xl"><i class="fas fa-image"></i></div>
                </div>
            </div>

            <!-- Draggable Layer (Texts & Stickers go here) -->
            <div id="overlay-layer" class="absolute inset-0 pointer-events-none overflow-hidden">
                <!-- Items added dynamically -->
            </div>
        </div>
        
        <div class="absolute bottom-4 right-4 text-xs text-gray-500 bg-white/50 px-2 py-1 rounded">
            Tips: Tarik foto ke foto lain untuk menukar. Gunakan AI Magic untuk caption instan!
        </div>
    </div>

    <script>
        /* --- CONFIG --- */
        const apiKey = ""; // Disuntikkan oleh sistem runtime

        /* --- STATE --- */
        let images = [];
        const grid = document.getElementById('collage-grid');
        const board = document.getElementById('collage-board');
        const overlay = document.getElementById('overlay-layer');

        /* --- INITIALIZATION --- */
        document.addEventListener('DOMContentLoaded', () => {
            setupInputs();
            setupDragAndDrop(); // Setup HTML5 DnD for swapping
        });

        /* --- 1. MEDIA HANDLING --- */
        document.getElementById('file-input').addEventListener('change', (e) => {
            const files = Array.from(e.target.files);
            if(files.length === 0) return;

            // Add new files to our list
            files.forEach(file => {
                const reader = new FileReader();
                reader.onload = (e) => {
                    addImageToGrid(e.target.result);
                };
                reader.readAsDataURL(file);
            });

            // Clear placeholders if this is the first upload
            if(images.length === 0) {
                grid.innerHTML = '';
            }
        });

        function addImageToGrid(src) {
            images.push(src);
            renderSingleImage(src, images.length - 1);
        }

        function renderSingleImage(src, index) {
            // Remove a placeholder if it exists
            const placeholder = grid.querySelector('.border-dashed');
            if(placeholder) placeholder.remove();

            const div = document.createElement('div');
            div.className = 'img-item bg-gray-200 overflow-hidden relative group';
            div.draggable = true; // Enable drag for swapping
            
            // Apply current settings
            div.style.borderRadius = document.getElementById('radius-input').value + 'px';
            const currentFilter = document.querySelector('select').value;
            if(currentFilter) div.classList.add(currentFilter);

            const img = document.createElement('img');
            img.src = src;
            img.draggable = false; // Disable default img drag to allow our custom pan logic
            
            // PANNING LOGIC (Move image inside frame)
            let isPanning = false;
            let startX, startY, initialPosX, initialPosY;

            // Initialize object position if not set
            if(!img.style.objectPosition) img.style.objectPosition = '50% 50%';

            img.addEventListener('mousedown', (e) => {
                e.preventDefault(); // prevent native drag
                isPanning = true;
                startX = e.clientX;
                startY = e.clientY;
                
                // Parse current percentages
                const currentPos = img.style.objectPosition.split(' ');
                initialPosX = parseFloat(currentPos[0]) || 50;
                initialPosY = parseFloat(currentPos[1]) || 50;
                
                img.style.cursor = 'grabbing';
            });

            window.addEventListener('mousemove', (e) => {
                if(!isPanning) return;
                
                const dx = e.clientX - startX;
                const dy = e.clientY - startY;
                
                // Sensitivity factor
                const factor = 0.2; 
                
                let newX = initialPosX - (dx * factor);
                let newY = initialPosY - (dy * factor);
                
                // Clamp 0-100
                newX = Math.max(0, Math.min(100, newX));
                newY = Math.max(0, Math.min(100, newY));

                img.style.objectPosition = `${newX}% ${newY}%`;
            });

            window.addEventListener('mouseup', () => {
                isPanning = false;
                if(img) img.style.cursor = 'grab';
            });

            div.appendChild(img);
            
            // Drag Events for Swapping
            div.addEventListener('dragstart', handleDragStart);
            div.addEventListener('dragover', handleDragOver);
            div.addEventListener('dragleave', handleDragLeave);
            div.addEventListener('drop', handleDrop);
            div.addEventListener('dragend', handleDragEnd);

            grid.appendChild(div);
        }

        /* --- 2. DRAG & DROP SWAPPING LOGIC --- */
        let dragSrcEl = null;

        function handleDragStart(e) {
            dragSrcEl = this;
            e.dataTransfer.effectAllowed = 'move';
            e.dataTransfer.setData('text/html', this.innerHTML);
            this.style.opacity = '0.4';
        }

        function handleDragOver(e) {
            if (e.preventDefault) e.preventDefault();
            e.dataTransfer.dropEffect = 'move';
            this.classList.add('dragging-over');
            return false;
        }

        function handleDragLeave(e) {
            this.classList.remove('dragging-over');
        }

        function handleDrop(e) {
            if (e.stopPropagation) e.stopPropagation();
            this.classList.remove('dragging-over');

            if (dragSrcEl !== this) {
                // Swap the inner HTML (Image)
                const srcHTML = dragSrcEl.innerHTML;
                const targetHTML = this.innerHTML;
                
                // We also need to swap object-position styles if we want to preserve panning
                // But for simplicity in this version, we just swap content. 
                // Re-attaching event listeners for the new images would be needed for robustness,
                // but since we replace innerHTML, the old listeners are gone.
                // Simpler approach: Swap src attributes.
                
                const srcImg = dragSrcEl.querySelector('img');
                const targetImg = this.querySelector('img');

                if(srcImg && targetImg) {
                    const tempSrc = srcImg.src;
                    srcImg.src = targetImg.src;
                    targetImg.src = tempSrc;
                    
                    // Reset positions on swap
                    srcImg.style.objectPosition = '50% 50%';
                    targetImg.style.objectPosition = '50% 50%';
                }
            }
            return false;
        }

        function handleDragEnd(e) {
            this.style.opacity = '1';
            const items = document.querySelectorAll('.img-item');
            items.forEach(item => item.classList.remove('dragging-over'));
        }

        /* --- 3. LAYOUT & STYLES --- */
        function setLayout(type) {
            grid.className = 'w-full h-full'; 
            if(type === 'vertical') grid.style.minHeight = 'auto';
            else grid.style.minHeight = '568px';
            
            grid.classList.add(`layout-${type}`);
        }

        function setGlobalFilter(filter) {
            const items = document.querySelectorAll('.img-item');
            items.forEach(item => {
                item.className = item.className.replace(/filter-\w+/g, ''); // remove old
                if(filter) item.classList.add(filter);
            });
        }

        function setBgColor(color) {
            board.style.backgroundColor = color;
            board.style.backgroundImage = 'none';
        }

        // Sliders
        function setupInputs() {
            document.getElementById('gap-input').addEventListener('input', (e) => {
                grid.style.gap = e.target.value + 'px';
                document.getElementById('gap-val').textContent = e.target.value;
            });
            document.getElementById('padding-input').addEventListener('input', (e) => {
                board.style.padding = e.target.value + 'px';
                document.getElementById('padding-val').textContent = e.target.value;
            });
            document.getElementById('radius-input').addEventListener('input', (e) => {
                document.querySelectorAll('.img-item').forEach(el => el.style.borderRadius = e.target.value + 'px');
                document.getElementById('radius-val').textContent = e.target.value;
            });
        }

        /* --- 4. DRAGGABLE TEXT & STICKERS (INTERACT.JS) --- */
        
        // Setup Interact.js for draggable elements
        interact('.draggable-element').draggable({
            listeners: {
                move (event) {
                    const target = event.target;
                    // keep the dragged position in the data-x/data-y attributes
                    const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
                    const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;

                    // translate the element
                    target.style.transform = `translate(${x}px, ${y}px)`;

                    // update the posiion attributes
                    target.setAttribute('data-x', x);
                    target.setAttribute('data-y', y);
                }
            },
            modifiers: [
                interact.modifiers.restrictRect({
                    restriction: 'parent',
                    endOnly: true
                })
            ]
        });

        function createOverlayElement(content, type = 'text') {
            const el = document.createElement('div');
            el.className = 'draggable-element pointer-events-auto flex items-center justify-center group';
            el.style.top = '40%';
            el.style.left = '40%';
            
            // Delete button
            const delBtn = document.createElement('div');
            delBtn.className = 'delete-element-btn';
            delBtn.innerHTML = '<i class="fas fa-times"></i>';
            delBtn.onclick = () => el.remove();
            el.appendChild(delBtn);

            const inner = document.createElement('div');
            if(type === 'text') {
                inner.textContent = content;
                inner.className = 'text-2xl font-bold text-gray-800 font-serif text-center px-4 py-2';
                inner.contentEditable = true; // Allow editing text directly
                inner.style.textShadow = '0 2px 4px rgba(255,255,255,0.8)';
                inner.onclick = (e) => e.stopPropagation(); // Focus for editing
            } else {
                inner.textContent = content;
                inner.className = 'text-5xl drop-shadow-md'; // Sticker size
            }
            
            el.appendChild(inner);
            overlay.appendChild(el);
            
            // Subtle animation to show it appeared
            el.animate([
                { transform: 'scale(0.5)', opacity: 0 },
                { transform: 'scale(1)', opacity: 1 }
            ], { duration: 300, easing: 'ease-out' });
        }

        function addText(textOverride) {
            const input = document.getElementById('new-text-input');
            const text = textOverride || input.value.trim() || "Judul Anda";
            createOverlayElement(text, 'text');
            input.value = '';
        }

        function addSticker(emoji) {
            createOverlayElement(emoji, 'sticker');
        }

        /* --- 5. GEMINI API INTEGRATION --- */
        async function generateAICaption() {
            const theme = document.getElementById('ai-theme-input').value;
            const btn = document.getElementById('btn-ai-caption');
            const originalText = btn.innerHTML;

            if(!theme) {
                alert("Mohon masukkan tema terlebih dahulu (cth: Liburan, Kucing, Kopi)");
                return;
            }

            // Loading state
            btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Loading...';
            btn.disabled = true;

            const prompt = `Buatkan caption Instagram yang singkat (maksimal 10 kata), menarik, dan estetik dalam Bahasa Indonesia untuk tema: "${theme}". Jangan gunakan tanda kutip.`;

            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }]
                    })
                });

                if (!response.ok) throw new Error('API Error');

                const data = await response.json();
                const aiText = data.candidates[0].content.parts[0].text.trim();
                
                // Add the generated text to the canvas
                addText(aiText);

            } catch (error) {
                console.error("Gemini Error:", error);
                alert("Maaf, AI sedang sibuk. Coba lagi nanti.");
            } finally {
                btn.innerHTML = originalText;
                btn.disabled = false;
            }
        }

        async function generateAIStickers() {
            const theme = document.getElementById('ai-theme-input').value;
            const btn = document.getElementById('btn-ai-stickers');
            const originalText = btn.innerHTML;

            if(!theme) {
                alert("Mohon masukkan tema terlebih dahulu (cth: Liburan, Kucing, Kopi)");
                return;
            }

            // Loading state
            btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Loading...';
            btn.disabled = true;

            const prompt = `Berikan 5 emoji yang sangat relevan dan estetik untuk tema: "${theme}". Hanya berikan emoji saja, dipisahkan dengan spasi. Jangan ada teks lain.`;

            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }]
                    })
                });

                if (!response.ok) throw new Error('API Error');

                const data = await response.json();
                const text = data.candidates[0].content.parts[0].text;
                
                // Extract emojis (simple split by space or just taking characters)
                // Use a regex to find emoji-like characters if the split is messy, but space split usually works for this prompt
                const emojis = text.trim().split(/\s+/).slice(0, 5); // Take max 5

                // Add stickers with a slight delay for visual effect
                emojis.forEach((emoji, i) => {
                    setTimeout(() => {
                        // Random offset slightly so they don't stack perfectly
                        createOverlayElement(emoji, 'sticker');
                        
                        // Select the last added element and move it slightly
                        const els = document.querySelectorAll('.draggable-element');
                        const last = els[els.length - 1];
                        if(last) {
                            last.style.left = (40 + (i * 5)) + '%';
                            last.style.top = (40 + (i * 5)) + '%';
                        }
                    }, i * 200);
                });

            } catch (error) {
                console.error("Gemini Error:", error);
                alert("Maaf, AI sedang sibuk. Coba lagi nanti.");
            } finally {
                btn.innerHTML = originalText;
                btn.disabled = false;
            }
        }

        /* --- 6. DOWNLOAD --- */
        async function downloadCollage() {
            // Unselect things
            document.querySelectorAll('.draggable-element').forEach(el => {
                el.style.border = 'none';
                el.querySelector('.delete-element-btn').style.display = 'none';
            });

            try {
                const canvas = await html2canvas(board, { scale: 2, useCORS: true });
                const link = document.createElement('a');
                link.download = `studio-kolase-${Date.now()}.png`;
                link.href = canvas.toDataURL('image/png');
                link.click();
            } catch (err) {
                alert('Error making image');
            }
        }

    </script>
</body>
</html>
