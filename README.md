// =================================================================
// Part 1: 設定區 (Configuration)
// 這裡將集中管理所有機器人的設定、訊息、規則與按鈕
// =================================================================
const CONFIG = {
  // --- 機器人訊息設定 ---
  MESSAGES: {
    // --- 通用訊息 ---
    GENERIC_ERROR: "查詢時發生了一點問題，請稍後再試或直接聯絡櫃檯，謝謝！",
    CONTACT_HUMAN_PROMPT: "好的，我已經立即通知我們的櫃檯人員！\n請您直接在這裡輸入您的問題，他們会盡快接手回覆您，請稍候片刻。",
    FINAL_FALLBACK: [
        "不好意思，我不太明白您的問題 😅",
        "不過，您可以直接點擊下方的「主選單」按鈕，或輸入「主選單」來查看我們的主要服務喔！",
    ].join("\n"),

    // --- 價格查詢相關 ---
    PRICE_INPUT_PROMPT: "請告訴我您想查詢的「入住日期」與「房型」喔！\n例如：「9/10-9/12 景觀雙人房」",
    ASK_FOR_DATE: (roomType) => `想查詢哪一天的 ${roomType} 價格呢？\n可以說「今天」或是一個日期區間（例如 9/10-9/12）。`,
    ASK_FOR_ROOM_TYPE: "想查詢哪一種房型呢？點擊下方按鈕或直接輸入都可以喔！",
    DATE_ONLY_PROMPT: "您好，請問是想查詢這個日期的房價嗎？請告訴我您想查詢的房型。",
    AVAILABILITY_DISCLAIMER: "\n\n提醒您：此為系統自動報價，實際空房狀況仍需由專人為您確認喔！",
    PRICE_NOT_FOUND: (date, roomType) => `抱歉，目前查不到 ${date} ${roomType} 的價格，可能是當日已無空房或日期格式有誤，建議直接向櫃檯洽詢喔！`,
    NEXT_YEAR_QUERY: (dateString) => `您好，您查詢的日期 (${dateString}) 為下個年度，我們尚未開放明年的訂房與報價喔，感謝您的詢問！`,
    SPECIAL_DAY_PRICE: (specialDate) => `您查詢的日期（${specialDate}）適逢特殊節日，房價請直接洽詢櫃檯人員喔～`,

    // --- 設施服務相關 ---
    WIFI_INFO: "我們有提供免費 Wi‑Fi，帳號是 Happyinn，密碼是 22232033，歡迎使用喔！",
    LUGGAGE_INFO: "不論是提早抵達還是退房後，我們都可以免費寄放行李，但記得退房後寄放的話要在當天取回喔！",
    PARKING_RULES: `停車補助規則如下：\n★ 限房費每晚 1150 元以上（背包房不適用）\n★ 每房每日補助一台車，最高 100 元\n★ 需出示紙本停車單據（電子發票無法補助）\n★ 限距離本館 1 公里內的一般費率停車場或停車格喔！`,
    
    // --- 在地嚮導 & 交通 ---
    FOOD_NOT_FOUND: "目前找不到附近美食資訊，請稍後再試～",
    TRANSPORT_ROUTING_PROMPT: "好的，請告訴我您的出發地點，例如：「從逢甲夜市到快樂腳旅棧」。",
    TRANSPORT_ERROR_FALLBACK: (origin, hotelAddr) => {
        const url = `https://www.google.com/maps/dir/?api=1&origin=${encodeURIComponent(origin)}&destination=${encodeURIComponent(hotelAddr)}`;
        return `抱歉，自動路線規劃暫時無法使用。\n\n建議您直接使用 Google Maps 導航至「${hotelAddr}」\n\n點此直接開啟導航：\n${url}`;
    },
  },

  // --- 按鈕 payload 設定 (讓程式碼更好讀) ---
// ================== 【已更新】的 ACTIONS 區塊 ==================
ACTIONS: {
  // --- 主選單 ---
  MENU_PRICE: "查房價／問訂房",
  MENU_FACILITY: "設施／服務",
  MENU_FOOD: "在地美食推薦",
  MENU_TRANSPORT: "交通指引",
  MENU_CONTACT_HUMAN: "聯絡真人客服",

  // --- 子選單動作 ---
  PRICE_ASK: "action:ask_price",
  PRICE_SHOW_ROOMS: "action:show_room_types",
  FACILITY_WIFI: "action:show_wifi",
  FACILITY_PARKING: "action:show_parking",
  FACILITY_LUGGAGE: "action:show_luggage",
  GUIDE_FOOD: "action:show_food",
  GUIDE_SIGHTS: "action:show_sights",
  GUIDE_CAFES: "action:show_cafes",
  TRANSPORT_FROM_TRA: "action:from_tra",
  TRANSPORT_FROM_HSR: "action:from_hsr",
  TRANSPORT_BY_CAR: "action:by_car",
  TRANSPORT_TRA_BUS: "action:tra_by_bus",
  TRANSPORT_TRA_WALK: "action:tra_by_walk",
  CONTACT_HUMAN: "action:contact_human",

  // --- 停車流程動作 ---
  NAVIGATE_PARKING: "action:navigate_parking",
  SHOW_PARKING_RULES: "action:show_parking_rules",

  // ✨✨✨ 新增：房型卡片動作 ✨✨✨
  ROOM_INTRO_PREFIX: "action:intro_", // 介紹房型的前綴
  ROOM_PRICE_TODAY_PREFIX: "action:price_today_", // 查當日價的前綴
  ROOM_PRICE_DATE_PREFIX: "action:price_date_", // 查指定日期價的前綴
},

  // --- 正規表示式 (Regex) 意圖判斷區 ---
// ================== 【已補全】的 INTENTS 區塊 ==================
INTENTS: {
  // 將判斷拆分成更小的單位，方便組合
  hasDate: /(今天|今日|\d{1,2}[\/\\-月]\d{1,2})/,
  hasRoomType: /(悠活|家庭|四人|雙床|景觀|雙人|背包|房型)/i,
  hasPriceWord: /(價|價格|費用|房價|多少)/i,

  isBreakfastPhotoQuery: /(早餐).*?(照片|圖片|相片)/i,
  isTransportationQuery: /(怎麼去|怎麼到|如何到|怎樣到|路線|走路|步行|開車|騎車|搭車|公車|轉乘|到(快樂腳旅棧|中華路一段185號))/i,
  isFoodQuery: /(附近|周邊|周遭).*?(美食|餐廳|小吃|酒吧|吃的|好吃|夜市)|美食推薦|吃什麼|推薦美食|要吃什麼/i,
  isParkingQuery: /(停車|停車位)/i,
  isPhotoQuery: /(照片|圖片|相片)/i,

  // 檢查邏輯更嚴謹
  isDateOnlyQuery: (msg) => {
    const hasDate = /(\d{1,2}[\/\\-月]\d{1,2})/.test(msg);
    const hasRoomType = /(悠活|家庭|四人|雙床|景觀|雙人|背包)/i.test(msg);
    const hasPriceWord = /(價|價格|費用|房價|多少)/i.test(msg);
    return hasDate && !hasRoomType && !hasPriceWord;
  },
  isPriceQuery: (msg) => {
    const hasDate = /(今天|今日|\d{1,2}[\/\\-月]\d{1,2})/.test(msg);
    const hasRoomType = /(悠活|家庭|四人|雙床|景觀|雙人|背包)/i.test(msg);
    const hasPriceWord = /(價|價格|費用|房價|多少)/i.test(msg);
    return hasPriceWord || (hasDate && hasRoomType);
  },
  // ✨✨✨ 這一行就是之前遺失的設定 ✨✨✨
  isTodayQuery: /(今天|今日)/i,
},
  
// ================== 【已修正】的 QUICK_REPLIES 區塊 ==================

  // --- 靜態回覆 (QUICK_REPLIES) ---
  QUICK_REPLIES: [
    { pattern: /(Wi[-]?Fi|wifi|無線網路)/i, reply: "我們有提供免費 Wi‑Fi，帳號是 Happyinn，密碼是 22232033，歡迎使用喔！" },
    { pattern: /(提早到|提早入住|提前入住)/i, reply: "我們入住時間是下午三點喔，如果您提早抵達，可以先到交誼廳稍作休息哦～" },
    { pattern: /(寄放|行李|退房後)/i, reply: "不論是提早抵達還是退房後，我們都可以免費寄放行李，但記得退房後寄放的話要在當天取回喔！" },
    { pattern: /(櫃台|服務時間|check in|check-in|checkin)/i, reply: "我們櫃台是 24 小時全年無休的，有任何問題隨時都可以找我們喔！" },
    { pattern: /(刷牙|牙刷|拖鞋|一次性|毛巾|浴巾|沐浴|洗髮|刮鬍刀|梳子|浴帽|備品)/i, reply: "從 114 年起我們不再主動提供牙刷、刮鬍刀、拖鞋等一次性備品喔～如果需要的話可以到櫃檯購買。\n不過洗沐浴乳、洗手乳、毛巾浴巾都還是有提供的！\n p.s.背包房需自備毛巾浴巾" },
    { pattern: /(訂房|預訂|預定|空房|還有房間)/i, reply: "訂房的部分要由櫃台人員協助喔～請您稍等一下，會有專人回覆您！" },
    { 
      pattern: /(附早餐|有早餐|看.*?菜單|想看菜單|早餐菜單|菜單內容)/i,
      reply: "我們旅棧本身沒有附早餐，但有提供代訂「晨家廚房」的服務哦～一份大約 $140～165，想吃熱呼呼的早餐只要前一天告知我們就好 😊\n\n（想看圖片的話可以輸入：早餐照片）" 
    }
  ],
}
// =================================================================
// Part 2: Worker 主程式入口
// 這是程式的起點，它會接收 Line 的請求並交給 handleEvent 處理
// =================================================================
export default {
  async fetch(request, env, ctx) {
    if (request.method === "GET") {
        return new Response("OK", { status: 200 });
    }
    if (request.method !== "POST") {
        return new Response("只接受 POST 請求", { status: 405 });
    }

    let body;
    try {
      body = await request.json();
    } catch (e) {
      console.error("JSON 解析錯誤:", e);
      return new Response("JSON 格式錯誤", { status: 400 });
    }
    
    const event = body.events?.[0];
    const message = event?.message?.text || "";
    const replyToken = event?.replyToken;
    
    // 如果沒有訊息或 replyToken，直接回覆 OK，不繼續執行
    if (!message || !replyToken) {
        return new Response("OK", { status: 200 });
    }
    
    console.log(`🙋 User said: ${message}`);

    // 使用 waitUntil 確保 handleEvent 即使在回覆後也能執行完成
    ctx.waitUntil(handleEvent({ message, replyToken, env }));
    
    // 立即回覆 Line 平台，避免逾時
    return new Response("OK", { status: 200 });
  },
};

// Part 3: 意圖路由器 (Intent Router)
// handleEvent 是個乾淨的「總機」，負責轉接給對應的處理函式
// =================================================================
async function handleEvent({ message, replyToken, env }) {
  const context = { message, replyToken, env };

  // --- 優先處理「固定指令」和「按鈕 Payload」 ---
  if (message.startsWith('action:')) return await handleAction(context);
  if (message === '主選單') return await handleMainMenu(context);

  // --- 處理來自「圖文選單」的點擊 ---
  switch (message) {
    case CONFIG.ACTIONS.MENU_PRICE: return await handlePriceInquiryMenu(context);
    case CONFIG.ACTIONS.MENU_FACILITY: return await handleFacilityMenu(context);
    case CONFIG.ACTIONS.MENU_FOOD: return await handleLocalGuideMenu(context);
    case CONFIG.ACTIONS.MENU_TRANSPORT: return await handleTransportationMenu(context);
    case CONFIG.ACTIONS.MENU_CONTACT_HUMAN: return await handleContactHuman(context);
  }

  // --- 自然語言意圖判斷 ---
  const { INTENTS } = CONFIG;
  const hasDate = INTENTS.hasDate.test(message);
  const hasRoomType = INTENTS.hasRoomType.test(message);
  const hasPriceWord = INTENTS.hasPriceWord.test(message);

  // --- 高優先級判斷 (會產生互動或按鈕的) ---
  if (INTENTS.isParkingQuery.test(message)) return await handleParking(context);
  if (INTENTS.isBreakfastPhotoQuery.test(message)) return await handleBreakfastPhoto(context);
  if (INTENTS.isTransportationQuery.test(message)) return await handleTransportationRouting(context);
  if (INTENTS.isFoodQuery.test(message)) return await handleFood(context);
  
  // --- 價格與日期相關的判斷 (最關鍵的邏輯) ---
  if (hasDate && hasRoomType) return await handlePriceCalculation(context); // e.g., "11/3 雙人房"
  if (hasPriceWord) return await handlePriceCalculation(context); // e.g., "雙人房多少錢"
  if (hasDate && !hasRoomType) return await handleDateOnly(context, message); // e.g., "11/3"
  
  // --- 其他判斷 ---
  if (INTENTS.isPhotoQuery.test(message)) return await handlePhoto(context);

  // --- 靜態回覆 (中優先級) ---
  for (const rule of CONFIG.QUICK_REPLIES) {
    if (rule.pattern.test(message)) {
      return await replyToLine(replyToken, rule.reply, env);
    }
  }

  // --- 如果以上都沒匹配，交給最終的 GPT Fallback ---
  return await handleGenericFallback(context);
}

// --- 處理按鈕點擊 ---
// =================================================================
// Part 4: 處理函式 (Handlers)
// 每個函式只負責一件事情，讓程式碼更容易維護
// =================================================================

async function handleAction({ message, replyToken, env }) {
  const context = { message, replyToken, env };

  // --- 處理房型卡片按鈕 ---
  if (message.startsWith(CONFIG.ACTIONS.ROOM_INTRO_PREFIX)) {
    const roomId = message.replace(CONFIG.ACTIONS.ROOM_INTRO_PREFIX, "");
    
    // 再次讀取房型總表，找到對應的房型
    const allRooms = await env.KV_ROOM.get("room_master_data", "json");
    const targetRoom = allRooms.find(room => room.id === roomId);

    if (targetRoom) {
      // 組合出詳細的介紹文字
      const features = targetRoom.features.map(f => `✓ ${f}`).join("\n");
      const introText = `【${targetRoom.name}】\n\n${targetRoom.detailedIntro}\n\n✨ 房型特色：\n${features}`;
      
      // 先傳送詳細介紹
      await replyToLine(replyToken, introText, env);
      
      // 接著，如果有很多張照片，可以再追加傳送
      if (targetRoom.photos && targetRoom.photos.length > 1) {
          const photos = targetRoom.photos.slice(0, 5).map(url => ({ 
              type: "image", 
              originalContentUrl: url, 
              previewImageUrl: url 
          }));
          // 注意：追加傳送需要使用不同的 Push API，這裡我們先引導使用者
          await replyToLine(replyToken, `想看「${targetRoom.name}」的更多照片嗎？請直接輸入：「${targetRoom.name}照片」`, env);
      }
    } else {
      await replyToLine(replyToken, "抱歉，找不到該房型的詳細資訊。", env);
    }
    return;
  }

  if (message.startsWith(CONFIG.ACTIONS.ROOM_PRICE_TODAY_PREFIX)) {
    const roomType = message.replace(CONFIG.ACTIONS.ROOM_PRICE_TODAY_PREFIX, "");
    const simulatedMessage = `今天 ${roomType} 價格`;
    return await handlePriceCalculation({ message: simulatedMessage, replyToken, env });
  }

  if (message.startsWith(CONFIG.ACTIONS.ROOM_PRICE_DATE_PREFIX)) {
    const roomType = message.replace(CONFIG.ACTIONS.ROOM_PRICE_DATE_PREFIX, "");
    const prompt = `好的，您選擇了查詢「${roomType}」。\n\n為了給您最準確的報價，請務必用【日期 + 房型】的格式回覆我喔！\n\n👇 請像這樣輸入 👇\n「12/24-12/26 ${roomType}」`;
    return await replyToLine(replyToken, prompt, env);
  }
  
  // 根據我們在 CONFIG 中定義的 ACTION payload 來決定要做什麼
  switch (message) {
    // --- 停車流程 ---
    case CONFIG.ACTIONS.NAVIGATE_PARKING:
      const mapUrl = "https://maps.app.goo.gl/sgYiziTeJNqSybkW6";
      return await replyToLine(replyToken, `好的，這是前往「第二市場停車場」的 Google Maps 導航連結：\n${mapUrl}`, env);
    case CONFIG.ACTIONS.SHOW_PARKING_RULES:
      return await replyToLine(replyToken, CONFIG.MESSAGES.PARKING_RULES, env);

    // --- 房價流程 ---
    case CONFIG.ACTIONS.PRICE_ASK:
      return await replyToLine(replyToken, CONFIG.MESSAGES.PRICE_INPUT_PROMPT, env);
    case CONFIG.ACTIONS.PRICE_SHOW_ROOMS:
      return await handleRoomTypeCarousel(context);

    // --- 設施流程 ---
    case CONFIG.ACTIONS.FACILITY_WIFI:
      return await replyToLine(replyToken, CONFIG.MESSAGES.WIFI_INFO, env);
    case CONFIG.ACTIONS.FACILITY_PARKING:
      return await handleFacilityMenu_Parking(context); // 停車有下一層
    case CONFIG.ACTIONS.FACILITY_LUGGAGE:
      return await replyToLine(replyToken, CONFIG.MESSAGES.LUGGAGE_INFO, env);

    // --- 在地嚮導流程 ---
    case CONFIG.ACTIONS.GUIDE_FOOD:
      return await handleFoodCarousel(context);
    
    // --- 交通流程 ---
    case CONFIG.ACTIONS.TRANSPORT_FROM_TRA:
      return await handleTransportationMenu_TRA(context);
    case CONFIG.ACTIONS.TRANSPORT_FROM_HSR:
      const hsrText = (await env.hotelInfoKV.get("hotel_routes", "json"))?.common.find(r => r.direction === "from_hsr_to_hotel")?.text;
      return await replyToLine(replyToken, hsrText || "高鐵路線查詢失敗", env);
    case CONFIG.ACTIONS.TRANSPORT_BY_CAR:
      return await replyToLine(replyToken, CONFIG.MESSAGES.TRANSPORT_ROUTING_PROMPT, env);
    case CONFIG.ACTIONS.TRANSPORT_TRA_BUS:
        const busText = (await env.hotelInfoKV.get("hotel_routes", "json"))?.common.find(r => r.direction === "from_station_to_hotel" && r.modes.includes('bus'))?.text;
        return await replyToLine(replyToken, busText || "火車站公車路線查詢失敗", env);
    case CONFIG.ACTIONS.TRANSPORT_TRA_WALK:
        const walkText = (await env.hotelInfoKV.get("hotel_routes", "json"))?.common.find(r => r.direction === "from_station_to_hotel" && r.modes.includes('walk'))?.text;
        return await replyToLine(replyToken, walkText || "火車站步行路線查詢失敗", env);

    // --- 聯絡真人 ---
    case CONFIG.ACTIONS.CONTACT_HUMAN:
      return await handleContactHuman(context);

    default:
      console.log(`未知的 Action: ${message}`);
      return await handleGenericFallback(context);
  }
}

// --- 處理主選單 & 子選單的顯示 ---
async function handleMainMenu({ replyToken, env }) {
  const text = "您好！我是快樂腳旅棧的 AI 助理，請問需要什麼服務呢？";
  const buttons = [
      { label: "查房價／問訂房", payload: CONFIG.ACTIONS.MENU_PRICE },
      { label: "設施／服務", payload: CONFIG.ACTIONS.MENU_FACILITY },
      { label: "在地美食推薦", payload: CONFIG.ACTIONS.MENU_FOOD },
      { label: "交通指引", payload: CONFIG.ACTIONS.MENU_TRANSPORT },
  ];
  await replyWithButtons(replyToken, text, buttons, env);
}

async function handlePriceInquiryMenu({ replyToken, env }) {
  const text = "想了解訂房資訊嗎？請選擇您的需求：";
  const buttons = [
    { label: "📅 查詢特定日期房價", payload: CONFIG.ACTIONS.PRICE_ASK },
    { label: "🏠 查看所有房型介紹", payload: CONFIG.ACTIONS.PRICE_SHOW_ROOMS },
    { label: "🧑‍💼 聯絡專人", payload: CONFIG.ACTIONS.CONTACT_HUMAN },
  ];
  await replyWithButtons(replyToken, text, buttons, env);
}

async function handleFacilityMenu({ replyToken, env }) {
  const text = "想了解我們提供的設施與服務嗎？請選擇您感興趣的項目：";
  const buttons = [
    { label: "📶 WiFi 密碼", payload: CONFIG.ACTIONS.FACILITY_WIFI },
    { label: "🅿️ 停車資訊", payload: CONFIG.ACTIONS.FACILITY_PARKING },
    { label: "🧳 行李寄放", payload: CONFIG.ACTIONS.FACILITY_LUGGAGE },
  ];
  await replyWithButtons(replyToken, text, buttons, env);
}

async function handleFacilityMenu_Parking({ replyToken, env }) {
    const text = "關於停車，我們有提供補助。推薦您前往附近的「第二市場停車場」。";
    const buttons = [
        { label: "📍 導航至停車場", payload: CONFIG.ACTIONS.NAVIGATE_PARKING },
        { label: "ℹ️ 了解詳細規則", payload: CONFIG.ACTIONS.SHOW_PARKING_RULES }
    ];
    await replyWithButtons(replyToken, text, buttons, env);
}

async function handleLocalGuideMenu({ replyToken, env }) {
  const text = "沒問題！想當個在地人嗎？讓我為您推薦：";
  const buttons = [
    { label: "🍜 附近必吃美食", payload: CONFIG.ACTIONS.GUIDE_FOOD },
    { label: "📸 走路就能到的景點", payload: CONFIG.ACTIONS.GUIDE_SIGHTS },
    { label: "☕ 私藏咖啡廳名單", payload: CONFIG.ACTIONS.GUIDE_CAFES },
  ];
  await replyWithButtons(replyToken, text, buttons, env);
} 

async function handleTransportationMenu({ replyToken, env }) {
  const text = "好的，請問您是從哪裡出發呢？";
  const buttons = [
    { label: "🚆 從台中火車站", payload: CONFIG.ACTIONS.TRANSPORT_FROM_TRA },
    { label: "🚄 從台中高鐵站", payload: CONFIG.ACTIONS.TRANSPORT_FROM_HSR },
    { label: "🚗 自行開車／其他地點", payload: CONFIG.ACTIONS.TRANSPORT_BY_CAR },
  ];
  await replyWithButtons(replyToken, text, buttons, env);
}

async function handleTransportationMenu_TRA({ replyToken, env }) {
    const text = "從火車站過來，推薦您選擇：";
    const buttons = [
        { label: "🚌 搭公車 (約 10 分鐘)", payload: CONFIG.ACTIONS.TRANSPORT_TRA_BUS },
        { label: "🚶‍♂️ 散步過來 (約 25 分鐘)", payload: CONFIG.ACTIONS.TRANSPORT_TRA_WALK }
    ];
    await replyWithButtons(replyToken, text, buttons, env);
}

async function handleContactHuman({ replyToken, env }) {
  await replyToLine(replyToken, CONFIG.MESSAGES.CONTACT_HUMAN_PROMPT, env);
}

// --- 處理實際功能 (之前已有的函式) ---
// ================== 【已升級】的 handleRoomTypeCarousel 函式 ==================
async function handleRoomTypeCarousel({ replyToken, env }) {
  try {
      const allRooms = await env.KV_ROOM.get("room_master_data", "json");
      if (!allRooms || !Array.isArray(allRooms) || allRooms.length === 0) {
          return await replyToLine(replyToken, "抱歉，目前找不到房型資料。", env);
      }

      const roomCards = allRooms.map(room => ({
          imageUrl: room.imageUrl,
          title: room.name,
          text: room.text, // 顯示簡短介紹
          buttons: [
              { 
                  label: "🛌 房型詳細介紹", 
                  payload: `${CONFIG.ACTIONS.ROOM_INTRO_PREFIX}${room.id}`
              },
              { 
                  label: "💰 查今日房價", 
                  payload: `${CONFIG.ACTIONS.ROOM_PRICE_TODAY_PREFIX}${room.name}` 
              },
              { 
                  label: "📅 查其他日期", 
                  payload: `${CONFIG.ACTIONS.ROOM_PRICE_DATE_PREFIX}${room.name}`
              }
          ]
      }));

      // Line 的輪播卡片一次最多只能顯示 10 張
      await replyWithCarousel(replyToken, "為您介紹我們的所有房型", roomCards.slice(0, 10), env);

  } catch (e) {
      console.error("💥 處理房型輪播卡片失敗:", e);
      await replyToLine(replyToken, CONFIG.MESSAGES.GENERIC_ERROR, env);
  }
}

async function handleFoodCarousel({ replyToken, env }) {
    const foodKV = await env.KV_FOOD.get("nearby_food", "json");
    if (!foodKV || !foodKV.nearby_food) {
        return await replyToLine(replyToken, CONFIG.MESSAGES.FOOD_NOT_FOUND, env);
    }
    const foodCards = foodKV.nearby_food.slice(0, 10).map(food => ({
        imageUrl: food.imageUrl || "https://i.imgur.com/default-food.jpg", // 請在KV中為美食加上圖片網址
        title: `${food.name} (${food.desc})`, text: `營業時間：${food.hours}`,
        buttons: [
            { label: "📍 導航過去", payload: `從這裡到 ${food.address}` },
            { label: "👍 了解更多", payload: `${food.name} 的介紹` }
        ]
    }));
    await replyWithCarousel(replyToken, "為您推薦附近美食", foodCards, env);
}

// --- 處理房型照片 ---
async function handlePhoto({ message, replyToken, env }) {
  try {
      const allRooms = await env.KV_ROOM.get("room_master_data", "json");
      if (!allRooms) throw new Error("無法讀取房型資料");

      // 嘗試從使用者的訊息中，找出他想看的房型
      const roomType = fuzzyMatchRoom(message);
      
      if (roomType) {
          // 在我們的房型總表中尋找匹配的房型
          const targetRoom = allRooms.find(room => room.name === roomType);
          if (targetRoom && targetRoom.photos && targetRoom.photos.length > 0) {
              // 如果找到了，就回傳該房型所有的照片 (最多5張)
              const photos = targetRoom.photos.slice(0, 5).map(url => ({ 
                  type: "image", 
                  originalContentUrl: url, 
                  previewImageUrl: url 
              }));
              await replyToLineMultiple(replyToken, photos, env);
              return;
          }
      }
      
      // 如果找不到特定房型，或使用者只說「照片」，就用按鈕引導
      await handleAction({ message: CONFIG.ACTIONS.SHOW_ROOM_TYPE_PHOTOS, replyToken, env });

  } catch (e) {
      console.error("💥 房型照片處理失敗:", e);
      await replyToLine(replyToken, CONFIG.MESSAGES.GENERIC_ERROR, env);
  }
}

// --- 處理只給日期的詢問---
async function handleDateOnly({ replyToken, env }, dateStr) {
  // 步驟 1 & 2: 確認已知資訊，並詢問缺少資訊
  const text = `好的，您選擇的日期是「${dateStr}」。\n請問您想查詢哪一種房型呢？`;

  // 步驟 3: 提供所有主要房型作為快速回覆選項
  const roomTypes = ["標準雙人房", "景觀雙人房", "標準雙床房", "經濟四人房", "悠活四人房", "背包床位"];

  const quickReplies = roomTypes.map(room => ({
    label: room, // 按鈕上顯示的文字，例如 "標準雙人房"
    payload: `${dateStr} ${room}` // 點擊後，傳送的訊息會是 "11/3-11/15 標準雙人房"
  }));

  await replyWithQuickReplies(replyToken, text, quickReplies, env);
}

// --- 處理房價(自然語言) ---
async function handlePriceCalculation({ message, replyToken, env }) {
  try {
    // 注意：這裡的 fuzzyMatchRoom 和 extractDates 依賴 `message` 這個變數
    let roomType = fuzzyMatchRoom(message);
    const dateInfo = extractDates(message, CONFIG.INTENTS.isTodayQuery);

    if (dateInfo.isNextYear) {
      return await replyToLine(replyToken, CONFIG.MESSAGES.NEXT_YEAR_QUERY(dateInfo.dateString), env);
    }

    if (!dateInfo.startDate) return await replyToLine(replyToken, CONFIG.MESSAGES.ASK_FOR_DATE(roomType || '房型'), env);
    if (!roomType) {
        const buttons = [
            { label: "標準雙人房", payload: `${formatDate(new Date())} 標準雙人房 價格` },
            { label: "景觀雙人房", payload: `${formatDate(new Date())} 景觀雙人房 價格` },
            { label: "經濟四人房", payload: `${formatDate(new Date())} 經濟四人房 價格` },
        ];
        return await replyWithButtons(replyToken, CONFIG.MESSAGES.ASK_FOR_ROOM_TYPE, buttons, env);
    }

    const result = await calculateTotalPrice(env.DB, roomType, dateInfo.startDate, dateInfo.endDate);
    
    if (result.isSpecial) {
      return await replyToLine(replyToken, CONFIG.MESSAGES.SPECIAL_DAY_PRICE(result.specialDate), env);
    }
    if (result.totalPrice > 0) {
      let replyMsg;
      if (dateInfo.isRange) {
        const displayEndDate = new Date(dateInfo.endDate);
        displayEndDate.setDate(displayEndDate.getDate() + 1);
        replyMsg = `您好，${roomType}\n從 ${formatDate(dateInfo.startDate)} 到 ${formatDate(displayEndDate)} (${result.nights}晚)\n總金額為 NT$${result.totalPrice} 元。`;
        if (result.priceDetails.length > 1) {
          replyMsg += "\n\n每日價格明細：\n" + result.priceDetails.map(p => `${p.date}: NT$${p.price}`).join("\n");
        }
      } else {
        const weekdayStr = `(${getWeekday(dateInfo.startDate)})`;
        replyMsg = `您好，${formatDate(dateInfo.startDate)}${weekdayStr} ${roomType} 的價格是 NT$${result.totalPrice} 元。`;
      }
      replyMsg += CONFIG.MESSAGES.AVAILABILITY_DISCLAIMER;
      await replyToLine(replyToken, replyMsg, env);
    } else {
      await replyToLine(replyToken, CONFIG.MESSAGES.PRICE_NOT_FOUND(formatDate(dateInfo.startDate), roomType), env);
    }
  } catch (e) {
    console.error("💥 房價查詢處理失敗:", message, e); // 增加日誌，方便追蹤
    await replyToLine(replyToken, CONFIG.MESSAGES.GENERIC_ERROR, env);
  }
}

// --- 處理通用 GPT Fallback ---
async function handleGenericFallback({ message, replyToken, env }) {
    try {
        const [basicInfo, routes, food] = await Promise.all([
            env.hotelInfoKV.get("basic_info", "json"),
            env.hotelInfoKV.get("hotel_routes", "json"),
            env.KV_FOOD.get("nearby_food", "json")
        ]);
        const knowledgeBase = { generalInfo: basicInfo, transportation: routes, nearbyFood: food?.nearby_food };
        const gptResponse = await callGPTGeneral(message, knowledgeBase, env.OPENAI_API_KEY);
        if (gptResponse) {
            return await replyToLine(replyToken, gptResponse, env);
        }
    } catch (e) {
        console.error("💥 GPT Fallback 處理失敗:", e);
    }
    await replyToLine(replyToken, CONFIG.MESSAGES.FINAL_FALLBACK, env);
}

// =================================================================
// Part 5: 工具函式 (Utilities)
// 所有最底層的輔助函式都集中在這裡
// =================================================================

// --- Line API 回覆函式 ---
async function replyToLine(replyToken, text, env) {
  try {
    let msg = String(text ?? "").trim();
    if (!msg) msg = "抱歉，我這邊暫時查不到相關資訊，請稍等真人櫃檯確認後會回覆您～";
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
      body: JSON.stringify({ replyToken, messages: [{ type: "text", text: msg.slice(0, 4999) }] })
    });
    if (!res.ok) console.error("❌ LINE 文字回覆失敗:", await res.text());
  } catch (err) {
    console.error("💥 replyToLine 發生例外：", err);
  }
}

async function replyToLineMultiple(replyToken, messages, env) {
  try {
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
      body: JSON.stringify({ replyToken, messages })
    });
    if (!res.ok) console.error("❌ LINE 多訊息回覆失敗：", await res.text());
  } catch (err) {
    console.error("💥 replyToLineMultiple 發生例外：", err);
  }
}

async function replyWithQuickReplies(replyToken, text, quickReplies, env) {
  const messages = [{
    type: "text",
    text: text,
    quickReply: {
      items: quickReplies.map(qr => ({
        type: "action",
        action: {
          type: "message",
          label: qr.label, // 顯示給使用者看的按鈕文字
          text: qr.payload  // 使用者點擊後，實際傳送給我們的訊息
        }
      }))
    }
  }];

  try {
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
      body: JSON.stringify({ replyToken, messages })
    });
    if (!res.ok) {
      console.error("❌ LINE 快速回覆失敗:", await res.text());
    }
  } catch (err) {
    console.error("💥 replyWithQuickReplies 發生例外：", err);
  }
}

// ================== 【偵錯專用】的 replyWithButtons 函式 ==================

async function replyWithButtons(replyToken, text, buttons, env) {
  const payload = {
    replyToken,
    messages: [{
      type: "template",
      altText: text, 
      template: {
        type: "buttons",
        text: text,
        actions: buttons.map(btn => ({
          type: "message",
          label: btn.label, 
          text: btn.payload
        }))
      }
    }]
  };

  // --- 偵錯日誌 #1 ---
  // 印出我們準備要傳送給 Line 的完整 JSON 資料
  console.log("🟡 [DEBUG] 準備傳送按鈕訊息，Payload 內容:");
  console.log(JSON.stringify(payload, null, 2)); // 用美化格式印出 JSON

  try {
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
      body: JSON.stringify(payload)
    });

    // --- 偵錯日誌 #2 ---
    // 無論成功失敗，都印出 Line 伺服器的回應狀態碼
    console.log(`🔵 [DEBUG]收到 Line API 回應，HTTP 狀態碼: ${res.status}`);

    if (!res.ok) {
      // --- 偵錯日誌 #3 ---
      // 如果狀態不是成功 (例如 400)，就印出詳細的錯誤內容
      const errorBody = await res.text();
      console.error("❌ [DEBUG] LINE 按鈕回覆失敗，伺服器錯誤訊息:", errorBody);
    } else {
      console.log("✅ [DEBUG] LINE 按鈕訊息成功發送。");
    }

  } catch (err) {
    // --- 偵錯日誌 #4 ---
    // 如果連 fetch 指令本身都失敗 (例如網路問題)，印出例外
    console.error("💥 [DEBUG] replyWithButtons 函式發生無法預期的例外:", err);
  }
}
async function replyWithCarousel(replyToken, altText, cards, env) {
  const messages = [{
    type: "template",
    altText: altText,
    template: {
      type: "carousel",
      columns: cards.map(card => ({
        thumbnailImageUrl: card.imageUrl,
        title: card.title,
        text: card.text,
        actions: card.buttons.map(btn => ({
          type: "message",
          label: btn.label,
          text: btn.payload
        }))
      }))
    }
  }];
  try {
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
        method: "POST",
        headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
        body: JSON.stringify({ replyToken, messages })
    });
    if (!res.ok) console.error("❌ LINE 輪播卡片回覆失敗:", await res.text());
  } catch (err) {
    console.error("💥 replyWithCarousel 發生例外：", err);
  }
}

// --- 日期與價格處理函式 ---
function formatDate(date) {
  return `${date.getMonth() + 1}/${date.getDate()}`;
}

function getWeekday(date) {
  return "日一二三四五六".charAt(date.getDay());
}

function isQueryForNextYear(dateStr) {
    const match = dateStr.match(/(\d{1,2})[\/\-月](\d{1,2})/);
    if (!match) return { isNextYear: false, fullDate: null };
    const month = String(match[1]).padStart(2, "0");
    const day = String(match[2]).padStart(2, "0");
    const now = new Date();
    let year = now.getFullYear();
    const today = new Date(now.getFullYear(), now.getMonth(), now.getDate());
    const prospectiveDate = new Date(`${year}-${month}-${day}`);
    if (prospectiveDate < today) {
        return { isNextYear: true, fullDate: `${year + 1}-${month}-${day}` };
    }
    return { isNextYear: false, fullDate: `${year}-${month}-${day}` };
}

function extractDates(text, todayWordsRegex) {
    if (todayWordsRegex.test(text)) {
        const today = new Date();
        return { startDate: today, endDate: today, isRange: false, isNextYear: false };
    }
    const rangeMatch = text.match(/(\d{1,2}[\/\\-月]\d{1,2})\s*(?:-|~|到|至|－)\s*(\d{1,2}[\/\\-月]\d{1,2})/);
    if (rangeMatch) {
        const startStr = rangeMatch[1];
        const endStr = rangeMatch[2];
        const checkResult = isQueryForNextYear(startStr);
        if (checkResult.isNextYear) {
            return { startDate: null, endDate: null, isRange: true, isNextYear: true, dateString: startStr };
        }
        const startDate = new Date(checkResult.fullDate);
        const endDateObj = new Date(isQueryForNextYear(endStr).fullDate);
        endDateObj.setDate(endDateObj.getDate() - 1);
        return { startDate, endDate: endDateObj, isRange: true, isNextYear: false };
    }
    const singleMatch = text.match(/(\d{1,2}[\/\\-月]\d{1,2})/);
    if (singleMatch) {
        const dateStr = singleMatch[0];
        const checkResult = isQueryForNextYear(dateStr);
        if (checkResult.isNextYear) {
            return { startDate: null, endDate: null, isRange: false, isNextYear: true, dateString: dateStr };
        }
        const date = new Date(checkResult.fullDate);
        return { startDate: date, endDate: date, isRange: false, isNextYear: false };
    }
    return { startDate: null, endDate: null, isRange: false, isNextYear: false };
}

function fuzzyMatchRoom(text) {
    const rooms = [
      { keys: ["悠活", "家庭", "高級四人"], value: "悠活四人房" },
      { keys: ["經濟四人"], value: "經濟四人房" },
      { keys: ["四人", "4人"], value: "經濟四人房" },
      { keys: ["雙床"], value: "標準雙床房" },
      { keys: ["景觀", "市景", "view"], value: "景觀雙人房" },
      { keys: ["標準雙人"], value: "標準雙人房" },
      { keys: ["雙人", "情人", "2人"], value: "標準雙人房" },
      { keys: ["女生", "女背包"], value: "女生背包房" },
      { keys: ["男生", "男背包"], value: "男生背包房" }
    ];
    for (const r of rooms) {
        if (r.keys.some(k => text.includes(k))) return r.value;
    }
    return null;
}

async function fetchPriceFromD1(db, roomType, date) {
    const dateObj = (date instanceof Date) ? date : new Date(date);
    if (isNaN(dateObj.getTime())) {
        console.error("傳入 fetchPriceFromD1 的日期無效:", date);
        return null;
    }
    const stmt = db.prepare("SELECT weekday_price AS 平日, friday_price AS 週五, saturday_price AS 週六 FROM room_prices WHERE room_type = ?");
    const result = await stmt.bind(roomType).first();
    if (!result) return null;
    const weekday = dateObj.getDay();
    if (weekday === 5) return result["週五"];
    if (weekday === 6 || weekday === 0) return result["週六"];
    return result["平日"];
}

async function calculateTotalPrice(db, roomType, startDate, endDate) {
    let totalPrice = 0;
    let nights = 0;
    let priceDetails = [];
    const specialDates = [["10-03", "10-06"], ["10-09", "10-12"], ["12-31", "12-31"]];
    const results = { totalPrice, nights, priceDetails, isSpecial: false, specialDate: '' };
    if (!startDate || !endDate || endDate < startDate) return results;
    let currentDate = new Date(startDate);
    while (currentDate <= endDate) {
        const dateStr = currentDate.toISOString().split('T')[0];
        const [, m, d] = dateStr.split("-");
        const mmdd = `${m}-${d}`;
        if (specialDates.some(([start, end]) => mmdd >= start && mmdd <= end)) {
            results.isSpecial = true;
            results.specialDate = `${m}/${d}`;
            return results;
        }
        const dailyPrice = await fetchPriceFromD1(db, roomType, currentDate);
        if (dailyPrice === null) {
            console.error(`查無 ${dateStr} ${roomType} 的價格`);
            return { ...results, totalPrice: 0 }; // 查不到價格就中斷
        }
        totalPrice += dailyPrice;
        priceDetails.push({ date: formatDate(currentDate), price: dailyPrice });
        nights++;
        currentDate.setDate(currentDate.getDate() + 1);
    }
    results.totalPrice = totalPrice;
    results.nights = nights;
    return results;
}

function extractRoomAndDate(text) {
  const room = fuzzyMatchRoom(text);
  const match = text.match(/(\d{1,2})[\/\-月](\d{1,2})/);
  if (!match) return { roomType: room, date: null };
  const year = new Date().getFullYear();
  const month = `${match[1]}`.padStart(2, "0");
  const day = `${match[2]}`.padStart(2, "0");
  return { roomType: room, date: `${year}-${month}-${day}` };
}

// --- 交通與地圖函式 ---
function parseTravelQuery(msg) {
    const mWalk = /(走路|步行)/i.test(msg);
    const mDrive = /(開車|自行開車|騎車|機車|汽車)/i.test(msg);
    const mTransit = /(公車|客運|轉乘|大眾運輸|搭車)/i.test(msg);
    const mode = mWalk ? "walk" : mDrive ? "drive" : mTransit ? "walk" : "walk";
    let origin = "台中火車站";
    const m1 = msg.match(/從(.+?)(?:到|去|前往)/);
    if (m1 && m1[1]) {
        origin = m1[1].trim().replace(/快樂腳旅棧|中華路一段185號/gi, "").trim();
    }
    if (!origin.includes("台中") && !origin.includes("高鐵")) {
        origin = "台中" + origin;
    }
    if (origin.includes("高鐵") && !origin.includes("台中")) {
        origin = "台中高鐵站";
    }
    return { origin, mode };
}

function pickRouteFromKV(origin, mode, facts) {
    if (!facts || !Array.isArray(facts.common)) return null;
    const o = (origin || "").toLowerCase();
    const m = (mode || "walk").toLowerCase();
    for (const item of facts.common) {
        const aliases = (item.aliases || []).map(x => String(x).toLowerCase());
        const modes = (item.modes || []).map(x => String(x).toLowerCase());
        if (aliases.some(a => o.includes(a)) && (!modes.length || modes.includes(m))) {
            return item.text;
        }
    }
    return null;
}

async function geocodeNominatim(query) {
    const taichungViewbox = "&viewbox=120.5,24.3,120.8,24.0&bounded=1";
    const url = `https://nominatim.openstreetmap.org/search?format=jsonv2&limit=1&addressdetails=1&countrycodes=tw&q=${encodeURIComponent(query)}${taichungViewbox}`;
    const res = await fetch(url, {
        headers: { "User-Agent": "HappyInnBot/1.0 (contact: cath82319@gmail.com)" } // 請務必填寫您的 Email
    });
    if (!res.ok) throw new Error(`Nominatim API failed with status: ${res.status}`);
    const data = await res.json().catch(() => null);
    if (!Array.isArray(data) || !data.length) return null;
    const p = data[0];
    return { lat: Number(p.lat), lon: Number(p.lon), display_name: p.display_name || "" };
}

async function osrmRoute({ origin, dest, mode, hotelName = "目的地", hotelAddr = "" }) {
    const profile = mode === "drive" ? "car" : "foot";
    const coords = `${origin.lon},${origin.lat};${dest.lon},${dest.lat}`;
    const url = `https://router.project-osrm.org/route/v1/${profile}/${coords}?overview=false&steps=true&alternatives=false&annotations=false`;
    const res = await fetch(url, {
        headers: { "User-Agent": "HappyInnBot/1.0 (contact: cath82319@gmail.com)" } // 請務必填寫您的 Email
    });
    if (!res.ok) throw new Error(`OSRM API failed with status: ${res.status}`);
    const data = await res.json().catch(() => null);
    if (!data || data.code !== "Ok" || !Array.isArray(data.routes) || !data.routes.length) return null;
    const route = data.routes[0];
    const leg = route.legs && route.legs[0];
    if (!leg) return null;
    const durMin = Math.round((route.duration || 0) / 60);
    const distKm = (route.distance || 0) / 1000;
    const steps = (leg.steps || []).slice(0, 6);
    const stepLines = steps.map((s, i) => {
        const road = (s.name || "").trim();
        const maneuver = s.maneuver?.type || "";
        const mod = s.maneuver?.modifier || "";
        const act = formatManeuver(maneuver, mod);
        return `${i + 1}. ${act}${road ? `（${road}）` : ""}`;
    });
    const headerIcon = profile === "foot" ? "🚶" : "🚗";
    const header = `${headerIcon} 路線：約 ${durMin} 分鐘（約 ${distKm.toFixed(1)} 公里）\n目的地：${hotelName}（${hotelAddr}）`;
    return [header, ...stepLines].join("\n");
}

function formatManeuver(type, mod) {
    const dirMap = { left: "向左", right: "向右", straight: "直行", slight_left: "微靠左", slight_right: "微靠右", uturn: "迴轉" };
    const dir = dirMap[mod] || "";
    switch (type) {
        case "depart": return "出發直行";
        case "arrive": return "抵達目的地";
        case "turn": return dir || "轉彎";
        case "roundabout": return "進入環島";
        case "fork": return "於岔路口";
        case "merge": return "匯入主線";
        default: return `前行${dir ? `（${dir}）` : ""}`;
    }
}

// --- GPT 函式 ---
async function callGPTGeneral(message, knowledgeBase, apiKey) {
    if (!apiKey || !knowledgeBase) return null;
    const context = JSON.stringify(knowledgeBase, null, 2);
    const system_prompt = `你是「快樂腳旅棧」的AI客服，你的任務是根據下方提供的【飯店知識庫】，用親切、簡潔的口吻回答使用者的問題。
【回答規則】
1. **絕對禁止**回答任何【飯店知識庫】中沒有提到的資訊，嚴禁任何猜測或編造。
2. 如果資料中明確指出「沒有」某項設施，請直接、肯定地回覆「我們沒有提供喔」，並可補充替代方案。
3. 如果【飯店知識庫】中找不到使用者問題的相關資訊，你的唯一標準回覆必須是：「不好意思，您的問題可能需要由真人櫃檯協助處理，我已經通知他們了，請您稍候喔！」
4. 你的回答必須完全基於提供的資料，不要添加任何額外的個人意見或建議。
5. 保持回覆簡短，最多 5 行，盡量條列式說明。
【飯店知識庫】
${context}`;
    const body = {
        model: "gpt-4o",
        temperature: 0.1,
        messages: [{ role: "system", content: system_prompt }, { role: "user", content: message }],
        max_tokens: 300
    };
    try {
        const res = await fetch("https://api.openai.com/v1/chat/completions", {
            method: "POST",
            headers: { "Content-Type": "application/json", Authorization: `Bearer ${apiKey}` },
            body: JSON.stringify(body)
        });
        if (!res.ok) return null;
        const data = await res.json();
        const responseText = data.choices?.[0]?.message?.content?.trim() || null;
        if (responseText && (responseText.includes("？") || responseText.includes("嗎？"))) {
            return "您的問題比較複雜，我已通知真人櫃檯，請稍候片刻，謝謝您！";
        }
        return responseText;
    } catch (e) {
        console.error("💥 GPT Fact-Based Q&A 失敗：", e);
        return null;
    }
}
