<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>原鄉谷尊榮導覽 - 智慧行程表</title>
    <!-- 核心函式庫 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; -webkit-tap-highlight-color: transparent; }
        .font-georgia { font-family: Georgia, serif; }
        
        /* 鎖定高度預留空間，徹底防止跑位 */
        .avatar-container { width: 140px; height: 140px; flex-shrink: 0; background-color: #fff; border: 2px solid #C9A063; }
        .car-container { width: 100%; height: 220px; flex-shrink: 0; background-color: #fff; border: 2px solid #C9A063; }
        
        .smooth-scroll { scroll-behavior: smooth; }
        
        /* AI 互動視窗動畫 */
        .modal-overlay { background-color: rgba(15, 23, 42, 0.8); backdrop-filter: blur(4px); }
        @keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
        .animate-slide-up { animation: slideUp 0.3s ease-out forwards; }
    </style>
</head>
<body class="bg-[#F3F4F6] smooth-scroll">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // --- Gemini API 基礎配置 (API Key 會由環境提供，若在 GitHub Pages 則需自行填入) ---
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
            // 預設載入您上傳的照片檔名
            const [avatar, setAvatar] = useState("1000013488-removebg-preview.png");
            const [carView, setCarView] = useState(" 2026年4月30日 .png");
            
            // AI 狀態
            const [aiContent, setAiContent] = useState({ title: "", text: "", type: "" });
            const [isAiLoading, setIsAiLoading] = useState(false);
            const [showModal, setShowModal] = useState(false);

            const reservation = {
                name: "陳佳君", phone: "+65 9877 0843", people: "3位大人2小孩", note: "尊榮包車五日方案",
                flight: "TR 896", time: "17:05", date: "2026/03/12", terminal: "第一航廈",
                driver: "張其芳", driverPhone: "0935-089253", carPlate: "TEB-6933"
            };

            const itinerary = [
                { day: 1, date: "2026年3月12日 星期四", pickup: true, points: [{ name: "桃園機場", desc: "專屬接機服務：尊榮司機準備中", type: "airport" }, { name: "羅東住宿", desc: "抵達飯店辦理入宿休息", type: "hotel" }], fare: 3000 },
                { day: 2, date: "2026年3月13日 星期五", start: "08:30 出發", points: [{ name: "清水地熱公園", desc: "體驗地熱煮食、泡腳放鬆", time: "50" }, { name: "稻香園", desc: "午餐推薦：在地美味私房料理", time: "30" }, { name: "奇麗灣珍奶博物館", desc: "DIY 珍珠奶茶體驗 (13:00預約)", time: "40" }, { name: "北后寺", desc: "精緻禪意建築與藝術參訪", time: "30" }, { name: "登篙林道", desc: "漫步山林芬多精", time: "45" }, { name: "望龍埤花田村", desc: "偶像劇景點，湖光山色", time: "20" }, { name: "羅東夜市", desc: "宜蘭美食與在地生活體驗", type: "hotel" }], fare: 4500 },
                { day: 3, date: "2026年3月14日 星期六", start: "09:00 出發", points: [{ name: "中華路永和豆漿", desc: "傳統中式早餐推薦", time: "40" }, { name: "明池山莊", desc: "【高海拔警示】氣溫較低，請務必備齊保暖衣物", time: "90", warning: true }, { name: "星寶體驗農場", desc: "三星蔥DIY與小動物互動", time: "50" }, { name: "梅花湖", desc: "環湖騎行或悠閒步道散策", time: "40" }, { name: "礁溪溫泉住宿", desc: "飯店休息、享受舒壓溫泉", type: "hotel" }], fare: 5000 },
                { day: 4, date: "2026年3月15日 星期日", start: "09:00 出發", points: [{ name: "賜福自助餐", desc: "在地早餐多樣化選擇", time: "30" }, { name: "蘭陽植物王國", desc: "豐富多樣的植物生態景觀", time: "40" }, { name: "百匯窯仔雞", desc: "礁溪必吃特色甕窯雞午餐", time: "30" }, { name: "斑比山丘", desc: "與梅花鹿親密互動體驗", time: "40" }, { name: "五峰旗瀑布", desc: "壯麗三層瀑布景觀步道", time: "45" }, { name: "礁溪夜市", desc: "宜蘭特色小吃巡禮與自由晚餐", type: "hotel" }], fare: 4500 },
                { day: 5, date: "2026年3月16日 星期一", start: "08:30 出發", points: [{ name: "清珍早點", desc: "知名老字號早餐，傳統美味", time: "40" }, { name: "龜山島賞鯨", desc: "海上探險：賞鯨豚與環島行程 (10:00預約)", time: "60" }, { name: "幸福36號海鮮餐廳", desc: "南方澳現撈鮮甜海鮮饗宴", time: "50" }, { name: "鼻頭角步道", desc: "海天一色絕美長城步道", time: "70" }, { name: "基隆夜市", desc: "基隆廟口美食巡禮 (預計18:00離開)", time: "40" }, { name: "台北路徒Plus行旅", desc: "平安抵達，祝旅途愉快", type: "hotel" }], fare: 5500 }
            ];

            const totalFare = itinerary.reduce((sum, d) => sum + d.fare, 0);

            // --- ✨ AI 智能互動功能 ---
            const callAi = async (type, payload = "") => {
                setIsAiLoading(true);
                setAiContent({ title: "AI 尊榮管家處理中...", text: "", type });
                setShowModal(true);

                let prompt = "";
                let useSearch = false;

                if (type === 'welcome') {
                    prompt = `你是原鄉谷包車VIP導遊。為貴賓 ${reservation.name} 寫一段 60 字內的熱情、優雅且具質感的歡迎辭。提及您張其芳司機。`;
                } else if (type === 'outfit') {
                    const day = itinerary[activeDay - 1];
                    prompt = `日期是 ${day.date}，地點包括 ${day.points.map(p => p.name).join(', ')}。分析宜蘭三月氣候與海拔差異，給予具體的衣著穿搭與天氣提醒。`;
                } else if (type === 'food') {
                    prompt = `請搜尋並推薦「${payload}」景點附近的 2 個最新且高評價的在地美食店家。請包含店名、特色理由。`;
                    useSearch = true;
                }

                try {
                    const body = {
                        contents: [{ parts: [{ text: prompt }] }]
                    };
                    if (useSearch) {
                        body.tools = [{ "google_search": {} }];
                    }

                    const res = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(body)
                    });

                    const text = res.candidates?.[0]?.content?.parts?.[0]?.text || "暫時無法獲取資訊。";
                    setAiContent({ 
                        title: type === 'welcome' ? "✨ 尊榮歡迎辭" : type === 'outfit' ? "✨ 行程穿搭建議" : `✨ 「${payload}」美食推薦`, 
                        text, 
                        type 
                    });
                } catch (e) {
                    setAiContent({ title: "服務忙碌", text: "目前 AI 管家連線較慢，請稍後再試，或直接詢問您的張司機。", type: "error" });
                } finally {
                    setIsAiLoading(false);
                }
            };

            const handleImage = (e, target) => {
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
                        const dataUrl = cvs.toDataURL('image/png');
                        if (target === 'avatar') setAvatar(dataUrl);
                        else setCarView(dataUrl);
                    };
                    img.src = ev.target.result;
                };
                reader.readAsDataURL(file);
            };

            const changeDay = (day) => {
                setActiveDay(day);
                setTimeout(() => {
                    const el = document.getElementById('itinerary-start');
                    if (el) el.scrollIntoView({ behavior: 'smooth' });
                }, 100);
            };

            return (
                <div className="min-h-screen bg-[#F3F4F6] flex flex-col items-center">
                    {/* Header */}
                    <header className="bg-[#0F172A] text-white py-12 px-4 w-full text-center flex-shrink-0">
                        <h1 className="text-4xl font-black tracking-[0.4em] mb-3 uppercase leading-tight">TAIWAN JOURNEY</h1>
                        <div className="w-16 h-1 bg-[#C9A063] mx-auto mb-4"></div>
                        <p className="text-[11px] font-light tracking-widest text-[#C9A063] uppercase">
                            PREMIUM PRM PRIVATE CHARTER by 原鄉谷
                        </p>
                    </header>

                    {/* Main Content Container */}
                    <div className="max-w-md w-full px-4 -mt-8 space-y-6 flex-grow pb-12">
                        
                        {/* AI Summary Section */}
                        <section className="bg-[#0F172A] p-6 rounded-[2.5rem] shadow-2xl text-white border-b-4 border-[#C9A063]">
                            <div className="flex items-center justify-between mb-4">
                                <div className="flex items-center gap-2 text-[#C9A063] font-bold text-xs uppercase"><i data-lucide="sparkles" class="w-4 h-4"></i> AI CONCIERGE</div>
                                <button onClick={() => callAi('welcome')} className="bg-[#C9A063] text-[10px] px-4 py-1.5 rounded-full font-bold uppercase shadow-lg active:scale-95">尊榮歡迎語</button>
                            </div>
                            <p className="text-[11px] text-slate-400 italic">由 Gemini 智慧驅動，為貴賓提供即時行程與美食建議。</p>
                        </section>

                        {/* Card 1: Reservation */}
                        <section className="bg-white rounded-[2.5rem] shadow-2xl overflow-hidden">
                            <div className="p-8 pb-6">
                                <div className="flex justify-between items-center mb-6">
                                    <h2 className="text-xl font-black text-[#0F172A] border-l-4 border-[#C9A063] pl-3">預約詳情</h2>
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
                            <div className="bg-slate-50 p-8 grid grid-cols-2 gap-y-6 border-t border-slate-100 text-slate-800">
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">航班日期</p><p className="font-georgia font-black">{reservation.date}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">抵達時間</p><p className="font-georgia font-black">{reservation.time}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">航班編號</p><p className="font-georgia font-black text-[#C9A063] text-lg">{reservation.flight}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">抵達航廈</p><p className="font-georgia font-black">{reservation.terminal}</p></div>
                            </div>
                        </section>

                        {/* Card 2: Driver - 固定高度，絕對防跑版 */}
                        <section className="bg-white rounded-[2.5rem] shadow-2xl p-8 text-center relative overflow-hidden">
                            <label className="mb-6 cursor-pointer inline-block group relative">
                                <div className="avatar-container rounded-full shadow-xl mx-auto flex items-center justify-center overflow-hidden">
                                    {avatar ? <img src={avatar} className="w-full h-full object-cover" onError={(e) => e.target.style.display='none'} /> : <i data-lucide="camera" class="w-8 h-8 text-slate-300"></i>}
                                </div>
                                <input type="file" className="hidden" onChange={(e) => handleImage(e, 'avatar')} accept="image/*" />
                            </label>
                            <div className="space-y-1 mb-8">
                                <p className="text-[15px] font-black text-slate-900 leading-none">司機：{reservation.driver} / 電話：{reservation.driverPhone}</p>
                                <p className="text-[15px] font-black font-georgia leading-none mt-2">車牌：<span className="text-red-600 underline decoration-red-100 font-bold">{reservation.carPlate}</span></p>
                            </div>
                            <label className="cursor-pointer block group">
                                <div className="car-container rounded-[2rem] flex items-center justify-center relative shadow-inner">
                                    {carView ? <img src={carView} className="max-w-full max-h-full object-contain p-2" onError={(e) => e.target.style.display='none'} /> : <div className="flex flex-col items-center"><i data-lucide="car" class="w-10 h-10 text-slate-300"></i><p className="text-[10px] text-slate-300 font-bold mt-1">上傳車輛照片</p></div>}
                                </div>
                                <input type="file" className="hidden" onChange={(e) => handleImage(e, 'car')} accept="image/*" />
                            </label>
                            <p className="mt-4 text-[11px] font-georgia font-black text-[#C9A063] tracking-[0.2em] uppercase leading-none">Mercedes-Benz Vito Tourer</p>
                        </section>

                        {/* Sticky Navigation */}
                        <nav className="flex justify-between gap-1 p-1 bg-white/95 backdrop-blur rounded-[1.5rem] shadow-lg sticky top-4 z-40 ring-1 ring-slate-100">
                            {[1, 2, 3, 4, 5].map((day) => (
                                <button key={day} onClick={() => changeDay(day)} className={`flex-1 py-3.5 rounded-2xl text-[11px] font-black transition-all ${activeDay === day ? 'bg-[#0F172A] text-white shadow-xl scale-105' : 'text-slate-400'}`}>DAY {day}</button>
                            ))}
                        </nav>

                        {/* Itinerary Area */}
                        <div id="itinerary-start" className="space-y-4 pt-2">
                            {/* AI Outfit */}
                            <div className="bg-white p-6 rounded-[2.5rem] shadow-lg border-t-2 border-[#C9A063]">
                                <div className="flex items-center justify-between mb-4 font-black text-xs uppercase text-[#0F172A]">
                                    <div className="flex items-center gap-2"><i data-lucide="sparkles" class="text-[#C9A063] w-4 h-4"></i> AI OUTFIT TIPS</div>
                                    <button onClick={() => callAi('outfit')} className="bg-slate-50 px-3 py-1 rounded-full font-bold text-[10px] active:scale-95 shadow-sm">分析穿搭</button>
                                </div>
                                <p className="text-[11px] text-slate-500 italic">點擊按鈕獲取由 AI 根據當日氣候與海拔生成的專屬穿搭指南。</p>
                            </div>

                            <div className="flex items-center gap-3 px-2">
                                <div className="bg-[#C9A063] p-2 rounded-xl text-white shadow-lg shadow-[#C9A063]/30"><i data-lucide="calendar" size={18}></i></div>
                                <h3 className="font-black text-xl text-[#0F172A]">{itinerary[activeDay-1].date}</h3>
                            </div>

                            {itinerary[activeDay-1].pickup && (
                                <div className="bg-blue-50/80 p-5 rounded-[2rem] border-l-4 border-blue-500 text-sm font-bold text-blue-900 shadow-sm leading-relaxed flex items-center gap-3">
                                    <i data-lucide="info" class="w-5 h-5 flex-shrink-0"></i>
                                    <span>司機會在降落時間加 60 分鐘。在入境大廳候客。</span>
                                </div>
                            )}

                            {itinerary[activeDay-1].start && (
                                <div className="bg-slate-100 p-4 rounded-2xl flex items-center gap-2 mb-2"><i data-lucide="clock" class="text-[#C9A063] w-4 h-4"></i><span className="text-xs font-bold italic text-slate-600">{itinerary[activeDay-1].start}</span></div>
                            )}

                            <div className="space-y-1">
                                {itinerary[activeDay-1].points.map((p, i) => (
                                    <div key={i}>
                                        <div className={`bg-white p-6 rounded-[2.5rem] shadow-md relative group transition-all hover:shadow-xl ${p.warning ? 'ring-2 ring-orange-100' : ''}`}>
                                            <div className="flex justify-between items-start">
                                                <div className="flex-1 pr-2">
                                                    <div className="flex items-center gap-2 mb-1.5 font-black text-[#0F172A] text-lg">
                                                        <i data-lucide={p.type==='hotel'?'moon':p.type==='airport'?'plane':'map-pin'} class={p.warning?"text-orange-500":"text-[#C9A063]"}></i>
                                                        {p.name}
                                                    </div>
                                                    <p className="text-xs text-slate-500 leading-relaxed mb-4">{p.desc}</p>
                                                    <button onClick={() => callAi('food', p.name)} className="flex items-center gap-1.5 text-[10px] font-bold text-[#C9A063] bg-[#C9A063]/10 px-3 py-1.5 rounded-full transition-all active:scale-95 shadow-sm">
                                                        <i data-lucide="sparkles" class="w-3 h-3"></i> ✨ AI 推薦美食
                                                    </button>
                                                </div>
                                                <div className="flex flex-col gap-2">
                                                    <button onClick={() => window.open(`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(p.name)}`)} className="p-3 bg-[#0F172A] rounded-full text-white shadow-lg active:scale-90 transition-transform"><i data-lucide="map-pin" class="w-4 h-4"></i></button>
                                                    <button onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent(p.name)}`)} className="p-3 bg-slate-50 rounded-full text-slate-500 active:scale-90 transition-transform"><i data-lucide="compass" class="w-4 h-4"></i></button>
                                                </div>
                                            </div>
                                            {p.type === 'hotel' && <div className="mt-4 pt-4 border-t border-dashed text-[10px] text-blue-600 font-black uppercase flex items-center gap-1 tracking-widest"><i data-lucide="moon" class="w-3 h-3"></i> Check-in / 休息 / 晚安</div>}
                                        </div>
                                        {p.time && <div className="flex justify-center items-center gap-4 py-1.5 opacity-60"><div className="h-[1px] bg-slate-200 flex-1"></div><i data-lucide="car" class="text-red-500 w-4 h-4"></i><span className="text-[9px] font-black text-slate-400 uppercase">Driving: {p.time} mins</span><div className="h-[1px] bg-slate-200 flex-1"></div></div>}
                                    </div>
                                ))}
                            </div>

                            {/* Daily Fare */}
                            <div className="mt-8 bg-[#0F172A] p-8 rounded-[2.5rem] text-white shadow-2xl relative overflow-hidden group">
                                <div className="absolute right-0 bottom-0 p-4 opacity-10 group-hover:rotate-6 transition-transform duration-700"><i data-lucide="car" class="w-24 h-24"></i></div>
                                <p className="text-[10px] text-[#C9A063] font-bold uppercase tracking-[0.3em] mb-2">Service Fare</p>
                                <p className="text-4xl font-georgia font-black">{itinerary[activeDay-1].fare.toLocaleString()} <span className="text-[11px] text-slate-500 font-bold uppercase tracking-widest">TWD / Day</span></p>
                            </div>
                        </div>

                        {/* Total Cost Card */}
                        <div className="bg-white p-8 rounded-[2.5rem] shadow-xl border-2 border-[#C9A063]/30 flex justify-between items-center mt-4 transition-transform hover:scale-[1.02]">
                            <div><p className="text-[10px] text-slate-400 font-black uppercase mb-1 tracking-widest">Total Journey Fee</p><p className="text-xs font-bold text-slate-800 italic">全行程代訂與私人包車總額</p></div>
                            <div className="text-right"><p className="text-3xl font-georgia font-black text-[#0F172A]">{totalFare.toLocaleString()}</p><p className="text-[10px] font-bold text-slate-400 uppercase">TWD TOTAL</p></div>
                        </div>
                    </div>

                    {/* Footer - 滿版色塊、60%高度、單行字 */}
                    <footer className="bg-[#0F172A] text-white py-7 w-full text-center flex-shrink-0 mt-auto shadow-2xl">
                        <p className="text-2xl font-black tracking-[0.5em] mb-2 uppercase leading-none">祝 您 旅 途 愉 快</p>
                        <p className="text-[10px] font-light tracking-[0.2em] text-[#C9A063] uppercase leading-none mt-2">
                            PREMIUM PRM PRIVATE CHARTER by 原鄉谷
                        </p>
                    </footer>

                    {/* ✨ AI 尊榮互動視窗 ✨ */}
                    {showModal && (
                        <div className="fixed inset-0 z-[100] flex items-center justify-center p-6 modal-overlay" onClick={() => setShowModal(false)}>
                            <div className="bg-white w-full max-w-sm rounded-[2.5rem] shadow-2xl overflow-hidden animate-slide-up" onClick={e => e.stopPropagation()}>
                                <div className="bg-[#0F172A] p-6 text-white flex justify-between items-center">
                                    <h3 className="font-black flex items-center gap-2 tracking-widest">
                                        <i data-lucide="sparkles" class="text-[#C9A063] w-5 h-5"></i>
                                        {aiContent.title}
                                    </h3>
                                    <button onClick={() => setShowModal(false)}><i data-lucide="x" class="w-6 h-6"></i></button>
                                </div>
                                <div className="p-8 max-h-[60vh] overflow-y-auto">
                                    {isAiLoading ? (
                                        <div className="flex flex-col items-center gap-4 py-8">
                                            <div className="w-12 h-12 border-4 border-[#C9A063]/20 border-t-[#C9A063] rounded-full animate-spin"></div>
                                            <p className="text-sm font-bold text-slate-400 animate-pulse uppercase tracking-[0.3em]">AI Searching...</p>
                                        </div>
                                    ) : (
                                        <div className="text-slate-600 leading-relaxed text-sm whitespace-pre-line font-medium">
                                            {aiContent.text}
                                        </div>
                                    )}
                                </div>
                                <div className="p-6 bg-slate-50 text-center">
                                    <button onClick={() => setShowModal(false)} className="bg-[#0F172A] text-white px-10 py-3 rounded-full font-bold uppercase tracking-widest text-xs shadow-lg active:scale-95 transition-all">關閉視窗</button>
                                </div>
                            </div>
                        </div>
                    )}
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
