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
        
        /* 鎖定圖片容器，徹底防止加載時跑位 */
        .fixed-avatar-container { width: 140px !important; height: 140px !important; flex-shrink: 0; background-color: #fff; border: 2px solid #C9A063; overflow: hidden; border-radius: 9999px; }
        .fixed-car-container { width: 100% !important; height: 220px !important; flex-shrink: 0; background-color: #fff; border: 2px solid #C9A063; overflow: hidden; border-radius: 1.5rem; }
        
        .smooth-scroll { scroll-behavior: smooth; }
        .modal-bg { background-color: rgba(15, 23, 42, 0.85); backdrop-filter: blur(4px); }
        @keyframes slideIn { from { transform: translateY(30px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
        .animate-slide-in { animation: slideIn 0.3s ease-out forwards; }
    </style>
</head>
<body class="bg-[#F3F4F6] smooth-scroll">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // --- Gemini API 配置 (API Key 留空，系統會自動處理) ---
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
            // 預設檔名設定 (請確保這些檔案有上傳到 GitHub 同一個資料夾)
            const [avatar, setAvatar] = useState("1000013488-removebg-preview.png");
            const [carView, setCarView] = useState(" 2026年4月30日 .png");

            // AI 互動視窗狀態
            const [modal, setModal] = useState({ show: false, title: "", content: "", loading: false });

            const reservation = {
                name: "陳佳君", phone: "+65 9877 0843", people: "3位大人2小孩", note: "尊榮包車五日方案",
                flight: "TR 896", time: "17:05", date: "2026/03/12", terminal: "第一航廈",
                driver: "張其芳", driverPhone: "0935-089253", carPlate: "TEB-6933"
            };

            const itineraryData = [
                { day: 1, date: "2026年3月12日 星期四", isPickup: true, points: [{ name: "桃園機場", desc: "專屬接機服務：尊榮旅程即將開啟", type: "airport" }, { name: "羅東住宿", desc: "抵達飯店辦理入宿休息", type: "hotel" }], fare: 3000 },
                { day: 2, date: "2026年3月13日 星期五", start: "08:30 出發 (優宿商旅)", points: [{ name: "清水地熱公園", desc: "體驗地熱煮食、泡腳放鬆", time: "50" }, { name: "稻香園", desc: "午餐推薦：在地美味私房料理", time: "30" }, { name: "奇麗灣珍奶博物館", desc: "DIY 珍珠奶茶 (13:00預約)", time: "40" }, { name: "北后寺", desc: "精緻禪意建築與藝術參訪", time: "30" }, { name: "登篙林道", desc: "漫步山林芬多精", time: "45" }, { name: "望龍埤花田村", desc: "偶像劇景點，湖光山色", time: "20" }, { name: "羅東夜市", desc: "宜蘭美食與自由夜生活", type: "hotel" }], fare: 4500 },
                { day: 3, date: "2026年3月14日 星期六", start: "09:00 出發 (優宿商旅)", points: [{ name: "中華路永和豆漿", desc: "傳統中式早餐推薦", time: "40" }, { name: "明池山莊", desc: "【高海拔警示】氣溫較低，請乘客備齊保暖衣物", time: "90", warning: true }, { name: "星寶體驗農場", desc: "三星蔥DIY與小動物互動", time: "50" }, { name: "梅花湖", desc: "環湖騎行或悠閒步道散策", time: "40" }, { name: "礁溪溫泉住宿", desc: "享受放鬆舒壓溫泉", type: "hotel" }], fare: 5000 },
                { day: 4, date: "2026年3月15日 星期日", start: "09:00 出發 (品文旅礁溪)", points: [{ name: "賜福自助餐", desc: "在地早餐選擇", time: "30" }, { name: "蘭陽植物王國", desc: "豐富植被生態景觀", time: "40" }, { name: "百匯窯仔雞", desc: "必吃特色甕窯雞午餐", time: "30" }, { name: "斑比山丘", desc: "與梅花鹿親密互動", time: "40" }, { name: "五峰旗瀑布", desc: "壯麗三層瀑布景觀", time: "45" }, { name: "礁溪夜市", desc: "特色小吃巡禮", type: "hotel" }], fare: 4500 },
                { day: 5, date: "2026年3月16日 星期一", start: "08:30 出發 (品文旅礁溪)", points: [{ name: "清珍早點", desc: "知名老字號早餐，傳統美味", time: "40" }, { name: "龜山島賞鯨", desc: "海上探險：賞鯨豚與環島 (10:00預約)", time: "60" }, { name: "幸福36號海鮮餐廳", desc: "南方澳現撈鮮美海鮮饗宴", time: "50" }, { name: "鼻頭角步道", desc: "海天一色絕美長城步道", time: "70" }, { name: "基隆夜市", desc: "美食巡禮 (預計18:00離開)", time: "40" }, { name: "台北路徒Plus行旅", desc: "行程結束，祝旅途愉快", type: "hotel" }], fare: 5500 }
            ];

            const totalFare = itineraryData.reduce((sum, d) => sum + d.fare, 0);

            // --- ✨ AI 尊榮功能 (Gemini 2.5) ---
            const callAi = async (type, point = "") => {
                setModal({ show: true, title: "AI 尊榮管家處理中...", content: "", loading: true });
                
                let prompt = "";
                let tools = null;

                if (type === 'welcome') {
                    prompt = `你是原鄉谷包車VIP管家。為貴賓 ${reservation.name} 寫一段約 60 字內的精緻歡迎辭。提到張其芳司機。`;
                } else if (type === 'outfit') {
                    const day = itineraryData[activeDay - 1];
                    prompt = `日期是 ${day.date}，地點有 ${day.points.map(p => p.name).join(', ')}。分析宜蘭三月氣候與海拔差異，給予穿搭建議。`;
                } else if (type === 'food') {
                    prompt = `請搜尋並推薦「${point}」景點附近的 2 個最新且高評價的在地美食店家，說明推薦原因。`;
                    tools = [{ "google_search": {} }];
                }

                try {
                    const res = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }], tools })
                    });
                    const text = res.candidates?.[0]?.content?.parts?.[0]?.text || "暫時無法獲取資訊。";
                    setModal({ 
                        show: true, 
                        title: type === 'welcome' ? "✨ 尊榮歡迎辭" : type === 'outfit' ? "✨ 行程穿搭建議" : `✨ ${point} 美食推薦`, 
                        content: text, 
                        loading: false 
                    });
                } catch (e) {
                    setModal({ show: true, title: "服務忙碌", content: "AI 管家目前連線較慢，請稍後再試。", loading: false });
                }
            };

            const changeDay = (day) => {
                setActiveDay(day);
                setTimeout(() => {
                    const el = document.getElementById('day-view');
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

                    {/* Main Content */}
                    <div className="max-w-md w-full px-4 -mt-8 space-y-6 flex-grow pb-12">
                        
                        {/* AI Concierge Summary */}
                        <section className="bg-[#0F172A] p-6 rounded-[2.5rem] shadow-2xl text-white border-b-4 border-[#C9A063]">
                            <div className="flex items-center justify-between">
                                <div className="flex items-center gap-2 text-[#C9A063] font-bold text-xs uppercase"><i data-lucide="sparkles" class="w-4 h-4"></i> AI CONCIERGE</div>
                                <button onClick={() => callAi('welcome')} className="bg-[#C9A063] text-[10px] px-4 py-1.5 rounded-full font-bold uppercase shadow-lg active:scale-95">生成歡迎語</button>
                            </div>
                            <p className="mt-3 text-[10px] text-slate-400 italic">點擊按鈕，由 AI 嚮導為貴賓準備行程序言。</p>
                        </section>

                        {/* Card 1: Reservation */}
                        <section className="bg-white rounded-[2.5rem] shadow-2xl overflow-hidden">
                            <div className="p-8 pb-6">
                                <div className="flex justify-between items-center mb-6 text-slate-900">
                                    <h2 className="text-xl font-black border-l-4 border-[#C9A063] pl-3">預約資訊</h2>
                                    <span className="bg-[#C9A063]/10 text-[#C9A063] px-3 py-1 rounded-full text-[10px] font-bold uppercase">{reservation.note}</span>
                                </div>
                                <div className="space-y-4">
                                    <div className="flex items-center gap-4 border-b border-slate-50 pb-3">
                                        <div className="bg-slate-50 p-2 rounded-xl text-[#C9A063]"><i data-lucide="users" size={18}></i></div>
                                        <div className="flex-1 font-bold text-lg text-slate-900">{reservation.name}</div>
                                    </div>
                                    <div className="flex items-center gap-4 border-b border-slate-50 pb-3">
                                        <div className="bg-slate-50 p-2 rounded-xl text-[#C9A063]"><i data-lucide="phone" size={18}></i></div>
                                        <div className="flex-1 font-bold text-lg font-georgia text-slate-900">{reservation.phone}</div>
                                    </div>
                                    <div className="flex items-center gap-4">
                                        <div className="bg-slate-50 p-2 rounded-xl text-[#C9A063]"><i data-lucide="info" size={18}></i></div>
                                        <div className="font-bold text-slate-700">人數：{reservation.people}</div>
                                    </div>
                                </div>
                            </div>
                            <div className="bg-slate-50 p-8 grid grid-cols-2 gap-y-6 border-t border-slate-100">
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">航班日期</p><p className="font-georgia font-black text-slate-800">{reservation.date}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">抵達時間</p><p className="font-georgia font-black text-slate-800">{reservation.time}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">航班編號</p><p className="font-georgia font-black text-[#C9A063] text-lg">{reservation.flight}</p></div>
                                <div><p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">抵達航廈</p><p className="font-georgia font-black text-slate-800">{reservation.terminal}</p></div>
                            </div>
                        </section>

                        {/* Card 2: Driver - 固定容器 + 金色邊框 */}
                        <section className="bg-white rounded-[2.5rem] shadow-2xl p-8 text-center relative overflow-hidden">
                            <div className="fixed-avatar-container mx-auto shadow-lg flex items-center justify-center mb-6">
                                <img src={avatar} className="w-full h-full object-cover" />
                            </div>
                            <div className="space-y-1 mb-8">
                                <p className="text-[15px] font-black text-slate-900 leading-none">司機：{reservation.driver} / 電話：{reservation.driverPhone}</p>
                                <p className="text-[15px] font-black font-georgia leading-none mt-2">車牌：<span className="text-red-600 underline decoration-red-100 font-bold">{reservation.carPlate}</span></p>
                            </div>
                            <div className="fixed-car-container flex items-center justify-center relative shadow-inner">
                                <img src={carView} className="max-w-full max-h-full object-contain p-2" />
                            </div>
                            <p className="mt-4 text-[11px] font-georgia font-black text-[#C9A063] tracking-[0.2em] uppercase leading-none">Mercedes-Benz Vito Tourer</p>
                        </section>

                        {/* Sticky Navigation */}
                        <nav className="flex justify-between gap-1 p-1 bg-white/95 backdrop-blur rounded-[1.5rem] shadow-lg sticky top-4 z-40 ring-1 ring-slate-100">
                            {[1, 2, 3, 4, 5].map((day) => (
                                <button key={day} onClick={() => changeDay(day)} className={`flex-1 py-3.5 rounded-2xl text-[11px] font-black transition-all ${activeDay === day ? 'bg-[#0F172A] text-white shadow-xl scale-105' : 'text-slate-400'}`}>DAY {day}</button>
                            ))}
                        </nav>

                        {/* Itinerary Area */}
                        <div id="day-view" className="space-y-4 pt-2">
                            {/* AI Outfit */}
                            <div className="bg-white p-6 rounded-[2.5rem] shadow-lg border-t-2 border-[#C9A063]">
                                <div className="flex items-center justify-between mb-4 font-black text-xs uppercase text-[#0F172A]">
                                    <div className="flex items-center gap-2"><i data-lucide="sparkles" class="text-[#C9A063] w-4 h-4"></i> AI OUTFIT TIPS</div>
                                    <button onClick={() => callAi('outfit')} className="bg-slate-50 px-3 py-1 rounded-full font-bold text-[10px] active:scale-95 shadow-sm">分析建議</button>
                                </div>
                                <p className="text-[11px] text-slate-500 italic">針對當日行程與氣候，提供最專業的穿著提醒。</p>
                            </div>

                            <div className="flex items-center gap-3 px-2">
                                <div className="bg-[#C9A063] p-2 rounded-xl text-white shadow-lg"><i data-lucide="calendar" size={18}></i></div>
                                <h3 className="font-black text-xl text-[#0F172A]">{itineraryData[activeDay-1].date}</h3>
                            </div>

                            {itineraryData[activeDay-1].isPickup && (
                                <div className="bg-blue-50/80 p-5 rounded-[2rem] border-l-4 border-blue-500 text-sm font-bold text-blue-900 shadow-sm leading-relaxed flex items-center gap-3">
                                    <i data-lucide="info" class="w-5 h-5 flex-shrink-0"></i>
                                    <span>司機會在降落時間加 60 分鐘。在入境大廳候客。</span>
                                </div>
                            )}

                            {itineraryData[activeDay-1].start && (
                                <div className="bg-slate-100 p-4 rounded-2xl flex items-center gap-2 mb-2"><i data-lucide="clock" class="text-[#C9A063] w-4 h-4"></i><span className="text-xs font-bold italic text-slate-600 font-serif">{itineraryData[activeDay-1].start} 出發</span></div>
                            )}

                            <div className="space-y-1">
                                {itineraryData[activeDay-1].points.map((p, i) => (
                                    <div key={i}>
                                        <div className={`bg-white p-6 rounded-[2.5rem] shadow-md relative group transition-all hover:shadow-xl ${p.warning ? 'ring-2 ring-orange-100' : ''}`}>
                                            <div className="flex justify-between items-start">
                                                <div className="flex-1 pr-2">
                                                    <div className="flex items-center gap-2 mb-1.5 font-black text-[#0F172A] text-lg leading-tight">
                                                        <i data-lucide={p.type==='hotel'?'moon':p.type==='airport'?'plane':'map-pin'} class={p.warning?"text-orange-500":"text-[#C9A063]"}></i>
                                                        {p.name}
                                                    </div>
                                                    <p className="text-xs text-slate-500 leading-relaxed mb-4">{p.desc}</p>
                                                    {p.type !== 'airport' && (
                                                        <button onClick={() => callAi('food', p.name)} className="flex items-center gap-1.5 text-[10px] font-bold text-[#C9A063] bg-[#C9A063]/10 px-3 py-1.5 rounded-full transition-all active:scale-95 shadow-sm">
                                                            <i data-lucide="sparkles" class="w-3 h-3"></i> ✨ AI 推薦美食
                                                        </button>
                                                    )}
                                                </div>
                                                <div className="flex flex-col gap-2">
                                                    <button onClick={() => window.open(`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(p.name)}`)} className="p-3 bg-[#0F172A] rounded-full text-white shadow-lg active:scale-90 transition-transform"><i data-lucide="map-pin" class="w-4 h-4"></i></button>
                                                    <button onClick={() => window.open(`https://www.google.com/search?q=${encodeURIComponent(p.name)}`)} className="p-3 bg-slate-50 rounded-full text-slate-500 active:scale-90 transition-transform"><i data-lucide="compass" class="w-4 h-4"></i></button>
                                                </div>
                                            </div>
                                            {p.type === 'hotel' && <div className="mt-4 pt-4 border-t border-dashed text-[10px] text-blue-600 font-black uppercase flex items-center gap-1 tracking-widest"><i data-lucide="moon" class="w-3 h-3"></i> Check-in / 休息 / 晚安</div>}
                                        </div>
                                        {p.time && <div className="flex justify-center items-center gap-4 py-1.5 opacity-60"><div className="h-[1px] bg-slate-200 flex-1"></div><i data-lucide="car" class="text-red-500 w-4 h-4"></i><span className="text-[9px] font-black text-slate-400 uppercase tracking-tighter font-georgia">Driving: {p.time} mins</span><div className="h-[1px] bg-slate-200 flex-1"></div></div>}
                                    </div>
                                ))}
                            </div>

                            {/* Daily Fare */}
                            <div className="mt-8 bg-[#0F172A] p-8 rounded-[2.5rem] text-white shadow-2xl relative overflow-hidden group">
                                <p className="text-[10px] text-[#C9A063] font-bold uppercase tracking-[0.3em] mb-2">Service Fare</p>
                                <p className="text-4xl font-georgia font-black">{itineraryData[activeDay-1].fare.toLocaleString()} <span className="text-[11px] text-slate-500">TWD / Day</span></p>
                            </div>
                        </div>

                        {/* Total Journey Fee */}
                        <div className="bg-white p-8 rounded-[2.5rem] shadow-xl border-2 border-[#C9A063]/30 flex justify-between items-center mt-4 transition-transform hover:scale-[1.02]">
                            <div><p className="text-[10px] text-slate-400 font-black uppercase mb-1">Total Journey Fee</p><p className="text-xs text-slate-600 font-bold italic">全行程代訂與私人包車總額</p></div>
                            <div className="text-right"><p className="text-3xl font-georgia font-black text-[#0F172A]">{totalFare.toLocaleString()}</p><p className="text-[10px] font-bold text-slate-400 uppercase">TWD TOTAL</p></div>
                        </div>
                    </div>

                    {/* Footer - 滿版色塊、60%高度、單行字 */}
                    <footer className="bg-[#0F172A] text-white py-7 w-full text-center flex-shrink-0 mt-auto">
                        <p className="text-2xl font-black tracking-[0.5em] mb-2 uppercase leading-none">祝 您 旅 途 愉 快</p>
                        <p className="text-[10px] font-light tracking-[0.2em] text-[#C9A063] uppercase leading-none mt-2">
                            PREMIUM PRM PRIVATE CHARTER by 原鄉谷
                        </p>
                    </footer>

                    {/* ✨ AI 互動視窗 (解決 alert 被攔截問題) ✨ */}
                    {modal.show && (
                        <div className="fixed inset-0 z-[100] flex items-center justify-center p-6 modal-bg" onClick={() => setModal({...modal, show: false})}>
                            <div className="bg-white w-full max-w-sm rounded-[2.5rem] shadow-2xl overflow-hidden animate-slide-in" onClick={e => e.stopPropagation()}>
                                <div className="bg-[#0F172A] p-6 text-white flex justify-between items-center border-b border-[#C9A063]/30">
                                    <h3 className="font-black flex items-center gap-2 tracking-widest text-sm uppercase">
                                        <i data-lucide="sparkles" class="text-[#C9A063] w-5 h-5"></i>
                                        {modal.title}
                                    </h3>
                                    <button onClick={() => setModal({...modal, show: false})} className="p-1 rounded-full hover:bg-white/10 transition-colors"><i data-lucide="x" class="w-6 h-6"></i></button>
                                </div>
                                <div className="p-8 max-h-[60vh] overflow-y-auto">
                                    {modal.loading ? (
                                        <div className="flex flex-col items-center gap-4 py-8">
                                            <div className="w-10 h-10 border-4 border-[#C9A063]/20 border-t-[#C9A063] rounded-full animate-spin"></div>
                                            <p className="text-[10px] font-black text-slate-400 uppercase tracking-[0.4em]">AI Searching...</p>
                                        </div>
                                    ) : (
                                        <div className="text-slate-600 leading-relaxed text-sm whitespace-pre-line font-medium italic">
                                            {modal.content}
                                        </div>
                                    )}
                                </div>
                                <div className="p-6 bg-slate-50 text-center">
                                    <button onClick={() => setModal({...modal, show: false})} className="bg-[#0F172A] text-white px-10 py-3 rounded-full font-bold uppercase tracking-widest text-xs shadow-lg active:scale-95 transition-all">關閉建議</button>
                                </div>
                            </div>
                        </div>
                    )}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
        
        const initIcons = () => { if (typeof lucide !== 'undefined') lucide.createIcons(); };
        setTimeout(initIcons, 600);
        setInterval(initIcons, 2000);
    </script>
</body>
</html>
