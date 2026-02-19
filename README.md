<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>行政程序期間計算器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { background-color: #f8fafc; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        .card { background: white; border-radius: 16px; box-shadow: 0 10px 25px -5px rgba(0,0,0,0.1); }
        .btn-active { background-color: #1e40af !important; color: white !important; border-color: #1e40af !important; }
    </style>
</head>
<body class="p-4 md:p-10">
    <div class="max-w-md mx-auto">
        <div class="card p-6 border border-slate-200">
            <h1 class="text-2xl font-bold text-slate-800 mb-2">⚖️ 期間計算器</h1>
            <p class="text-slate-500 text-xs mb-6">依行政程序法第 48 條自動判定假日順延</p>
            
            <div class="space-y-5">
                <div>
                    <label class="block text-xs font-bold text-slate-400 mb-2 uppercase">案件性質</label>
                    <div class="grid grid-cols-2 gap-2">
                        <button onclick="setType('normal')" id="btn-normal" class="btn-active py-2 px-3 border rounded-lg text-sm font-medium transition-all">一般案件</button>
                        <button onclick="setType('adverse')" id="btn-adverse" class="py-2 px-3 border border-slate-200 text-slate-600 rounded-lg text-sm font-medium transition-all">不利處分</button>
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-bold text-slate-400 mb-1 uppercase">送達日期 (始日)</label>
                    <input type="date" id="startDate" class="w-full p-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>

                <div class="flex gap-4">
                    <div class="flex-[2]">
                        <label class="block text-xs font-bold text-slate-400 mb-1 uppercase">期間</label>
                        <input type="number" id="periodValue" value="30" class="w-full p-3 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div class="flex-1">
                        <label class="block text-xs font-bold text-slate-400 mb-1 uppercase">單位</label>
                        <select id="periodUnit" class="w-full p-3 rounded-xl border border-slate-200 bg-white outline-none">
                            <option value="day">日</option>
                            <option value="month">月</option>
                            <option value="year">年</option>
                        </select>
                    </div>
                </div>

                <button onclick="calculate()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 rounded-xl shadow-lg transition-transform active:scale-95">
                    執行計算
                </button>
            </div>

            <div id="result-area" class="mt-8 hidden border-t pt-6">
                <div id="warning-box" class="hidden mb-4 p-3 bg-orange-50 border border-orange-100 text-orange-700 text-xs rounded-lg">
                    ⚠️ 查無該年度國定假日資料，改以週六日判定。
                </div>
                <div class="bg-slate-50 p-5 rounded-2xl border border-slate-100">
                    <span id="typeTag" class="inline-block px-2 py-1 rounded bg-blue-100 text-blue-700 text-[10px] font-bold mb-2"></span>
                    <div class="text-sm text-slate-400">屆滿日期 (末日)</div>
                    <div id="finalDate" class="text-3xl font-black text-slate-800"></div>
                    <div id="dayOfWeek" class="text-sm font-bold text-blue-600 mt-1"></div>
                    <p id="calcDetail" class="mt-4 text-xs text-slate-500 leading-relaxed"></p>
                </div>
            </div>
        </div>
    </div>

    <script>
        const HOLIDAY_DATA = {"2018":["1000011000001100000110000011000","0011000001100011111100011001","0011000001100000110000011000000","100111110000011000001100000110","0000110000011000001100000110000","011000001100000111000011000001","1000001100000110000011000001100","0001100000110000011000001100000","110000011000001100000111000011","0000011001001100000110000011000","001100000110000011000001100000","1100000110000011000000100000111"],"2019":["1000110000011000000100000110000","0111111111000001100000010001","1110000011000001100000110000011","000111100000110000011000001100","0001100000110000011000001100000","110000111000001100000110000011","0000011000001100000110000011000","0011000001100000110000011000001","100000110000111000001100000110","0000010001111000001100000110000","011000001100000110000011000001","1000001100000110000011000001100"],"2020":["1001100000110000011000111111100","11000001100000010000011000011","1000001100000110000011000001100","011110000011000001100000110000","0110000011000001100000110000011","000001100000110000001000111100","0001100000110000011000001100000","1100000110000011000001100000110","000011000001100000110000001000","1111000011100000110000011000001","100000110000011000001100000110","0000110000011000001100000110000"],"2021":["1110000011000001100000110000011","0000011001111111000010000011","1000011000001100000110000011000","011110000110000011000001100000","1100000110000011000001100000110","000011000001110000110000011000","0011000001100000110000011000001","1000001100000110000011000001100","000110000001000001111000110000","0110000011100001100000110000011","000001100000110000011000001100","0001100000110000011000001100001"],"2022":["1100000110000011000000100000111","1111110000011000001100000111","0000110000011000001100000110000","011110001100000110000011000001","1000001100000110000011000001100","001110000011000001100000110000","0110000011000001100000110000011","0000011000001100000110000011000","001100001110000011000001100000","1100000111000011000001100000110","000011000001100000110000011000","0011000001100000110000011000001"],"2023":["1100000100000110000111111111100","0000100000110000001000001111","0001100000110000011000000100000","111110011000001100000110000011","0000011000001100000110000011000","001100000110000001000111100000","1100000110000011000001100000110","0000110000011000001100000110000","011000001100000110000001000011","1000001111000110000011000001100","000110000011000001100000110000","0110000011000001100000110000011"],"2024":["1000011000001100000110000011000","00110001111111000100000110010","0110000011000001100000110000011","000111100000110000011000001100","0001100000110000011000001100000","110000011100001100000110000011","0000011000001100000110000011000","0011000001100000110000011000001","100000110000011010001100000110","0000110001011000001100000110000","011000001100000110000011000001","1000001100000110000011000001100"],"2025":["1001100000110000011000001111111","1100000010000011000001100001","1100000110000011000001100000110","001111000001100000110000011000","0011000001100000110000011000011","100000110000011000001100000110","0000110000011000001100000110000","0110000011000001100000110000011","000001100000110000011000001110","0001110001110000011000011100000","110000011000001100000110000011","0000011000001100000110001011000"],"2026":["1011000001100000110000011000001","1000001100000111111111000011","1000001100000110000011000001100","001111000011000001100000110000","1110000011000001100000110000011","000001100000110000111000001100","0001100000110000011000001100000","1100000110000011000001100000110","000011000001100000110000111100","0011000011100000110000011100001","100000110000011000001100000110","0000110000011000001100001110000"]};

        let currentType = 'normal';

        function setType(type) {
            currentType = type;
            document.getElementById('btn-normal').className = type === 'normal' ? 'btn-active py-2 px-3 border rounded-lg text-sm font-medium transition-all' : 'py-2 px-3 border border-slate-200 text-slate-600 rounded-lg text-sm font-medium transition-all';
            document.getElementById('btn-adverse').className = type === 'adverse' ? 'btn-active py-2 px-3 border rounded-lg text-sm font-medium transition-all' : 'py-2 px-3 border border-slate-200 text-slate-600 rounded-lg text-sm font-medium transition-all';
        }

        // 核心假日判斷邏輯
        function isHoliday(date) {
            const year = date.getFullYear();
            const month = date.getMonth(); // 0-11
            const day = date.getDate();    // 1-31

            if (HOLIDAY_DATA[year]) {
                document.getElementById('warning-box').classList.add('hidden');
                const monthStr = HOLIDAY_DATA[year][month];
                // 檢查該日期對應的位元 (1 為假日)
                return monthStr.charAt(day - 1) === "1";
            } else {
                // 沒資料時改用週六日判定
                document.getElementById('warning-box').classList.remove('hidden');
                const dow = date.getDay();
                return (dow === 0 || dow === 6);
            }
        }

        function calculate() {
            const startStr = document.getElementById('startDate').value;
            if (!startStr) return;

            const period = parseInt(document.getElementById('periodValue').value);
            const unit = document.getElementById('periodUnit').value;
            let date = new Date(startStr);
            let detail = "";

            if (currentType === 'normal') {
                document.getElementById('typeTag').innerText = "一般案件";
                detail = "始日不計入。";
                
                // 計算名義末日
                if (unit === 'day') {
                    date.setDate(date.getDate() + period);
                } else if (unit === 'month') {
                    date.setMonth(date.getMonth() + period);
                } else {
                    date.setFullYear(date.getFullYear() + period);
                }

                // 假日順延
                let skipCount = 0;
                while (isHoliday(date)) {
                    date.setDate(date.getDate() + 1);
                    skipCount++;
                }
                if (skipCount > 0) detail += ` 末日適逢假日，順延 ${skipCount} 天。`;
            } else {
                document.getElementById('typeTag').innerText = "不利處分";
                detail = "不利處分：始日計 1 日，假日照計。";
                
                if (unit === 'day') {
                    date.setDate(date.getDate() + (period - 1));
                } else {
                    if (unit === 'month') date.setMonth(date.getMonth() + period);
                    else date.setFullYear(date.getFullYear() + period);
                    date.setDate(date.getDate() - 1);
                }
            }

            const weekNames = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];
            document.getElementById('finalDate').innerText = date.toISOString().split('T')[0];
            document.getElementById('dayOfWeek').innerText = weekNames[date.getDay()];
            document.getElementById('calcDetail').innerText = detail;
            document.getElementById('result-area').classList.remove('hidden');
        }

        // 預設今天
        document.getElementById('startDate').valueAsDate = new Date();
    </script>
</body>
</html>
