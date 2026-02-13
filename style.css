document.addEventListener('DOMContentLoaded', () => {
    let items = JSON.parse(localStorage.getItem('myItems')) || [];
    let history = JSON.parse(localStorage.getItem('myHistory')) || [];
    let currentView = 'daily';
    let expenseChart = null;
    let map = null;

    const colors = { food: '#fb923c', daily: '#34d399', hobby: '#f472b6', other: '#94a3b8' };
    const catNames = { food: '食費', daily: '日用品', hobby: '趣味', other: '他' };

    // --- 1. グラフ初期化 ---
    const ctx = document.getElementById('expenseChart').getContext('2d');
    expenseChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: [],
            datasets: ['food', 'daily', 'hobby', 'other'].map(cat => ({
                label: catNames[cat],
                data: [],
                backgroundColor: colors[cat],
                stack: 'stack0'
            }))
        },
        options: { 
            responsive: true, 
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: { x: { stacked: true }, y: { stacked: true, beginAtZero: true } }
        }
    });

    // --- 2. 画面更新(計算)ロジック ---
    function updateUI() {
        const totalBudget = parseInt(document.getElementById('monthlyBudget').value) || 0;
        const pastTotalSpent = history.reduce((sum, h) => sum + h.total, 0);
        
        let currentListTotal = 0;
        const itemList = document.getElementById('itemList');
        itemList.innerHTML = '';

        items.forEach((item, index) => {
            currentListTotal += item.price;
            const li = document.createElement('li');
            li.innerHTML = `<span><span class="badge bg-${item.category}">${catNames[item.category]}</span>${item.name}</span>
                            <span>¥${item.price.toLocaleString()} <button class="del-btn" data-index="${index}" style="border:none; background:none; color:red; cursor:pointer;">✕</button></span>`;
            itemList.appendChild(li);
        });

        document.getElementById('totalPrice').innerText = `¥${currentListTotal.toLocaleString()}`;
        const remaining = totalBudget - pastTotalSpent - currentListTotal;
        const remDisplay = document.getElementById('remainingBalance');
        remDisplay.innerText = `¥${remaining.toLocaleString()}`;
        remaining < 0 ? remDisplay.classList.add('negative') : remDisplay.classList.remove('negative');

        // グラフ更新
        const labels = [...new Set(history.map(h => {
            if (currentView === 'weekly') {
                const d = new Date(h.fullDate);
                return `${d.getMonth() + 1}月 第${Math.ceil(d.getDate() / 7)}週`;
            }
            return h.date;
        }))];

        expenseChart.data.labels = labels;
        ['food', 'daily', 'hobby', 'other'].forEach((cat, i) => {
            expenseChart.data.datasets[i].data = labels.map(label => {
                return history.filter(h => {
                    const hLabel = currentView === 'weekly' ? `${new Date(h.fullDate).getMonth() + 1}月 第${Math.ceil(new Date(h.fullDate).getDate() / 7)}週` : h.date;
                    return hLabel === label;
                }).reduce((sum, h) => sum + (h.breakdown ? h.breakdown[cat] || 0 : 0), 0);
            });
        });
        expenseChart.update();

        localStorage.setItem('myItems', JSON.stringify(items));
        localStorage.setItem('myHistory', JSON.stringify(history));
        localStorage.setItem('myBudget', totalBudget);
    }

    // --- 3. イベント登録 ---
    document.getElementById('addBtn').addEventListener('click', () => {
        const name = document.getElementById('itemName').value;
        const price = parseInt(document.getElementById('itemPrice').value);
        const category = document.getElementById('itemCategory').value;
        if (name && !isNaN(price)) {
            items.push({ name, price, category });
            document.getElementById('itemName').value = '';
            document.getElementById('itemPrice').value = '';
            updateUI();
        }
    });

    document.getElementById('clearBtn').addEventListener('click', () => {
        if (items.length === 0) return;
        const total = items.reduce((sum, i) => sum + i.price, 0);
        const breakdown = { food: 0, daily: 0, hobby: 0, other: 0 };
        items.forEach(i => breakdown[i.category] += i.price);
        const now = new Date();
        history.push({ 
            date: `${now.getMonth() + 1}/${now.getDate()}`, 
            total: total, 
            breakdown: breakdown, 
            fullDate: now.toISOString() 
        });
        items = [];
        updateUI();
    });

    // マップ機能
    document.getElementById('findStoresBtn').addEventListener('click', function() {
        document.getElementById('map').style.display = 'block';
        if (!navigator.geolocation) return alert("位置情報非対応です");

        navigator.geolocation.getCurrentPosition((pos) => {
            const lat = pos.coords.latitude;
            const lon = pos.coords.longitude;
            if (!map) {
                map = L.map('map').setView([lat, lon], 15);
                L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
                L.marker([lat, lon]).addTo(map).bindPopup("現在地").openPopup();

                fetch(`https://overpass-api.de/api/interpreter?data=[out:json];node["shop"="supermarket"](around:2000,${lat},${lon});out;`)
                .then(res => res.json())
                .then(data => {
                    data.elements.forEach(s => {
                        L.marker([s.lat, s.lon]).addTo(map).bindPopup(s.tags.name || "スーパー");
                    });
                });
            }
        });
    });

    // 削除ボタン・切り替え
    document.getElementById('itemList').addEventListener('click', (e) => { if (e.target.classList.contains('del-btn')) { items.splice(e.target.dataset.index, 1); updateUI(); } });
    document.getElementById('viewDaily').addEventListener('click', (e) => { currentView = 'daily'; setActiveBtn(e.target); updateUI(); });
    document.getElementById('viewWeekly').addEventListener('click', (e) => { currentView = 'weekly'; setActiveBtn(e.target); updateUI(); });
    function setActiveBtn(btn) { document.querySelectorAll('.view-selector button').forEach(b => b.classList.remove('active')); btn.classList.add('active'); }
    document.getElementById('monthlyBudget').addEventListener('input', updateUI);

    updateUI();
});
