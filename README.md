// ================== 常數 & 意圖判斷 ==================
const SPECIAL_DATES = [
  ["10-03", "10-06"],
  ["10-09", "10-12"],
  ["12-31", "12-31"]
];

const QUICK_REPLIES = [
  { pattern: /(Wi[-]?Fi|wifi|無線網路)/i, reply: "我們有提供免費 Wi‑Fi，帳號是 Happyinn，密碼是 22232033，歡迎使用喔！" },
  { pattern: /(提早到|提早入住|提前入住|提早抵達|提前抵達|提前check[- ]?in|早點入住|13點入住|1點入住)/i,
    reply: "我們入住時間是下午三點喔，如果您提早抵達，可以先到交誼廳稍作休息哦～" },
  { pattern: /(寄放|行李|退房後)/i, reply: "不論是提早抵達還是退房後，我們都可以免費寄放行李，但記得退房後寄放的話要在當天取回喔！" },
  { pattern: /(櫃台|服務時間|check in|check-in|checkin)/i, reply: "我們櫃台是 24 小時全年無休的，有任何問題隨時都可以找我們喔！" },
  { pattern: /(停車|停車位)/i, reply:
`8/31 前我們有提供免費停車（第二市場停車場），8/31 後則改為補助制：
★ 限房費每晚 1150 元以上（背包房不適用）
★ 每房每日補助一台車，最高 100 元
★ 需出示紙本停車單據（電子發票無法補助）
★ 限距離本館 1 公里內的一般費率停車場或停車格喔！` },
  { pattern: /(刷牙|牙刷|拖鞋|一次性|毛巾|浴巾|沐浴|洗髮|刮鬍刀|梳子|浴帽)/i,
    reply: "從 114 年起我們不再主動提供牙刷、刮鬍刀、拖鞋等一次性備品喔～如果需要的話可以到櫃檯購買。\n不過洗沐浴乳、洗手乳、毛巾浴巾都還是有提供的！\n p.s.背包房需自備毛巾浴巾" },
  { pattern: /(訂房|預訂|預定|空房|還有房間)/i, reply: "訂房的部分要由櫃台人員協助喔～請您稍等一下，會有專人回覆您！" },
  { pattern: /(附早餐|有早餐|看.*菜單|想看菜單|早餐菜單|菜單內容)/i,
    reply: "我們旅棧本身沒有附早餐，但有提供代訂「晨家廚房」的服務哦～一份大約 $140～165，想吃熱呼呼的早餐只要前一天告知我們就好 😊\n\n🍔 麥香雞漢堡套餐 $160\n🥩 里肌豬排漢堡套餐 $165\n👦 兒童早餐套餐 $140\n🐟 鮪魚漢堡套餐 $170\n🐔 燻雞漢堡套餐 $175\n🥬 素食選項也有提供唷～\n\n（想看圖片的話可以輸入：早餐照片）" }
];

const PRICE_WORDS = /(價|價格|費用|房價|多少|有房)/i;
const TODAY_WORDS = /(今天|今日)/i;
const LOCAL_WORDS = /(景點|美食|餐廳|酒吧|小吃|義大利|甜點|好吃|好玩|附近|怎麼去|交通|公車|火車|高鐵|柳川|一中|中華夜市|審計|中國醫|夜市)/i;
const PHOTO_WORDS = /(照片|圖片|相片)/i;
const DATE_WORDS  = /(\d{1,2}[\/\-月]\d{1,2})/;
const FACILITY_WORDS = /(設施|設備|庭院|交誼廳|飲水機|咖啡機|飲料機|氣泡水|市景|洗衣|自助洗衣|投幣|送洗|烘衣|腳踏車|單車|租借|YouBike|wifi|wi[- ]?fi|無線網路|寄放|行李|前台|櫃台|入住|退房|check[\- ]?in|check[\- ]?out|停車|寵物|禁菸|一次性|備品|毛巾|牙刷)/i;

function isLocalInfoQuery(msg){ return LOCAL_WORDS.test(msg) && !PHOTO_WORDS.test(msg); }

// ================== Worker 入口 ==================
export default {
  async fetch(request, env, ctx) {
    if (request.method === "GET") return new Response("ok", { status: 200 });
    if (request.method !== "POST") return new Response("只接受 POST 請求", { status: 405 });

    let body;
    try {
      body = await request.json();
      console.log("📥 RAW body:", JSON.stringify(body).slice(0, 2000));
    } catch {
      return new Response("JSON 格式錯誤", { status: 400 });
    }
    const event = body.events?.[0];
    const message = event?.message?.text || "";
    const replyToken = event?.replyToken;
    if (!message || !replyToken) return new Response("OK", { status: 200 });
    console.log("🙋 User said:", message);

    ctx.waitUntil(handleEvent(message, replyToken, env));
    return new Response("OK", { status: 200 });
  },
};

async function handleEvent(message, replyToken, env) {
  // 取得所需 KV（注意：這裡就是三個都拿）
  const [roomPhotoKV, menuObj, foodKV] = await Promise.all([
    env.KV_ROOM?.get?.("room_photos", "json"),
    env.KV_MENU?.get?.("breakfast_menu", "json"),
    env.KV_FOOD?.get?.("nearby_food", "json"),
  ]);

  // 0) 固定政策硬攔截（GPT 之前）
  const FIXED_KEYWORDS = [
    {
      pattern: /(早餐|早上吃什麼|morning meal)/i,
      condition: msg => !/(照片|圖片|相片)/i.test(msg),
      reply: "我們的房費是不含早餐的呦，但有提供代訂「晨家廚房」的服務哦～一份大約 $140～165，想吃熱呼呼的早餐只要前一天告知我們就好 😊\n\n🍔 麥香雞漢堡套餐 $160\n🥩 里肌豬排漢堡套餐 $165\n👦 兒童早餐套餐 $140\n🐟 鮪魚漢堡套餐 $170\n🐔 燻雞漢堡套餐 $175\n🥬 素食選項也有提供唷～\n\n（想看圖片的話可以輸入：早餐照片）"
    },
    {
      pattern: /(停車)/i,
      reply: "8/31 前入住可免費停車於第二市場停車場；9/1 起改為補助制，需房費每晚1150元以上（背包房不補助），每房每日限補助一車、最多補助100元，限1公里內停車場或一般費率停車格，憑紙本單據補助（載具不適用）。"
    },
    {
      pattern: /(牙刷|拖鞋|一次性備品|盥洗用品)/i,
      reply: "從 114 年起我們不再主動提供一次性盥洗用品喔～若需要可至櫃檯購買；洗沐浴乳、洗手乳、毛巾浴巾都有提供。背包房需自備毛巾浴巾。"
    },
    { pattern: /(入住時間|check[- ]?in)/i, reply: "入住時間為下午三點；提早抵達可先在交誼廳休息喔～" },
    { pattern: /(退房|check[- ]?out)/i, reply: "退房時間為上午11點，若需延後退房請提前告知並付款，每小時加收200元（最晚至13:00）。" }
  ];
  for (const kw of FIXED_KEYWORDS) {
    if (kw.pattern.test(message) && (!kw.condition || kw.condition(message))) {
      await replyToLine(replyToken, kw.reply, env);
      return;
    }
  }

  // 1) 早餐「照片」— 必須在所有房型照片判斷之前
  if (/(早餐).*(照片|圖片|相片)/i.test(message)) {
    const items = Array.isArray(menuObj?.breakfast_menu) ? menuObj.breakfast_menu : [];
    const photos = items.map(i => i.image).filter(Boolean).slice(0, 5);
    if (photos.length) {
      const msgs = photos.map(url => ({ type: "image", originalContentUrl: url, previewImageUrl: url }));
      await replyToLineMultiple(replyToken, msgs, env);
      return;
    }
    await replyToLine(replyToken, "目前找不到早餐照片，請稍後再試～", env);
    return;
  }

  // 2) 房型照片（排除含「早餐」的語句）
  if (PHOTO_WORDS.test(message) && !/早餐/.test(message)) {
    const { roomType } = extractRoomAndDate(message);
    if (roomType && roomPhotoKV && typeof roomPhotoKV === "object") {
      const urls = roomPhotoKV[roomType];
      if (Array.isArray(urls) && urls.length > 0) {
        const photos = urls.slice(0, 5).map(url => ({ type: "image", originalContentUrl: url, previewImageUrl: url }));
        await replyToLineMultiple(replyToken, photos, env);
        return;
      }
    }
    await replyToLine(replyToken, "目前找不到這個房型的照片，請再確認房型名稱或詢問櫃檯喔～", env);
    return;
  }

  // 3) 模糊房型照片引導（避免誤吃早餐）
  if (/(房型|交誼廳|環境|房間).*(照片|圖片|相片)?/i.test(message) && !/早餐/.test(message)) {
    await replyToLine(
      replyToken,
      "您想看哪一種照片呢？例如：交誼廳、景觀房、雙床房、經濟四人房、背包房等～，並請+照片",
      env
    );
    return;
  }
  // 3.x) 交通路線（混合模式：先查 KV，其次 OSM：Nominatim + OSRM）
if (/(怎麼去|怎麼到|如何到|怎樣到|路線|走路|步行|開車|騎車|搭車|公車|轉乘|到.*快樂腳旅棧|到.*中華路一段185號)/i.test(message)) {
  const travel = parseTravelQuery(message); // 解析起點與模式
  const routesKV = await env.hotelInfoKV.get("hotel_routes", "json");
  const hotelName = routesKV?.hotel_name || "快樂腳旅棧";
  const hotelAddr = routesKV?.hotel_address || "台中市中區中華路一段185號";
  const hotelCoord = routesKV?.hotel_coords || null; // 可選：{lat, lon}

  // 1) 先用 KV 常見起點（最穩）
  const kvHit = pickRouteFromKV(travel.origin, travel.mode, routesKV);
  if (kvHit) {
    await replyToLine(replyToken, kvHit, env);
    return;
  }

  // 2) Nominatim 地理編碼（免費）
  const dest = hotelCoord || await geocodeNominatim(hotelAddr);
  const orig = await geocodeNominatim(travel.origin);
  if (orig && dest) {
    // 3) OSRM 公開路由（免費）
    const routeText = await osrmRoute({
      origin: orig,
      dest: dest,
      mode: travel.mode,
      hotelName,
      hotelAddr
    });
    if (routeText) {
      await replyToLine(replyToken, routeText, env);
      return;
    }
  }

  // 4) 全部失敗 → 安全回覆（附地圖連結）
  const gmap = `https://www.google.com/maps/dir/?api=1&destination=${encodeURIComponent(hotelAddr)}`;
  await replyToLine(
    replyToken,
    `建議用地圖導航查詢「${travel.origin} → ${hotelAddr}」。\n快速開啟：${gmap}`,
    env
  );
  return;
}

  // 4) 附近美食（KV 硬攔截）— 一定要在 GPT 之前
  if (/(附近|周邊|周遭).*(美食|餐廳|小吃|酒吧|吃的|好吃|夜市)|美食推薦|吃什麼|推薦美食|要吃什麼/i.test(message)) {
    // 你的 KV 結構：{ "nearby_food": [ ... ] }
    const list = Array.isArray(foodKV?.nearby_food) ? foodKV.nearby_food : (Array.isArray(foodKV) ? foodKV : []);
    if (list.length) {
      let fiveMin = "🥢 步行五分鐘可達：\n";
      let tenMin  = "🛍️ 步行十分鐘：\n";
      let bar     = "🍸 酒吧推薦：\n";

      list.forEach(item => {
        const line = `• ${item.name}（${item.desc}）｜${item.hours}${item.closed ? `｜公休：${item.closed}` : ""}｜${item.address}`;
        if ((item.desc || "").includes("酒吧") || (item.address || "").includes("本館樓下")) {
          bar += line + "\n";
        } else if ((item.address || "").includes("成功路") || (item.address || "").includes("篤行路") || (item.address || "").includes("台灣大道一段560號") || (item.address || "").includes("中華路一段154巷")) {
          fiveMin += line + "\n";
        } else {
          tenMin += line + "\n";
        }
      });

      const replyMsg = `${fiveMin}\n${tenMin}\n${bar}`.trim();
      await replyToLine(replyToken, replyMsg, env);
      return;
    }
    await replyToLine(replyToken, "目前找不到附近美食資訊，請稍後再試～", env);
    return;
  }

  // 5) QUICK_REPLIES（保留）
  for (const rule of QUICK_REPLIES) {
    if (rule.pattern.test(message)) {
      await replyToLine(replyToken, rule.reply, env);
      return;
    }
  }

  // 6) 設施／規定／旅棧介紹（只依 KV）
  if (FACILITY_WORDS.test(message) || /(介紹|有什麼|館內).*(設施|設備)/i.test(message)) {
    const facts = await getHotelFacts(env);

    if (/洗衣|自助洗衣|投幣|送洗|烘衣/.test(message) && facts?.nonFacilities?.laundry?.available === false) {
      const alt = facts.nonFacilities.laundry.alternatives || {};
      const laundromat = alt.laundromat ? `\n• 投幣式洗衣店：${alt.laundromat.address}（${alt.laundromat.description}）` : "";
      const svc = alt.laundryService ? `\n• 代為送洗：${alt.laundryService.price}，${alt.laundryService.time}` : "";
      await replyToLine(replyToken, `我們館內沒有洗衣設備。${laundromat}${svc}\n需要我把位置或價格再詳細傳給您嗎？`, env);
      return;
    }

    if (/腳踏車|單車|租借|YouBike/i.test(message) && facts?.nonFacilities?.bikeRental?.available === false) {
      const yb = facts.nonFacilities.bikeRental.alternatives?.youBike;
      const locs = Array.isArray(yb?.locations) ? `（附近站點：${yb.locations.join("、")}）` : "";
      await replyToLine(replyToken, `我們沒有腳踏車出租服務。可使用 YouBike${locs}，走路約 5 分鐘可到。需要我傳 Google 地圖給您嗎？`, env);
      return;
    }

    const intro = renderFacilityIntro(facts);
    await replyToLine(replyToken, intro, env);
    return;
  }

  // 7) 房價導流（不等 LLM）
  if (PRICE_WORDS.test(message)) {
    const hasToday = TODAY_WORDS.test(message);
    const hasDateToken = DATE_WORDS.test(message) || hasToday;
    let { roomType } = extractRoomAndDate(message);
    let rt = roomType || fuzzyMatchRoom(message);

    // 新增：今天房價若沒有房型 → 預設標準雙人房
    if (hasToday && !rt) {
        rt = "標準雙人房";
    }

    if (!hasDateToken && !rt) {
        await replyToLine(replyToken,
            "房價會依日期與房型不同～請告訴我：\n• 入住日期（例如 7/20 或 今天）\n• 房型（例如 標準雙人房、景觀雙人房、經濟四人房…）\n範例：7/20 標準雙人房 價格",
            env
        );
        return;
    }
    if (!hasDateToken) {
        await replyToLine(replyToken, `想查哪一天的價格呢？可以輸入「今天」或日期（例如 7/20）。\n範例：今天 ${rt} 價格`, env);
        return;
    }
    if (!rt) {
        await replyToLine(replyToken,
            "想查哪一種房型呢？例如：標準雙人房、標準雙床房、景觀雙人房、經濟四人房、女生背包房…",
            env
        );
        return;
    }

    let qDate;
    let dateDisplay = "";
    if (hasToday) {
        const t = new Date();
        const y = t.getFullYear();
        const m = t.getMonth() + 1;
        const d = t.getDate();
        // 星期顯示
        const days = ["日","一","二","三","四","五","六"];
        const w = days[t.getDay()];
        dateDisplay = `(${m}月${d}日星期${w})`;
        qDate = `${y}-${String(m).padStart(2, "0")}-${String(d).padStart(2, "0")}`;
    } else {
        qDate = extractRoomAndDate(message).date;
    }

    if (qDate && isSpecialDate(qDate)) {
        await replyToLine(replyToken, `這段期間（${qDate}）為特殊節日，房價請聯絡櫃檯報價喔～`, env);
        return;
    }

    const price = qDate ? await fetchPriceFromD1(env.DB, rt, qDate) : null;
    if (price) {
        if (hasToday) {
            await replyToLine(replyToken, `今天${dateDisplay} ${rt} 的價格是 NT$${price}。\n如果需要其他房型或日期，請再詢問我哦~`, env);
        } else {
            await replyToLine(replyToken, `${qDate} ${rt} 的價格是 NT$${price}。\n如果需要其他房型或日期，請再詢問我哦~`, env);
        }
    } else {
        await replyToLine(replyToken, `${hasToday ? "今天" : qDate} ${rt} 的價格目前查不到，可能為特殊日期或已滿房，建議聯絡櫃檯喔～`, env);
    }
    return;
}



  // 8) LLM 意圖 & 其餘分支（最後才進 GPT）
  if (isLocalInfoQuery(message)) {
    // 這裡仍保留在地 GPT，但因為我們已經把「美食」專案硬攔截了，所以不會亂推全台
    try {
      const text = await callGPTForLocalInfo(message, env.OPENAI_API_KEY);
      await replyToLine(replyToken, text || "我這邊暫時查不到相關資訊，您可以先跟櫃檯確認～", env);
    } catch {
      await replyToLine(replyToken, "抱歉我這邊暫時忙線上，請稍後再問我一次～", env);
    }
    return;
  }

  const intent = await classifyIntent(message, env.OPENAI_API_KEY);
  console.log("🧭 intent:", intent);

  if (intent?.type === "PHOTOS") {
    const rt = intent.roomType || fuzzyMatchRoom(message);
    if (!rt) { await replyToLine(replyToken, "想看哪種房型的照片呢？例如：景觀雙人房、標準雙床房…", env); return; }
    const kv = await env.KV_ROOM.get("room_photos", "json");
    const urls = kv?.[rt];
    if (Array.isArray(urls) && urls.length) {
      const photos = urls.slice(0, 5).map(url => ({ type: "image", originalContentUrl: url, previewImageUrl: url }));
      await replyToLineMultiple(replyToken, photos, env);
    } else {
      await replyToLine(replyToken, `目前找不到「${rt}」的照片，換個房型試試看嗎？`, env);
    }
    return;
  }

  if (intent?.type === "LOCAL_INFO") {
    const text = await callGPTForLocalInfo(message, env.OPENAI_API_KEY);
    await replyToLine(replyToken, text || "我這邊暫時查不到，您可以先跟櫃檯確認～", env);
    return;
  }

  if (intent?.type === "PRICE_TODAY") {
    const rt = intent.roomType || fuzzyMatchRoom(message) || "標準雙人房";
    const t = new Date(); const y = t.getFullYear(); const m = String(t.getMonth() + 1).padStart(2, "0"); const d = String(t.getDate()).padStart(2, "0");
    const dateStr = `${y}-${m}-${d}`;
    const price = await fetchPriceFromD1(env.DB, rt, dateStr);
    await replyToLine(replyToken, price ? `今天 ${rt} 的價格是 NT$${price}。` : `今天 ${rt} 的價格查不到，可能為特殊日期或已滿房，建議聯絡櫃檯喔～`, env);
    return;
  }

  if (intent?.type === "PRICE_DATED" && intent.date) {
    const rt = intent.roomType || fuzzyMatchRoom(message) || "標準雙人房";
    if (isSpecialDate(intent.date)) { await replyToLine(replyToken, `這段期間（${intent.date}）為特殊節日，房價請聯絡櫃檯報價喔～`, env); return; }
    const price = await fetchPriceFromD1(env.DB, rt, intent.date);
    await replyToLine(replyToken, price ? `${intent.date} ${rt} 的價格是 NT$${price}。` : `${intent.date} ${rt} 的價格目前查不到，建議聯絡櫃檯確認喔～`, env);
    return;
  }

  // 9) 最後一般 GPT & Fallback
  try {
    const gpt = await callGPTGeneral(message, env.OPENAI_API_KEY);
    if (gpt) { await replyToLine(replyToken, gpt, env); return; }
  } catch (e) {
    console.error("💥 GPT 一般回覆失敗：", e);
  }

  await replyToLine(replyToken, [
    "我還不太確定您的意思～您可以問我：",
    "• 查詢房價（請輸入日期與房型）",
    "• 看早餐照片 / 早餐菜單",
    "• 看交誼廳或房型照片（輸入：景觀房照片）",
    "• 詢問 Wi‑Fi、寄放行李、入住時間等～",
  ].join("\n"), env);
}

// ================== 工具函式 ==================

// 判斷特殊節日
function isSpecialDate(date) {
  const [, m, d] = date.split("-");
  const mmdd = `${m}-${d}`;
  return SPECIAL_DATES.some(([start, end]) => mmdd >= start && mmdd <= end);
}

// 文字回覆（加入保底文字，避免空字串）
async function replyToLine(replyToken, text, env) {
  try {
    let msg = String(text ?? "").trim();
    if (!msg) msg = "抱歉，我這邊暫時查不到相關資訊，幫您請櫃檯確認一下～";
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
      body: JSON.stringify({ replyToken, messages: [{ type: "text", text: msg.slice(0, 1900) }] })
    });
    const body = await res.text();
    console.log("🔎 LINE reply status:", res.status);
    if (!res.ok) console.error("❌ LINE 回覆失敗:", body);
  } catch (err) {
    console.error("💥 replyToLine 發生例外：", err);
  }
}

// 多圖回覆
async function replyToLineMultiple(replyToken, messages, env) {
  try {
    const res = await fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${env.CHANNEL_ACCESS_TOKEN}` },
      body: JSON.stringify({ replyToken, messages })
    });
    const t = await res.text();
    console.log("🔎 LINE multi status:", res.status);
    if (!res.ok) console.error("❌ LINE 多圖回覆失敗：", t);
  } catch (err) {
    console.error("❌ 發送多圖時發生錯誤：", err);
  }
}

// 擷取房型與日期
function extractRoomAndDate(text) {
  const room = fuzzyMatchRoom(text);
  const match = text.match(/(\d{1,2})[\/\-月](\d{1,2})/);
  if (!match) return { roomType: room, date: null };
  const year = new Date().getFullYear();
  const month = `${match[1]}`.padStart(2, "0");
  const day   = `${match[2]}`.padStart(2, "0");
  return { roomType: room, date: `${year}-${month}-${day}` };
}

// 模糊房型
function fuzzyMatchRoom(text) {
  const rooms = [
    { keys: ["女生", "女背包"], value: "女生背包房" },
    { keys: ["男生", "男背包"], value: "男生背包房" },
    { keys: ["悠活", "家庭", "高級四人"], value: "悠活四人房" },
    { keys: ["四人"], value: "經濟四人房" },
    { keys: ["雙床"], value: "標準雙床房" },
    { keys: ["景觀", "市景", "view"], value: "景觀雙人房" },
    { keys: ["雙人", "情人", "2人"], value: "標準雙人房" }
  ];
  for (const r of rooms) if (r.keys.some(k => text.includes(k))) return r.value;
  console.warn("⚠️ 無法辨識房型，輸入為：", text);
  return null;
}

// D1 查價
async function fetchPriceFromD1(db, roomType, date) {
  const stmt = db.prepare("SELECT weekday_price AS 平日, friday_price AS 週五, saturday_price AS 週六 FROM room_prices WHERE room_type = ?");
  const result = await stmt.bind(roomType).first();
  if (!result) return null;
  const weekday = new Date(date).getDay();
  if (weekday === 5) return result["週五"];
  if (weekday === 6) return result["週六"];
  return result["平日"];
}

// GPT：在地資訊
async function callGPTForLocalInfo(message, apiKey) {
  if (!apiKey) return null;

  const system = `你是台中中區「快樂腳旅棧」的櫃台人員。
【回答規則】
- 只用陳述句，嚴禁提出任何問題或反問。
- 只提供台中市中區與本館步行可及（約 1 公里內）的資訊；若超出範圍，請改為提供中區內替代選項。
- 優先提供具體名稱、地址、步行時間/路線、營業時間。
- 內容最多 5 行，條列清楚，勿寒暄。
- 禁止使用「你喜歡…?」「要不要…?」「想不想…?」「是否」「方便告訴我…」等語句。
- 若資訊不足，也請直接給 2–4 個可行選項（仍以陳述句呈現，不要要對方選擇）。`;

  const user = `使用者提問（中文）：${message}
已知地標：台中市中區中華路一段185號（快樂腳旅棧）
請依【回答規則】產出最終答案。`;

  const body = {
    model: "gpt-4o",
    temperature: 0.4,
    messages: [
      { role: "system", content: system },
      { role: "user", content: user }
    ],
  };

  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort("timeout"), 8000);

  try {
    const res = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${apiKey}` },
      body: JSON.stringify(body),
      signal: controller.signal,
    });
    const textBody = await res.text();
    if (!res.ok) return null;
    let data; try { data = JSON.parse(textBody); } catch { return null; }
    const out = data?.choices?.[0]?.message?.content?.trim() || "";
    return sanitizeNoQuestions(out);
  } catch {
    return null;
  } finally {
    clearTimeout(timer);
  }
}

// 讀取旅館 facts（hotelInfoKV → key: "hotel_info"）
async function getHotelFacts(env) {
  try {
    const facts = await env.hotelInfoKV.get("hotel_info", "json");
    return facts || {};
  } catch (e) {
    console.error("讀取 hotelInfoKV 失敗：", e);
    return {};
  }
}

// 用 KV 組「館內設施介紹」
function renderFacilityIntro(facts = {}) {
  const name = facts.name || "本館";
  const lines = [];

  if (Array.isArray(facts.facilities) && facts.facilities.length) {
    lines.push(`🏨 ${name}館內設施：${facts.facilities.join("、")}`);
  }
  if (facts.wifi?.ssid) {
    const pw = facts.wifi.password ? `，密碼 ${facts.wifi.password}` : "";
    lines.push(`📶 Wi‑Fi：SSID ${facts.wifi.ssid}${pw}`);
  }
  if (facts.frontDesk) lines.push(`🕒 櫃台：${facts.frontDesk}`);
  if (facts.checkIn || facts.checkOut) lines.push(`⏰ 入住 ${facts.checkIn || "—"}／退房 ${facts.checkOut || "—"}`);
  if (facts.luggage) {
    const a = facts.luggage.earlyCheckInStorage ? `抵達可寄放` : "";
    const b = facts.luggage.afterCheckOutStorage ? `退房後寄放須當日取回` : "";
    if (a || b) lines.push(`🧳 行李：${[a, b].filter(Boolean).join("；")}`);
  }
  if (facts.parking) {
    const before = facts.parking.before_0831 ? "8/31 前提供第二市場免費停車" : "";
    const after = facts.parking.after_0901 ? "9/1 起改補助制（每房每日上限100元、紙本單據、1公里內）" : "";
    const t = [before, after].filter(Boolean).join("；");
    if (t) lines.push(`🅿️ 停車：${t}`);
  }
  if (Array.isArray(facts.notes) && facts.notes.length) {
    lines.push(`ℹ️ 注意事項：${facts.notes.join("、")}`);
  }
  const nf = facts.nonFacilities || {};
  if (nf.laundry?.available === false) {
    const l = nf.laundry.alternatives || {};
    const laundromat = l.laundromat ? `投幣式洗衣店：${l.laundromat.address}（${l.laundromat.description}）` : "";
    const svc = l.laundryService ? `代為送洗：${l.laundryService.price}，${l.laundryService.time}` : "";
    const seg = [laundromat, svc].filter(Boolean).join("；");
    if (seg) lines.push(`🧼 洗衣：館內無洗衣設備；${seg}`);
  }
  if (nf.bikeRental?.available === false) {
    const yb = nf.bikeRental.alternatives?.youBike;
    const locs = Array.isArray(yb?.locations) ? `（附近站點：${yb.locations.join("、")}）` : "";
    lines.push(`🚲 腳踏車：館內無出租，可使用 YouBike${locs}`);
  }

  const out = lines.join("\n").trim();
  return out || `${name}目前提供交誼廳、Wi‑Fi（如需帳密請洽櫃檯）、飲水機／咖啡機與高樓層市景。入住 ${facts.checkIn || "15:00"}、退房 ${facts.checkOut || "11:00"}。其他細節歡迎再問我～`;
}

function sanitizeNoQuestions(text) {
  if (!text) return "";
  // 拆成行，過濾掉看起來像問句或引導偏好的句子
  const banWords = /(喜歡|偏好|要不要|是否|想不想|方便告訴我|可否提供|你介意|你比較想|需要嗎)/;
  return String(text)
    .split(/\n+/)
    .map(s => s.trim())
    .filter(s => s && !/[?？]$/.test(s) && !banWords.test(s))
    .join("\n")
    .trim();
}
// GPT：一般問題
async function callGPTGeneral(message, apiKey) {
  if (!apiKey) return null;
  const system = `你是台中「快樂腳旅棧」的櫃台助理。
【回答規則】
- 只回覆與旅棧/入住/在地資訊直接相關的內容；無關就婉拒並建議聯絡櫃台。
- 用陳述句，不要提問或引導偏好，不要要求對方提供更多資訊。
- 最多 5 行，條列清楚，避免寒暄。`;

  const body = {
    model: "gpt-4o",
    temperature: 0.3,
    messages: [
      { role: "system", content: system },
      { role: "user", content: message }
    ],
    max_tokens: 300
  };

  const res = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: { "Content-Type": "application/json", Authorization: `Bearer ${apiKey}` },
    body: JSON.stringify(body)
  });
  const raw = await res.text();
  if (!res.ok) return null;
  let data; try { data = JSON.parse(raw); } catch { return null; }
  const out = data.choices?.[0]?.message?.content?.trim() || "";
  return sanitizeNoQuestions(out);
}
// 解析使用者文字中的起點與模式
function parseTravelQuery(msg) {
  const mWalk = /(走路|步行)/i.test(msg);
  const mDrive = /(開車|自行開車|騎車|機車|汽車)/i.test(msg);
  const mTransit = /(公車|客運|轉乘|大眾運輸|搭車)/i.test(msg);
  const mode = mWalk ? "walk" : mDrive ? "drive" : mTransit ? "walk" : "walk"; // OSRM demo 無 transit，轉為步行

  // 從「從X到/去/前往」抽取起點；預設用「台中火車站」
  let origin = "台中火車站";
  const m1 = msg.match(/從(.+?)(?:到|去|前往)/);
  if (m1 && m1[1]) origin = m1[1].trim();
  origin = origin.replace(/快樂腳旅棧|中華路一段185號/gi, "").trim() || "台中火車站";
  return { origin, mode };
}

// 在 KV 固定路線中尋找匹配（aliases + 模式）
function pickRouteFromKV(origin, mode, facts) {
  if (!facts || !Array.isArray(facts.common)) return null;
  const o = (origin || "").toLowerCase();
  const m = (mode || "walk").toLowerCase();
  for (const item of facts.common) {
    const aliases = (item.aliases || []).map(x => String(x).toLowerCase());
    const modes = (item.modes || []).map(x => String(x).toLowerCase());
    const aliasHit = aliases.some(a => o.includes(a));
    const modeOk = !modes.length || modes.includes(m);
    if (aliasHit && modeOk && item.text) return item.text;
  }
  return null;
}

// Nominatim 地理編碼（免費，記得帶 User-Agent）
async function geocodeNominatim(query) {
  const url = `https://nominatim.openstreetmap.org/search?format=jsonv2&limit=1&addressdetails=1&countrycodes=tw&q=${encodeURIComponent(query)}`;
  const res = await fetch(url, {
    headers: {
      // 請換成你的信箱，符合 Nominatim 使用規範
      "User-Agent": "HappyInnBot/1.0 (contact: your-email@example.com)"
    }
  });
  if (!res.ok) return null;
  const data = await res.json().catch(() => null);
  if (!Array.isArray(data) || !data.length) return null;
  const p = data[0];
  return { lat: Number(p.lat), lon: Number(p.lon), display_name: p.display_name || "" };
}

// OSRM 公開路由（demo 服務：router.project-osrm.org）
// mode: "walk"|"drive" 會轉為 foot/car
async function osrmRoute({ origin, dest, mode, hotelName = "目的地", hotelAddr = "" }) {
  const profile = mode === "drive" ? "car" : "foot"; // demo 支援 foot/ car / bike
  const coords = `${origin.lon},${origin.lat};${dest.lon},${dest.lat}`;
  const url = `https://router.project-osrm.org/route/v1/${profile}/${coords}?overview=false&steps=true&alternatives=false&annotations=false`;

  const res = await fetch(url, { headers: { "User-Agent": "HappyInnBot/1.0 (contact: your-email@example.com)" }});
  if (!res.ok) return null;
  const data = await res.json().catch(() => null);
  if (!data || data.code !== "Ok" || !Array.isArray(data.routes) || !data.routes.length) return null;

  const route = data.routes[0];
  const leg = route.legs && route.legs[0];
  if (!leg) return null;

  const durMin = Math.round((route.duration || 0) / 60);
  const distKm = (route.distance || 0) / 1000;

  const steps = Array.isArray(leg.steps) ? leg.steps.slice(0, 6) : [];
  const stepLines = steps.map((s, i) => {
    // 將 OSRM 指示簡化成中文：道路名稱 + 方向
    const road = (s.name || "").trim();
    const maneuver = s.maneuver && s.maneuver.type ? s.maneuver.type : "";
    const mod = s.maneuver && s.maneuver.modifier ? s.maneuver.modifier : "";
    const act = formatManeuver(maneuver, mod);
    return `${i + 1}. ${act}${road ? `（${road}）` : ""}`;
  });

  const headerIcon = profile === "foot" ? "🚶" : "🚗";
  const header = `${headerIcon} 路線：約 ${durMin} 分鐘（約 ${distKm.toFixed(1)} 公里）\n目的地：${hotelName}（${hotelAddr}）`;
  return [header, ...stepLines].join("\n");
}

// 將 OSRM 的 maneuver 簡化成中文動詞
function formatManeuver(type, mod) {
  const dir = { left:"向左", right:"向右", straight:"直行", slight_left:"微靠左", slight_right:"微靠右", uturn:"迴轉" }[mod] || "";
  const act =
    type === "depart" ? "出發直行" :
    type === "arrive" ? "抵達目的地" :
    type === "turn" ? (dir || "轉彎") :
    type === "roundabout" ? "進入環島" :
    type === "fork" ? "於岔路口" :
    type === "merge" ? "匯入主線" :
    "前行";
  return `${act}${dir && type !== "turn" ? `（${dir}）` : ""}`;
}

// ========== LLM 意圖分類器 ==========
function safePick(obj, key, fallback = null) {
  return (obj && typeof obj === "object" && key in obj) ? obj[key] : fallback;
}
function normalizeDateToYYYYMMDD(input) {
  if (!input || typeof input !== "string") return null;
  const m = input.match(/(\d{1,2})[\/\-月](\d{1,2})/);
  if (!m) return null;
  const y = new Date().getFullYear();
  const mm = String(m[1]).padStart(2, "0");
  const dd = String(m[2]).padStart(2, "0");
  return `${y}-${mm}-${dd}`;
}
async function classifyIntent(message, apiKey) {
  const system = `你是一個嚴謹的意圖分類器，只輸出 JSON。
可能的意圖(type)： "PRICE_TODAY"|"PRICE_DATED"|"PHOTOS"|"LOCAL_INFO"|"GENERIC_QA"|"UNKNOWN"
回傳：{"type":string,"roomType":string|null,"date":string|null}
date 必須 YYYY-MM-DD；不確定就填 null。`;
  const fewshot = [
    { role:"user", content:"今天雙人房多少" },
    { role:"assistant", content:'{"type":"PRICE_TODAY","roomType":"標準雙人房","date":null}' },
    { role:"user", content:"7/20 四人房價錢" },
    { role:"assistant", content:'{"type":"PRICE_DATED","roomType":"經濟四人房","date":"2025-07-20"}' },
    { role:"user", content:"想看景觀房照片" },
    { role:"assistant", content:'{"type":"PHOTOS","roomType":"景觀雙人房","date":null}' },
    { role:"user", content:"快樂腳旅棧附近有推薦的景點嗎？走路就到的那種" },
    { role:"assistant", content:'{"type":"LOCAL_INFO","roomType":null,"date":null}' }
  ];
  try {
    const res = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${apiKey}` },
      body: JSON.stringify({
        model: "gpt-4o-mini",
        temperature: 0,
        response_format: { type: "json_object" },
        messages: [{role:"system", content:system}, ...fewshot, {role:"user", content:message}],
      })
    });
    const raw = await res.text();
    if (!res.ok) return { type:"UNKNOWN", roomType:null, date:null };
    let outer; try { outer = JSON.parse(raw); } catch { outer = {}; }
    let content = safePick(safePick(outer, "choices", [])[0]?.message, "content", "{}");
    let obj; try { obj = JSON.parse(content); } catch { obj = {}; }
    let type = String(obj.type || "").toUpperCase();
    const allowed = ["PRICE_TODAY","PRICE_DATED","PHOTOS","LOCAL_INFO","GENERIC_QA","UNKNOWN"];
    if (!allowed.includes(type)) type = "UNKNOWN";
    let roomType = obj.roomType ?? null;
    let date = obj.date ?? null;
    if (date && /^\d{1,2}[\/\-月]\d{1,2}$/.test(date)) date = normalizeDateToYYYYMMDD(date);
    if (type === "PRICE_TODAY") date = null;
    return { type, roomType, date };
  } catch {
    return { type:"UNKNOWN", roomType:null, date:null };
  }
}
