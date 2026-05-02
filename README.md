
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>原鄉谷尊榮導覽 - 行程表</title>
    <!-- 引入核心函式庫 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; -webkit-tap-highlight-color: transparent; }
        .font-georgia { font-family: Georgia, serif; }
        
        /* 絕對修復跑位問題：預留固定高度空間 */
        .fixed-avatar-box { width: 140px !important; height: 140px !important; flex-shrink: 0; }
        .fixed-car-box { width: 100% !important; height: 220px !important; flex-shrink: 0; }
        
        .smooth-scroll { scroll-behavior: smooth; }
        
        /* AI 載入動畫 */
        .ai-pulse {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: .5; }
        }
    </style>
</head>
<body class="bg-[#F3F4F6] smooth-scroll">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // --- Gemini API 基礎配置 ---
        const apiKey = ""; 

        const fetchWithRetry = async (url, options, retries = 5, backoff = 1000) => {
            try {
                const response = await fetch(url, options);
                if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                return await response.json();
            } catch (error) {
                if (retries > 0) {
                    await new Promise(resolve => setTimeout(resolve, backoff));
                    return fetchWithRetry(url, options, retries - 1, backoff * 2);
                }
                throw error;
            }
        };

        const App = () => {
            const [activeDay, setActiveDay] = useState(1);
            const [avatar, setAvatar] = useState(null);
            const [carView, setCarView] = useState(null);
            const itineraryRef = useRef(null);

            // AI 狀態
            const [aiWelcome, setAiWelcome] = useState("");
            const [isWelcomeLoading, setIsWelcomeLoading] = useState(false);
            const [aiOutfit, setAiOutfit] = useState("");
            const [isOutfitLoading, setIsOutfitLoading] = useState(false);
            const [foodLoadingPoint, setFoodLoadingPoint] = useState(null);

            const reservation = {
                name: "陳佳君", phone: "+65 9877 0843", people: "3位大人2小孩", note: "尊榮包車五日方案",
                flight: "TR 896", time: "17:05", date: "2026/03/12", terminal: "第一航廈",
                driver: "張其芳", driverPhone: "0935-089253", carPlate: "TEB-6933"
            };

            const itineraryData = [
                { day: 1, date: "2026年3月12日 星期四", isPickup: true, points: [{ name: "桃園機場", desc: "專屬接機服務：尊榮旅程即將開啟", type: "airport" }, { name: "羅東住宿", desc: "抵達飯店辦理入宿休息", type: "hotel" }], fare: 3000 },
                { day: 2, date: "2026年3月13日 星期五", start: "08:30 出發 (優宿商旅)", points: [{ name: "清水地熱公園", desc: "體驗地熱煮食樂趣", time: "50" }, { name: "稻香園", desc: "午餐推薦：在地特色料理", time: "30" }, { name: "奇麗灣珍奶博物館", desc: "DIY 珍珠奶茶體驗 (13:00預約)", time: "40" }, { name: "北后寺", desc: "精緻禪意建築參訪", time: "30" }, { name: "登篙林道", desc: "漫步林道芬多精", time: "45" }, { name: "望龍埤花田村", desc: "偶像劇景點湖光山色", time: "20" }, { name: "羅東夜市", desc: "宜蘭美食與自由活動", type: "hotel" }], fare: 4500 },
                { day: 3, date: "2026年3月14日 星期六", start: "09:00 出發 (優宿商旅)", points: [{ name: "中華路永和豆漿", desc: "傳統中式早餐", time: "40" }, { name: "明池山莊", desc: "【高海拔警示】氣溫較低，請務必備齊保暖衣物", time: "90", warning: true }, { name: "星寶體驗農場", desc: "三星蔥DIY與小動物互動", time: "50" }, { name: "梅花湖", desc: "環湖騎行悠閒時光", time: "40" }, { name: "礁溪溫泉住宿", desc: "飯店休息、享受溫泉設施", type: "hotel" }], fare: 5000 },
                { day: 4, date: "2026年3月15日 星期日", start: "09:00 出發 (品文旅礁溪)", points: [{ name: "賜福自助餐", desc: "在地早餐選擇", time: "30" }, { name: "蘭陽植物王國", desc: "豐富多樣植被景觀", time: "40" }, { name: "百匯窯仔雞", desc: "午餐推薦：特色甕窯雞", time: "30" }, { name: "斑比山丘", desc: "與梅花鹿互動體驗", time: "40" }, { name: "五峰旗瀑布", desc: "壯麗瀑布景觀步道", time: "45" }, { name: "礁溪夜市", desc: "美食探索與自由晚餐", type: "hotel" }], fare: 4500 },
                { day: 5, date: "2026年3月16日 星期一", start: "08:30 出發 (品文旅礁溪)", points: [{ name: "清珍早點", desc: "知名傳統早餐老店", time: "40" }, { name: "龜山島賞鯨", desc: "海上探險、賞鯨豚行程 (10:00預約)", time: "60" }, { name: "幸福36號海鮮餐廳", desc: "南方澳現撈海鮮饗宴", time: "50" }, { name: "鼻頭角步道", desc: "海天一色絕美長城步道", time: "70" }, { name: "基隆夜市", desc: "基隆廟口美食巡禮 (預計18:00離場)", time: "40" }, { name: "台北路徒Plus行旅", desc: "行程平安結束，祝旅途愉快", type: "hotel" }], fare: 5500 }
            ];

            const totalFare = itineraryData.reduce((sum, d) => sum + d.fare, 0);

            // --- AI 核心功能 ---
            const handleAiWelcome = async () => {
                if (aiWelcome) return;
                setIsWelcomeLoading(true);
                try {
                    const prompt = `你是原鄉谷包車VIP導遊。請為貴賓 ${reservation.name} 寫一段 50 字內的熱情、優雅且具質感的歡迎辭。`;
                    const res = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST', headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
                    });
                    setAiWelcome(res.candidates?.[0]?.content?.parts?.[0]?.text);
                } finally { setIsWelcomeLoading(false); }
            };

            const handleAiOutfit = async () => {
                setIsOutfitLoading(true);
                try {
                    const day = itineraryData[activeDay - 1];
                    const prompt = `根據日期：${day.date}，景點包括：${day.points.map(p => p.name).join(', ')}。分析宜蘭三月氣候，給予專業穿著建議。`;
                    const res = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST', headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
                    });
                    setAiOutfit(res.candidates?.[0]?.content?.parts?.[0]?.text);
                } finally { setIsOutfitLoading(false); }
            };

            const handleAiFood = async (pName) => {
                setFoodLoadingPoint(pName);
                try {
                    const prompt = `請搜尋「${pName}」景點附近評價最高的 2 個真實美食店家。`;
                    const res = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST', headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ 
                            contents: [{ parts: [{ text: prompt }] }],
                            tools: [{ "google_search": {} }]
                        })
                    });
                    alert(`✨ AI 精選推薦「${pName}」美食：\n\n${res.candidates?.[0]?.content?.parts?.[0]?.text}`);
                } catch(e) {
                    alert("目前 API 忙碌中，建議詢問您的司機張先生。");
                } finally { setFoodLoadingPoint(null); }
            };

            // 圖片上傳邏輯 (Canvas 清理防黑底)
            const handleImage = (e, type) => {
                const file = e.target.files[0];
                if (!file) return;
                const reader = new FileReader();
                reader.onload = (ev) => {
                    const img = new Image();
                    img.onload = () => {
                        const cvs = document.createElement('canvas');
                        cvs.width = img.width; cvs.height = img.height;
                        const ctx = cvs.getContext('2d');
                        ctx.clearRect(0, 0, cvs.width, cvs.height);
                        ctx.drawImage(img, 0, 0);
                        const data = cvs.toDataURL('image/png');
                        if (type === 'avatar') setAvatar(data);
                        else setCarView(data);
                    };
                    img.src = ev.target.result;
                };
                reader.readAsDataURL(file);
            };

            const changeDay = (day) => {
                setActiveDay(day);
                setAiOutfit("");
                setTimeout(() => {
                    const el = document.getElementById('itinerary-view');
                    if (el) el.scrollIntoView({ behavior: 'smooth' });
                }, 100);
            };

            return (
                <div className="min-h-screen bg-[#F3F4F6] flex flex-col items-center">
                    {/* Header */}
                    <header className="bg-[#0F172A] text-white py-12 px-4 w-full text-center flex-shrink-0">
                        <h1 className="text-4xl font-black tracking-[0.5em] mb-3 uppercase leading-tight">TAIWAN JOURNEY</h1>
                        <div className="w-20 h-1 bg-[#C9A063] mx-auto mb-4"></div>
                        <p className="text-[11px] font-light tracking-widest text-[#C9A063] uppercase">
                            PREMIUM PRM PRIVATE CHARTER by 原鄉谷
                        </p>
                    </header>

                    {/* Main Container */}
                    <div className="max-w-md w-full px-4 -mt-8 space-y-6 flex-grow pb-12">
                        
                        {/* AI Welcome */}
                        <section className="bg-[#0F172A] p-6 rounded-[2.5rem] shadow-2xl text-white border-b-4 border-[#C9A063]">
                            <div className="flex items-center justify-between mb-4">
                                <div className="flex items-center gap-2 text-[#C9A063] font-bold text-xs uppercase"><i data-lucide="sparkles" class="w-4 h-4"></i> AI CONCIERGE</div>
                                {!aiWelcome && <button onClick={handleAiWelcome} disabled={isWelcomeLoading} className="bg-[#C9A063] text-[10px] px-3 py-1.5 rounded-full font-bold uppercase transition-all shadow-lg active:scale-95">生成歡迎辭</button>}
                            </div>
                            <div className="min-h-[40px] flex items-center">
                                {isWelcomeLoading ? <div className="dot-flashing ml-3 bg-[#C9A063] w-2 h-2 rounded-full ai-pulse"></div> : <p className="text-[11px] text-slate-300 italic leading-relaxed">{aiWelcome || "點擊按鈕獲取專屬 AI 歡迎辭..."}</p>}
                            </div>
                        </section>

                        {/* Card 1: Reservation */}
                        <section className="bg-white rounded-[2.5rem] shadow-2xl overflow-hidden">
                            <div className="p-8 pb-6">
                                <div className="flex justify-between items-center mb-6">
                                    <h2 className="text-xl font-black text-[#0F172A] border-l-4 border-[#C9A063] pl-3">預約資訊</h2>
                                    <span className="bg-[#C9A063]/10 text-[#C9A063] px-3 py-1 rounded-full text-[10px] font-bold uppercase">{reservation.note}</span>
                                </div>
                                <div className="space-y-4">
                                    <div className="flex items-center gap-4 border-b border-slate-50 pb-3">
                                        <div className="bg-slate-50 p-2 rounded-xl text-[#C9A063]"><i data-lucide="users" class="w-5 h-5"></i></div>
                                        <div className="flex-1 font-bold text-lg text-slate-900">{reservation.name}</div>
                                    </div>
                                    <div className="flex items-center gap-4 border-b border-slate-50 pb-3">
                                        <div className="bg-slate-50 p-2 rounded-xl text-[#C9A063]"><i data-lucide="phone" class="w-5 h-5"></i></div>
                                        <div className="flex-1 font-bold text-lg font-georgia text-slate-900">{reservation.phone}</div>
                                    </div>
                                    <div className="flex items-center gap-4">
                                        <div className="bg-slate-50 p-2 rounded-xl text-[#C9A063]"><i data-lucide="info" class="w-5 h-5"></i></div>
                                        <div className="font-bold text-slate-700">人數：{reservation.people}</div>
                                    </div>
                                </div>
                            </div>
                            <div className="bg-slate-50 p-8 grid grid-cols-2 gap-y-6 border-t border-slate-100">
                                <div><p className="text-[10px] text-slate-400 font-bold">航班日期</p><p className="font-georgia font-black">{reservation.date}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold">抵達時間</p><p className="font-georgia font-black">{reservation.time}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold">航班編號</p><p className="font-georgia font-black text-[#C9A063] text-lg">{reservation.flight}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold">航廈</p><p className="font-georgia font-black">{reservation.terminal}</p></div>
                            </div>
                        </section>

                        {/* Card 2: Driver - 修復跑位與加上金色細框 */}
                        <section className="bg-white rounded-[2.5rem] shadow-2xl p-8 text-center relative overflow-hidden">
                            <label className="mb-6 cursor-pointer inline-block group relative">
                                <div className="fixed-avatar-box rounded-full border-2 border-[#C9A063] p-1 shadow-xl bg-white mx-auto overflow-hidden flex items-center justify-center">
                                    {avatar ? <img src={avatar} className="w-full h-full object-cover" /> : <div className="flex flex-col items-center"><i data-lucide="camera" class="w-8 h-8 text-slate-300"></i><p className="text-[8px] text-slate-300">上傳大頭照</p></div>}
                                </div>
                                <input type="file" className="hidden" onChange={(e) => handleImage(e, 'avatar')} accept="image/*" />
                            </label>
                            <div className="space-y-1 mb-8">
                                <p className="text-[15px] font-black text-slate-900 leading-none">司機：{reservation.driver} / 電話：{reservation.driverPhone}</p>
                                <p className="text-[15px] font-black font-georgia leading-none mt-2">車牌：<span className="text-red-600 underline decoration-red-100 font-bold">{reservation.carPlate}</span></p>
                            </div>
                            <label className="cursor-pointer block group">
                                <div className="fixed-car-box border-2 border-[#C9A063] rounded-[2rem] overflow-hidden bg-white flex items-center justify-center relative shadow-inner">
                                    {carView ? <img src={carView} className="max-w-full max-h-full object-contain p-2" /> : <div className="flex flex-col items-center"><i data-lucide="car" class="w-10 h-10 text-slate-300"></i><p className="text-[10px] text-slate-300 font-bold uppercase">上傳汽車外觀</p></div>}
                                </div>
                                <input type="file" className="hidden" onChange={(e) => handleImage(e, 'car')} accept="image/*" />
                            </label>
                            <p className="mt-4 text-[11px] font-georgia font-black text-[#C9A063] tracking-[0.2em] uppercase">Mercedes-Benz Vito Tourer</p>
                        </section>

                        {/* Navigation Tabs */}
                        <nav className="flex justify-between gap-1 p-1 bg-white/95 backdrop-blur rounded-[1.5rem] shadow-lg sticky top-4 z-40 ring-1 ring-slate-100">
                            {[1, 2, 3, 4, 5].map((day) => (
                                <button key={day} onClick={() => changeDay(day)} className={`flex-1 py-3.5 rounded-2xl text-[11px] font-black transition-all ${activeDay === day ? 'bg-[#0F172A] text-white shadow-xl scale-105' : 'text-slate-400 hover:bg-slate-50'}`}>DAY {day}</button>
                            ))}
                        </nav>

                        {/* Itinerary Area */}
                        <div id="itinerary-view" className="space-y-4 pt-2">
                            {/* AI Outfit */}
                            <div className="bg-white p-6 rounded-[2.5rem] shadow-lg border-t-2 border-[#C9A063]">
                                <div className="flex items-center justify-between mb-4 font-black text-xs uppercase text-[#0F172A]">
                                    <div className="flex items-center gap-2"><i data-lucide="sparkles" class="text-[#C9A063] w-4 h-4"></i> AI OUTFIT</div>
                                    <button onClick={handleAiOutfit} disabled={isOutfitLoading} className="bg-slate-50 px-3 py-1 rounded-full font-bold text-[10px] transition-all active:scale-95">分析穿搭</button>
                                </div>
                                <div className="min-h-[20px]">
                                    {isOutfitLoading ? <div className="ai-pulse ml-3 bg-[#C9A063] w-2 h-2 rounded-full"></div> : <p className="text-[11px] text-slate-600 leading-relaxed">{aiOutfit || "點擊按鈕獲取當日天氣穿搭建議..."}</p>}
                                </div>
                            </div>

                            <div className="flex items-center gap-3 px-2">
                                <div className="bg-[#C9A063] p-2 rounded-xl text-white shadow-lg"><i data-lucide="calendar" class="w-5 h-5"></i></div>
                                <h3 className="font-black text-xl text-[#0F172A]">{itineraryData[activeDay-1].date}</h3>
                            </div>

                            {itineraryData[activeDay-1].isPickup && (
                                <div className="bg-blue-50/80 p-5 rounded-[2rem] border-l-4 border-blue-500 text-sm font-bold text-blue-900 shadow-sm leading-relaxed">
                                    司機會在降落時間加 60 分鐘。在入境大廳候客。
                                </div>
                            )}

                            {itineraryData[activeDay-1].start && (
                                <div className="bg-slate-100 p-4 rounded-2xl flex items-center gap-2 mb-2"><i data-lucide="clock" class="text-[#C9A063] w-4 h-4"></i><span className="text-xs font-bold italic text-slate-600">{itineraryData[activeDay-1].start}</span></div>
                            )}

                            <div className="space-y-1">
                                {itineraryData[activeDay-1].points.map((p, i) => (
                                    <div key={i}>
                                        <div className={`bg-white p-6 rounded-[2.5rem] shadow-md relative group transition-all hover:shadow-xl ${p.warning ? 'ring-2 ring-orange-100' : ''}`}>
                                            <div className="flex justify-between items-start">
                                                <div className="flex-1 pr-2">
                                                    <div className="flex items-center gap-2 mb-1.5 font-black text-[#0F172A] text-lg">
                                                        <i data-lucide={p.type==='hotel'?'moon':p.type==='airport'?'plane':'map-pin'} class={p.warning?"text-orange-500":"text-[#C9A063]"}></i>
                                                        {p.name}
                                                    </div>
                                                    <p className={`text-xs ${p.warning?'text-orange-700 font-bold':'text-slate-500'}`}>{p.desc}</p>
                                                    
                                                    {p.type !== 'airport' && (
                                                        <button onClick={() => handleAiFood(p.name)} disabled={foodLoadingPoint === p.name} className="mt-4 flex items-center gap-1.5 text-[10px] font-bold text-[#C9A063] bg-[#C9A063]/10 px-3 py-1.5 rounded-full transition-all active:scale-95 shadow-sm">
                                                            {foodLoadingPoint === p.name ? <i data-lucide="loader-2" class="w-3 h-3 animate-spin"></i> : <i data-lucide="sparkles" class="w-3 h-3"></i>} ✨ AI 推薦美食
                                                        </button>
                                                    )}
                                                </div>
                                                {/* 補回導航與搜尋功能 */}
                                                <div className="flex flex-col gap-2">
                                                    <button onClick={() => window.open(`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(p.name)}`)} className="p-3 bg-[#0F172A] rounded-full text-white shadow-lg active:scale-90 transition-transform"><i data-lucide="map-pin" class="w-5 h-5"></i></button>
                                                    <button onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent(p.name)}`)} className="p-3 bg-slate-50 rounded-full text-slate-500 active:scale-90 transition-transform"><i data-lucide="compass" class="w-5 h-5"></i></button>
                                                </div>
                                            </div>
                                            {p.type === 'hotel' && <div className="mt-4 pt-4 border-t border-dashed text-[10px] text-blue-600 font-black uppercase flex items-center gap-1"><i data-lucide="moon" class="w-3 h-3"></i> Check-in / 休息 / 晚安</div>}
                                        </div>
                                        {p.time && <div className="flex justify-center items-center gap-4 py-1.5 opacity-60"><div className="h-[1px] bg-slate-200 flex-1"></div><i data-lucide="car" class="text-red-500 w-4 h-4"></i><span className="text-[9px] font-black text-slate-400 uppercase">Driving: {p.time} mins</span><div className="h-[1px] bg-slate-200 flex-1"></div></div>}
                                    </div>
                                ))}
                            </div>

                            {/* 今日費用 */}
                            <div className="mt-8 bg-[#0F172A] p-8 rounded-[2.5rem] text-white shadow-2xl relative overflow-hidden group">
                                <div className="absolute right-0 bottom-0 p-4 opacity-10 group-hover:rotate-6 transition-transform"><i data-lucide="car" class="w-24 h-24"></i></div>
                                <p className="text-[10px] text-[#C9A063] font-bold uppercase tracking-[0.3em] mb-2">Service Fare</p>
                                <p className="text-4xl font-georgia font-black">{itineraryData[activeDay-1].fare.toLocaleString()} <span className="text-[11px] text-slate-500 font-bold uppercase tracking-widest">TWD / Day</span></p>
                            </div>
                        </div>

                        {/* 總費用卡片 */}
                        <div className="bg-white p-8 rounded-[2.5rem] shadow-xl border-2 border-[#C9A063]/30 flex justify-between items-center mt-4 transition-transform hover:scale-[1.02]">
                            <div><p className="text-[10px] text-slate-400 font-black uppercase tracking-[0.2em] mb-1">Total Fee</p><p className="text-xs text-slate-500 italic">行程代訂與私人包車總額</p></div>
                            <div className="text-right"><p className="text-3xl font-georgia font-black text-[#0F172A]">{totalFare.toLocaleString()}</p><p className="text-[10px] font-bold text-slate-400 uppercase">TWD TOTAL</p></div>
                        </div>
                    </div>

                    {/* Footer - 色塊滿版、60%高度、單行字 */}
                    <footer className="bg-[#0F172A] text-white py-7 w-full text-center flex-shrink-0 mt-auto">
                        <p className="text-2xl font-black tracking-[0.5em] mb-2 uppercase leading-none">祝 您 旅 途 愉 快</p>
                        <p className="text-[10px] font-light tracking-[0.2em] text-[#C9A063] uppercase leading-none mt-2">
                            PREMIUM PRM PRIVATE CHARTER by 原鄉谷
                        </p>
                    </footer>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
        
        // 初始化 Lucide 圖示
        const initIcons = () => { if (typeof lucide !== 'undefined') lucide.createIcons(); };
        setTimeout(initIcons, 600);
        setInterval(initIcons, 2000);
    </script>
</body>
</html>
