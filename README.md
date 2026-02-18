<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>伴手禮紀錄助手</title>
    
    <!-- PWA / 行動裝置 App 化設定 -->
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="伴手禮小幫手">
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/1162/1162283.png">
    
    <!-- 引入外部套件 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <style>
        * { -webkit-tap-highlight-color: transparent; }
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.3);
        }
        ::-webkit-scrollbar { display: none; }
        body { 
            padding-top: env(safe-area-inset-top); 
            padding-bottom: env(safe-area-inset-bottom); 
            font-family: "PingFang TC", "Microsoft JhengHei", sans-serif;
        }
        .loading-overlay {
            position: fixed;
            inset: 0;
            background: rgba(255,255,255,0.9);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 9999;
            font-weight: bold;
            color: #1e293b;
        }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .nav-btn.active { color: #2563eb; }
        .dimension-btn.active { 
            background-color: #1e293b; 
            color: white;
            border-color: #1e293b;
        }
        /* 圖片容器焦點樣式 */
        .image-container:focus {
            ring: 2px;
            ring-color: #3b82f6;
            outline: none;
        }
    </style>
</head>
<body class="bg-slate-100 min-h-screen pb-24 font-sans text-slate-900 select-none">

    <!-- 載入中遮罩 -->
    <div id="auth-loading" class="loading-overlay">☁️ 正在連接雲端資料庫...</div>

    <!-- 頂部狀態欄 -->
    <nav class="sticky top-0 z-50 bg-white/90 backdrop-blur-md border-b">
        <div class="max-w-xl mx-auto px-6 py-4">
            <div class="flex justify-between items-center mb-4">
                <h1 class="text-xl font-black text-slate-800 tracking-tight">伴手禮助手 🎁</h1>
                <div id="exchange-info" class="text-[10px] text-slate-400 font-mono bg-slate-50 px-2 py-1 rounded-full border">載入匯率中...</div>
            </div>
            
            <div class="grid grid-cols-2 gap-3">
                <div class="bg-blue-600 p-3 rounded-2xl text-white shadow-lg shadow-blue-200">
                    <p class="text-[9px] font-bold opacity-80 uppercase tracking-widest">預算規劃 (TWD)</p>
                    <p class="text-lg font-black" id="planned-total">$0</p>
                </div>
                <div class="bg-emerald-500 p-3 rounded-2xl text-white shadow-lg shadow-emerald-200">
                    <p class="text-[9px] font-bold opacity-80 uppercase tracking-widest">目前已花 (TWD)</p>
                    <p class="text-lg font-black" id="purchased-total">$0</p>
                </div>
            </div>
        </div>
    </nav>

    <main class="max-w-xl mx-auto px-6 pt-6">
        
        <!-- 分頁 1：清單管理 -->
        <div id="tab-list" class="tab-content active">
            
            <!-- 標籤設定摺疊面板 -->
            <section class="mb-6">
                <details class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
                    <summary class="px-5 py-3 cursor-pointer font-bold text-slate-600 text-sm flex justify-between items-center">
                        🏷️ 標籤與對象設定
                        <span class="text-[10px] text-slate-300">點擊編輯</span>
                    </summary>
                    <div class="p-5 border-t border-slate-100 bg-slate-50/50 space-y-4">
                        <div>
                            <h3 class="text-[10px] font-black text-slate-400 mb-2 uppercase tracking-widest">👤 贈送對象</h3>
                            <div class="flex gap-2 mb-2">
                                <input type="text" id="tag-person-name" placeholder="例如：同事" class="flex-1 px-4 py-2 border rounded-xl text-xs outline-none">
                                <input type="color" id="tag-person-color" value="#3b82f6" class="w-10 h-10 p-1 bg-white border rounded-xl">
                                <button onclick="addTag('person')" class="bg-blue-600 text-white px-4 rounded-xl text-xs font-bold">新增</button>
                            </div>
                            <div id="tags-person-list" class="flex flex-wrap gap-2"></div>
                        </div>
                        <div>
                            <h3 class="text-[10px] font-black text-slate-400 mb-2 uppercase tracking-widest">📍 購買地點</h3>
                            <div class="flex gap-2 mb-2">
                                <input type="text" id="tag-location-name" placeholder="例如：免稅店" class="flex-1 px-4 py-2 border rounded-xl text-xs outline-none">
                                <input type="color" id="tag-location-color" value="#f59e0b" class="w-10 h-10 p-1 bg-white border rounded-xl">
                                <button onclick="addTag('location')" class="bg-amber-600 text-white px-4 rounded-xl text-xs font-bold">新增</button>
                            </div>
                            <div id="tags-location-list" class="flex flex-wrap gap-2"></div>
                        </div>
                    </div>
                </details>
            </section>

            <!-- 新增表單 -->
            <section class="mb-10">
                <div class="bg-white p-6 rounded-[2.5rem] shadow-xl shadow-slate-200/50 border border-white">
                    <div class="space-y-4">
                        <div class="grid grid-cols-3 gap-3">
                            <div>
                                <label class="block text-[9px] font-black text-slate-400 mb-1 uppercase">幣別</label>
                                <select id="item-currency" class="w-full px-3 py-3 border rounded-2xl text-sm bg-slate-50 font-bold outline-none">
                                    <option value="TWD">TWD 台幣</option>
                                    <option value="KRW">KRW 韓元</option>
                                    <option value="JPY">JPY 日圓</option>
                                    <option value="USD">USD 美金</option>
                                    <option value="THB">THB 泰銖</option>
                                </select>
                            </div>
                            <div class="col-span-2">
                                <label class="block text-[9px] font-black text-slate-400 mb-1 uppercase">商品名稱</label>
                                <input type="text" id="item-name" placeholder="輸入名稱" class="w-full px-4 py-3 border rounded-2xl text-sm outline-none bg-slate-50">
                            </div>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-[9px] font-black text-slate-400 mb-1 uppercase">當地價格</label>
                                <input type="number" id="item-price" placeholder="0" class="w-full px-4 py-3 border rounded-2xl text-sm bg-slate-50 font-mono font-bold">
                            </div>
                            <div>
                                <label class="block text-[9px] font-black text-blue-600 mb-1 uppercase font-black">📸 點我並貼上圖片</label>
                                <div id="form-image-preview" tabindex="0" class="w-full h-11 bg-slate-50 border-2 border-dashed border-slate-200 rounded-2xl flex items-center justify-center overflow-hidden cursor-pointer focus:border-blue-500 focus:bg-blue-50 outline-none" onclick="this.focus();">
                                    <span class="text-[9px] text-slate-300 font-bold uppercase tracking-widest">Tap to Paste</span>
                                </div>
                            </div>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-[9px] font-black text-slate-400 mb-1 uppercase">贈送對象</label>
                                <select id="item-person-select" class="w-full px-4 py-3 border rounded-2xl text-xs bg-slate-50 outline-none"></select>
                            </div>
                            <div>
                                <label class="block text-[9px] font-black text-slate-400 mb-1 uppercase">購買地點</label>
                                <select id="item-location-select" class="w-full px-4 py-3 border rounded-2xl text-xs bg-slate-50 outline-none"></select>
                            </div>
                        </div>
                        <button id="add-btn" onclick="addItem()" class="w-full bg-slate-900 text-white font-black py-4 rounded-2xl shadow-lg active:scale-95 transition-all text-xl tracking-widest">
                            新增至清單
                        </button>
                    </div>
                </div>
            </section>

            <!-- 清單列表區域 -->
            <div id="items-grid" class="grid grid-cols-1 gap-6"></div>
            <div id="empty-state" class="text-center py-20 text-slate-300">
                <p class="text-4xl mb-4">🛒</p>
                <p class="text-xs font-black uppercase tracking-widest">目前清單空空的</p>
            </div>
        </div>

        <!-- 分頁 2：圓餅圖統計 -->
        <div id="tab-stats" class="tab-content">
            <div class="bg-white p-6 rounded-[2.5rem] shadow-xl border border-white space-y-6">
                <div class="flex flex-col gap-3">
                    <h2 class="text-xs font-black text-slate-400 uppercase tracking-widest">分析視角</h2>
                    <div class="flex bg-slate-100 p-1 rounded-xl gap-1">
                        <button onclick="changeDimension('person')" id="dim-person" class="dimension-btn active flex-1 py-2 text-[10px] font-black rounded-lg border border-transparent transition-all">按對象</button>
                        <button onclick="changeDimension('location')" id="dim-location" class="dimension-btn flex-1 py-2 text-[10px] font-black rounded-lg border border-transparent transition-all">按地點</button>
                        <button onclick="changeDimension('currency')" id="dim-currency" class="dimension-btn flex-1 py-2 text-[10px] font-black rounded-lg border border-transparent transition-all">按幣別</button>
                    </div>
                </div>
                <hr class="border-slate-100">
                <div class="text-center">
                    <h3 id="current-dimension-title" class="text-lg font-black text-slate-800 mb-4">支出分析 (TWD)</h3>
                    <div class="aspect-square max-w-[300px] mx-auto relative">
                        <canvas id="main-chart"></canvas>
                    </div>
                    <div id="stats-breakdown" class="mt-8 space-y-2 text-left"></div>
                </div>
            </div>
        </div>
    </main>

    <!-- 底部導覽欄 -->
    <div class="fixed bottom-0 inset-x-0 bg-white/90 backdrop-blur-xl border-t px-10 py-4 flex justify-around items-center z-50">
        <button id="nav-list-btn" onclick="switchTab('list')" class="nav-btn active flex flex-col items-center gap-1">
            <span class="text-2xl">📝</span>
            <span class="text-[10px] font-black uppercase tracking-tighter">購物清單</span>
        </button>
        <button id="nav-stats-btn" onclick="switchTab('stats')" class="nav-btn flex flex-col items-center gap-1 text-slate-400">
            <span class="text-2xl">📊</span>
            <span class="text-[10px] font-black uppercase tracking-tighter">花費統計</span>
        </button>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, addDoc, updateDoc, deleteDoc, onSnapshot, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase 配置 - 請在擁有自己的專案後替換此處
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
            apiKey: "", 
            authDomain: "",
            projectId: "",
            storageBucket: "",
            messagingSenderId: "",
            appId: ""
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'my-souvenir-tracker-v1';

        const getPublicDataRef = (collectionName) => collection(db, 'artifacts', appId, 'public', 'data', collectionName);
        const getPublicDocRef = (collectionName, docId) => doc(db, 'artifacts', appId, 'public', 'data', collectionName, docId);

        let items = [];
        let tags = {
            person: [{ id: 'p1', name: '自己', color: '#64748b' }],
            location: [{ id: 'l1', name: '免稅店', color: '#10b981' }]
        };
        let exchangeRates = { TWD: 1 };
        let currentFormImage = null;
        let currentUser = null;
        let mainChart = null;
        let currentDimension = 'person';

        // 核心功能：圖片壓縮 (避免 Firestore 1MB 文件限制)
        async function compressImage(base64Str, maxWidth = 800, maxHeight = 800, quality = 0.6) {
            return new Promise((resolve) => {
                const img = new Image();
                img.src = base64Str;
                img.onload = () => {
                    const canvas = document.createElement('canvas');
                    let width = img.width;
                    let height = img.height;
                    if (width > height) {
                        if (width > maxWidth) { height *= maxWidth / width; width = maxWidth; }
                    } else {
                        if (height > maxHeight) { width *= maxHeight / height; height = maxHeight; }
                    }
                    canvas.width = width;
                    canvas.height = height;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0, width, height);
                    resolve(canvas.toDataURL('image/jpeg', quality));
                };
            });
        }

        // 初始化驗證
        const initAuth = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (err) { console.error("Auth Error:", err); }
        };

        onAuthStateChanged(auth, (user) => {
            currentUser = user;
            if (user) {
                document.getElementById('auth-loading').classList.add('hidden');
                startDataListener();
            }
        });

        function startDataListener() {
            if (!currentUser) return;
            // 監聽清單變動
            onSnapshot(getPublicDataRef('items'), (snapshot) => {
                items = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                // 排序邏輯：未支付在前，已支付在後，兩者內部分別依時間降序
                items.sort((a, b) => {
                    if (a.bought !== b.bought) return a.bought ? 1 : -1;
                    return (b.createdAt || 0) - (a.createdAt || 0);
                });
                renderItems();
                updateSummary();
                if (document.getElementById('tab-stats').classList.contains('active')) renderCharts();
            });

            // 監聽標籤設定
            onSnapshot(getPublicDocRef('settings', 'global'), (snapshot) => {
                if (snapshot.exists()) {
                    tags = snapshot.data().tags || tags;
                    renderTags();
                    renderItems();
                } else { 
                    saveTags(); 
                }
            });
        }

        async function saveTags() {
            if (!currentUser) return;
            await setDoc(getPublicDocRef('settings', 'global'), { tags });
        }

        window.switchTab = (tabName) => {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(el => el.classList.remove('active', 'text-blue-600'));
            document.querySelectorAll('.nav-btn').forEach(el => el.classList.add('text-slate-400'));
            document.getElementById(`tab-${tabName}`).classList.add('active');
            const activeBtn = document.getElementById(`nav-${tabName}-btn`);
            activeBtn.classList.add('active', 'text-blue-600');
            activeBtn.classList.remove('text-slate-400');
            if (tabName === 'stats') renderCharts();
        };

        window.changeDimension = (dim) => {
            currentDimension = dim;
            document.querySelectorAll('.dimension-btn').forEach(btn => btn.classList.remove('active'));
            document.getElementById(`dim-${dim}`).classList.add('active');
            renderCharts();
        };

        window.addTag = async (type) => {
            const name = document.getElementById(`tag-${type}-name`).value.trim();
            const color = document.getElementById(`tag-${type}-color`).value;
            if (!name) return;
            tags[type].push({ id: type[0] + Date.now(), name, color });
            document.getElementById(`tag-${type}-name`).value = '';
            await saveTags();
        };

        window.deleteTag = async (type, id) => {
            tags[type] = tags[type].filter(t => t.id !== id);
            await saveTags();
        };

        window.addItem = async () => {
            if (!currentUser) return;
            const name = document.getElementById('item-name').value.trim();
            const price = parseFloat(document.getElementById('item-price').value) || 0;
            const currency = document.getElementById('item-currency').value;
            if (!name) return;
            
            const btn = document.getElementById('add-btn');
            btn.disabled = true;
            btn.innerText = "處理中...";
            
            try {
                await addDoc(getPublicDataRef('items'), {
                    name, price, currency,
                    priceInTWD: currency === 'TWD' ? price : (price / (exchangeRates[currency] || 1)),
                    personTagId: document.getElementById('item-person-select').value,
                    locationTagId: document.getElementById('item-location-select').value,
                    bought: false,
                    image: currentFormImage,
                    createdAt: Date.now()
                });
                currentFormImage = null;
                document.getElementById('form-image-preview').innerHTML = `<span class="text-[9px] text-slate-300 font-bold uppercase tracking-widest">Tap to Paste</span>`;
                document.getElementById('item-name').value = '';
                document.getElementById('item-price').value = '';
            } catch (err) {
                console.error("Firebase Error:", err);
            } finally { 
                btn.disabled = false; 
                btn.innerText = "新增至清單";
            }
        };

        window.toggleBought = async (id) => {
            if (!currentUser) return;
            const docRef = getPublicDocRef('items', id);
            try {
                const snap = await getDoc(docRef);
                if (snap.exists()) {
                    await updateDoc(docRef, { bought: !snap.data().bought });
                }
            } catch (err) { console.error("Update Error:", err); }
        };

        window.deleteItem = async (id) => {
            if (!currentUser) return;
            if (confirm('確定要刪除這筆紀錄嗎？')) {
                try {
                    await deleteDoc(getPublicDocRef('items', id));
                } catch (err) { console.error("Delete Error:", err); }
            }
        };

        // 剪貼簿圖片處理
        window.addEventListener('paste', function(e) {
            const items = (e.clipboardData || e.originalEvent.clipboardData).items;
            let blob = null;
            for (let i = 0; i < items.length; i++) {
                if (items[i].type.indexOf("image") !== -1) { blob = items[i].getAsFile(); break; }
            }
            if (blob) {
                const reader = new FileReader();
                reader.onload = async function(event) {
                    const compressed = await compressImage(event.target.result);
                    const activeEl = document.activeElement;
                    if (activeEl.id === 'form-image-preview') {
                        currentFormImage = compressed;
                        activeEl.innerHTML = `<img src="${compressed}" class="w-full h-full object-cover">`;
                    } else if (activeEl.classList.contains('image-container')) {
                        const id = activeEl.getAttribute('data-id');
                        const docRef = getPublicDocRef('items', id);
                        const snap = await getDoc(docRef);
                        if (snap.exists()) {
                            await updateDoc(docRef, { image: compressed });
                        }
                    }
                };
                reader.readAsDataURL(blob);
            }
        }, true);

        function renderCharts() {
            const canvasEl = document.getElementById('main-chart');
            if (!canvasEl) return;
            const ctx = canvasEl.getContext('2d');
            const breakdownEl = document.getElementById('stats-breakdown');
            breakdownEl.innerHTML = '';
            const stats = {};
            
            items.forEach(item => {
                let label, color;
                if (currentDimension === 'person') {
                    const tag = tags.person.find(t => t.id === item.personTagId);
                    label = tag ? tag.name : '未分類對象';
                    color = tag ? tag.color : '#cbd5e1';
                } else if (currentDimension === 'location') {
                    const tag = tags.location.find(t => t.id === item.locationTagId);
                    label = tag ? tag.name : '未分類地點';
                    color = tag ? tag.color : '#cbd5e1';
                } else {
                    label = item.currency;
                    const currencyColors = { TWD: '#3b82f6', KRW: '#f59e0b', JPY: '#ef4444', USD: '#10b981', THB: '#8b5cf6' };
                    color = currencyColors[label] || '#94a3b8';
                }
                if (!stats[label]) stats[label] = { val: 0, color };
                stats[label].val += item.priceInTWD || 0;
            });

            const labels = Object.keys(stats);
            const data = Object.values(stats).map(s => Math.round(s.val));
            const colors = Object.values(stats).map(s => s.color);
            const total = data.reduce((a, b) => a + b, 0);

            labels.forEach((label, idx) => {
                const val = data[idx];
                const pct = total > 0 ? Math.round((val / total) * 100) : 0;
                breakdownEl.innerHTML += `
                    <div class="flex items-center justify-between">
                        <div class="flex items-center gap-2">
                            <span class="w-2 h-2 rounded-full" style="background:${colors[idx]}"></span>
                            <span class="text-xs font-bold text-slate-600">${label}</span>
                        </div>
                        <div class="text-xs font-mono font-black text-slate-400">
                            $${val.toLocaleString()} <span class="text-[10px] opacity-50">(${pct}%)</span>
                        </div>
                    </div>`;
            });

            if (mainChart) mainChart.destroy();
            mainChart = new Chart(ctx, {
                type: 'doughnut',
                data: { labels, datasets: [{ data, backgroundColor: colors, borderWidth: 5, borderColor: '#ffffff', hoverOffset: 15 }] },
                options: { cutout: '65%', plugins: { legend: { display: false } } }
            });
        }

        function renderTags() {
            ['person', 'location'].forEach(type => {
                const list = document.getElementById(`tags-${type}-list`);
                const select = document.getElementById(`item-${type}-select`);
                if(!list || !select) return;
                list.innerHTML = ''; select.innerHTML = '<option value="">未選擇</option>';
                tags[type].forEach(tag => {
                    list.innerHTML += `
                        <span class="inline-flex items-center gap-1 px-2 py-1 bg-white border rounded-xl text-[10px] font-bold shadow-sm">
                            <span class="w-1.5 h-1.5 rounded-full" style="background:${tag.color}"></span>
                            ${tag.name}
                            <button onclick="deleteTag('${type}','${tag.id}')" class="ml-1 text-slate-300 hover:text-red-500">✕</button>
                        </span>`;
                    select.innerHTML += `<option value="${tag.id}">${tag.name}</option>`;
                });
            });
        }

        function renderItems() {
            const grid = document.getElementById('items-grid');
            if(!grid) return;
            grid.innerHTML = '';
            items.forEach(item => {
                const pTag = tags.person.find(t => t.id === item.personTagId);
                const lTag = tags.location.find(t => t.id === item.locationTagId);
                const statusClass = item.bought ? 'border-emerald-400 ring-4 ring-emerald-50' : 'border-white';
                
                grid.innerHTML += `
                    <div class="glass-card rounded-[2rem] overflow-hidden shadow-lg border transition-all duration-300 ${statusClass}">
                        <div class="flex">
                            <div class="w-28 h-28 bg-slate-200 image-container relative shrink-0 focus:ring-2 focus:ring-blue-400" data-id="${item.id}" tabindex="0" onclick="this.focus();">
                                ${item.image ? `<img src="${item.image}" class="w-full h-full object-cover">` : `<div class="flex items-center justify-center h-full text-xl opacity-20">📸</div>`}
                                ${item.bought ? `<div class="absolute inset-0 bg-emerald-500/10 flex items-center justify-center"><span class="bg-white text-emerald-500 text-[9px] font-black px-2 py-0.5 rounded-full shadow-md">PAID</span></div>` : ''}
                            </div>
                            <div class="p-4 flex-1 flex flex-col justify-between min-w-0">
                                <div class="min-w-0">
                                    <div class="flex gap-1 mb-1 overflow-x-auto no-scrollbar">
                                        ${pTag ? `<span class="px-1.5 py-0.5 rounded-full text-[8px] font-black shrink-0" style="background:${pTag.color}15; color:${pTag.color}">${pTag.name}</span>` : ''}
                                        ${lTag ? `<span class="px-1.5 py-0.5 rounded-full text-[8px] font-black shrink-0" style="background:${lTag.color}15; color:${lTag.color}">${lTag.name}</span>` : ''}
                                    </div>
                                    <h3 class="font-black text-slate-800 text-sm truncate">${item.name}</h3>
                                    <p class="text-xs font-mono font-black text-blue-600">${item.currency} ${item.price.toLocaleString()}</p>
                                </div>
                                <div class="flex gap-2 mt-2">
                                    <button onclick="toggleBought('${item.id}')" class="flex-1 py-1.5 rounded-xl text-[9px] font-black transition-colors ${item.bought ? 'bg-emerald-500 text-white' : 'bg-slate-100 text-slate-500'} uppercase">
                                        ${item.bought ? '已支付' : '待支付'}
                                    </button>
                                    <button onclick="deleteItem('${item.id}')" class="px-2 text-slate-300 hover:text-red-500">✕</button>
                                </div>
                            </div>
                        </div>
                    </div>`;
            });
            document.getElementById('empty-state').classList.toggle('hidden', items.length > 0);
        }

        function updateSummary() {
            const planned = items.filter(i => !i.bought).reduce((s, i) => s + (i.priceInTWD || 0), 0);
            const purchased = items.filter(i => i.bought).reduce((s, i) => s + (i.priceInTWD || 0), 0);
            document.getElementById('planned-total').innerText = `$${Math.round(planned).toLocaleString()}`;
            document.getElementById('purchased-total').innerText = `$${Math.round(purchased).toLocaleString()}`;
        }

        async function fetchRates() {
            try {
                const res = await fetch('https://open.er-api.com/v6/latest/TWD');
                const data = await res.json();
                exchangeRates = data.rates;
                document.getElementById('exchange-info').innerText = `1 TWD ≈ ${exchangeRates.KRW.toFixed(1)} KRW / ${exchangeRates.JPY.toFixed(1)} JPY`;
            } catch(e) { 
                document.getElementById('exchange-info').innerText = `匯率獲取失敗`;
            }
        }

        window.onload = () => { fetchRates(); initAuth(); };
    </script>
</body>
</html>

