const DISCOVERY_DOC = 'https://www.googleapis.com/discovery/v1/apis/drive/v3/rest';
const SCOPES = 'https://www.googleapis.com/auth/drive.file'; // 僅請求存取由應用程式建立或開啟的檔案

let tokenClient;
let gapiInited = false;
let gisInited = false;
let calendar;

// 1. 載入 Google API 核心
function gapiLoaded() { gapi.load('client', async () => { gapiInited = true; maybeInitialize(); }); }
function gisLoaded() { gisInited = true; maybeInitialize(); }

function maybeInitialize() {
  const savedClientId = localStorage.getItem('cal_client_id');
  const savedFileId = localStorage.getItem('cal_file_id');
  
  if (savedClientId && savedFileId) {
    document.getElementById('setup-panel').style.display = 'none';
    if (gapiInited && gisInited) {
      gapi.client.init({ discoveryDocs: [DISCOVERY_DOC] }).then(() => {
        tokenClient = google.accounts.oauth2.initTokenClient({
          client_id: savedClientId,
          scope: SCOPES,
          callback: async (resp) => {
            if (resp.error) return;
            document.getElementById('auth-panel').style.display = 'none';
            document.getElementById('calendar').style.display = 'block';
            await loadCalendarData(savedFileId);
          }
        });
        document.getElementById('auth-panel').style.display = 'block';
      });
    }
  }
}

// 2. 儲存設定
function saveSettings() {
  const clientId = document.getElementById('client-id').value.trim();
  const fileId = document.getElementById('file-id').value.trim();
  if (clientId && fileId) {
    localStorage.setItem('cal_client_id', clientId);
    localStorage.setItem('cal_file_id', fileId);
    location.reload(); // 重新載入以初始化 API
  }
}

// 3. 觸發登入
function handleAuthClick() {
  if (gapi.client.getToken() === null) {
    tokenClient.requestAccessToken({prompt: 'consent'});
  } else {
    tokenClient.requestAccessToken({prompt: ''});
  }
}

// 自動顏色判斷
function getEventColor(title) {
  const t = title.toLowerCase();
  if (t.includes('rp8')) return '#D50000'; // 紅色
  if (t.includes('sukiya') || title.includes('すき家')) return '#F6BF26'; // 橙色
  if (t.includes('上學')) return '#3F51B5'; // 藍色
  if (t.includes('個人')) return '#616161'; // 灰色
  return '#039be5'; // 預設藍色
}

// 4. 從 Drive 讀取資料並渲染日曆
async function loadCalendarData(fileId) {
  try {
    const response = await gapi.client.drive.files.get({
      fileId: fileId,
      alt: 'media' // 指示直接下載檔案內容
    });
    
    const eventsData = response.result || []; // 假設內容是 JSON
    renderCalendar(eventsData, fileId);
  } catch (error) {
    alert("讀取失敗，請確認你是否擁有該檔案的權限。");
    console.error(error);
  }
}

function renderCalendar(eventsData, fileId) {
  const calendarEl = document.getElementById('calendar');
  calendar = new FullCalendar.Calendar(calendarEl, {
    initialView: 'dayGridMonth',
    events: eventsData,
    selectable: true,
    
    // 新增行程
    select: async function(info) {
      const title = prompt('請輸入行程名稱:');
      if (title) {
        calendar.addEvent({ 
          title: title, 
          start: info.startStr, 
          end: info.endStr,
          backgroundColor: getEventColor(title),
          borderColor: getEventColor(title)
        });
        await syncToDrive(fileId);
      }
      calendar.unselect();
    },

    // 刪除行程
    eventClick: async function(info) {
      if (confirm(`確定要刪除行程「${info.event.title}」嗎？`)) {
        info.event.remove();
        await syncToDrive(fileId);
      }
    }
  });
  calendar.render();
}

// 5. 覆寫更新回 Google Drive
async function syncToDrive(fileId) {
  const currentEvents = calendar.getEvents().map(e => ({
    title: e.title, start: e.startStr, end: e.endStr, backgroundColor: e.backgroundColor, borderColor: e.borderColor
  }));

  const token = gapi.client.getToken().access_token;
  
  // Google Drive API 更新檔案內容需要用 PATCH 發送到 upload 端點
  await fetch(`https://www.googleapis.com/upload/drive/v3/files/${fileId}?uploadType=media`, {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(currentEvents)
  });
}
