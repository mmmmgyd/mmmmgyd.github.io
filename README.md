# mmmmgyd.github.io
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<!-- 分享标题、封面设置 -->
<meta property="og:title" content="每日日历签到打卡">
<meta property="og:description" content="简单好用的个人签到日历">
<title>日历签到打卡</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:微软雅黑}
body{background:#f4f7fa;padding:20px 12px}
.wrap{max-width:540px;margin:0 auto;background:#fff;border-radius:20px;padding:30px 20px;box-shadow:0 2px 14px rgba(0,0,0,0.06)}
h1{text-align:center;color:#222;margin-bottom:24px;font-size:26px}
.info-bar{display:flex;justify-content:space-around;margin-bottom:24px}
.info-item{text-align:center}
.num{font-size:34px;color:#00b96b;font-weight:bold}
.txt{font-size:14px;color:#666;margin-top:4px}
.calendar-box{margin-bottom:24px}
.calendar-head{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px}
.calendar-head button{border:none;background:#eee;padding:6px 14px;border-radius:8px;font-size:16px}
.calendar-title{font-size:20px;font-weight:500}
.week-row{display:grid;grid-template-columns:repeat(7,1fr);gap:6px;margin-bottom:8px}
.week-cell{text-align:center;color:#888;font-size:15px;padding:6px}
.day-row{display:grid;grid-template-columns:repeat(7,1fr);gap:6px;margin-bottom:6px}
.day-cell{aspect-ratio:1/1;display:flex;align-items:center;justify-content:center;border-radius:10px;font-size:16px;background:#f7f7f7}
.day-cell.sign{background:#00b96b;color:#fff}
.day-cell.today{border:2px solid #00b96b}
.day-cell.other{opacity:0.3}
#signBtn{width:100%;height:58px;border:none;border-radius:30px;background:#00b96b;color:#fff;font-size:19px}
#signBtn:disabled{background:#ccc}
.share-tip{margin-top:20px;text-align:center;color:#888;font-size:14px}
</style>
</head>
<body>
<div class="wrap">
<h1>日历签到打卡</h1>
<div class="info-bar">
<div class="info-item">
<div class="num" id="continueNum">0</div>
<div class="txt">连续签到</div>
</div>
<div class="info-item">
<div class="num" id="totalNum">0</div>
<div class="txt">累计签到</div>
</div>
</div>

<div class="calendar-box">
<div class="calendar-head">
<button id="prevMonth">&lt;</button>
<div class="calendar-title" id="calTitle"></div>
<button id="nextMonth">&gt;</button>
</div>
<div class="week-row">
<div class="week-cell">日</div>
<div class="week-cell">一</div>
<div class="week-cell">二</div>
<div class="week-cell">三</div>
<div class="week-cell">四</div>
<div class="week-cell">五</div>
<div class="week-cell">六</div>
</div>
<div id="calendarBody"></div>
</div>

<button id="signBtn">立即签到</button>
<div class="share-tip">复制网页链接即可分享给好友</div>
</div>

<script>
const btn = document.getElementById('signBtn');
const continueNum = document.getElementById('continueNum');
const totalNum = document.getElementById('totalNum');
const calTitle = document.getElementById('calTitle');
const calendarBody = document.getElementById('calendarBody');
const prevBtn = document.getElementById('prevMonth');
const nextBtn = document.getElementById('nextMonth');

let currentViewYear = new Date().getFullYear();
let currentViewMonth = new Date().getMonth();

function formatDate(d){
    const y = d.getFullYear();
    let m = d.getMonth()+1;
    let day = d.getDate();
    m = m<10 ? '0'+m : m;
    day = day<10 ? '0'+day : day;
    return `${y}-${m}-${day}`;
}
function getTodayStr(){
    return formatDate(new Date());
}
function getYesterdayStr(){
    const d = new Date();
    d.setDate(d.getDate()-1);
    return formatDate(d);
}
function getSignData(){
    const cache = localStorage.getItem('signData');
    return cache ? JSON.parse(cache) : {last:'',continue:0,total:0,signList:[]};
}
function saveSignData(data){
    localStorage.setItem('signData', JSON.stringify(data));
}
function renderCalendar(year, month){
    calendarBody.innerHTML = '';
    const firstDay = new Date(year, month, 1);
    const lastDay = new Date(year, month+1, 0);
    const signData = getSignData();
    const signSet = new Set(signData.signList);
    const todayStr = getTodayStr();
    calTitle.innerText = `${year}年${month+1}月`;
    let rowDiv = document.createElement('div');
    rowDiv.className = 'day-row';
    const startWeek = firstDay.getDay();
    for(let i=0;i<startWeek;i++){
        let cell = document.createElement('div');
        cell.className = 'day-cell other';
        rowDiv.appendChild(cell);
    }
    for(let d=1;d<=lastDay.getDate();d++){
        const dateObj = new Date(year, month, d);
        const dateStr = formatDate(dateObj);
        let cell = document.createElement('div');
        cell.className = 'day-cell';
        cell.innerText = d;
        if(signSet.has(dateStr)) cell.classList.add('sign');
        if(dateStr === todayStr) cell.classList.add('today');
        rowDiv.appendChild(cell);
        if((startWeek + d) % 7 === 0){
            calendarBody.appendChild(rowDiv);
            rowDiv = document.createElement('div');
            rowDiv.className = 'day-row';
        }
    }
    if(rowDiv.children.length > 0) calendarBody.appendChild(rowDiv);
}
function refreshStatus(){
    const data = getSignData();
    const today = getTodayStr();
    continueNum.innerText = data.continue;
    totalNum.innerText = data.total;
    if(data.last === today){
        btn.innerText = '今日已签到';
        btn.disabled = true;
    }else{
        btn.innerText = '立即签到';
        btn.disabled = false;
    }
}
function doSign(){
    const today = getTodayStr();
    const yesterday = getYesterdayStr();
    const data = getSignData();
    if(data.last === yesterday){
        data.continue += 1;
    }else{
        data.continue = 1;
    }
    data.total += 1;
    data.last = today;
    data.signList.push(today);
    saveSignData(data);
    alert('签到成功！复制上方链接分享给他人');
    refreshAll();
}
function refreshAll(){
    renderCalendar(currentViewYear, currentViewMonth);
    refreshStatus();
}
prevBtn.onclick = ()=>{
    currentViewMonth--;
    if(currentViewMonth < 0){
        currentViewMonth = 11;
        currentViewYear--;
    }
    renderCalendar(currentViewYear, currentViewMonth);
}
nextBtn.onclick = ()=>{
    currentViewMonth++;
    if(currentViewMonth > 11){
        currentViewMonth = 0;
        currentViewYear++;
    }
    renderCalendar(currentViewYear, currentViewMonth);
}
btn.onclick = doSign;
window.onload = refreshAll;
</script>
</body>
</html>
