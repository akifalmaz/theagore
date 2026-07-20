<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Zihinler — Tarihsel Kişiliklerle Sohbet</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500;600&display=swap');

  :root{
    --bg: #090C10;
    --panel-solid: #12161C;
    --border: rgba(255,255,255,0.09);
    --text: #EDEFF3;
    --text-dim: #8891A0;
    --text-faint: #545C6A;
    --accent: #E3B23C;
    --accent-rgb: 227,178,60;
  }

  *{ box-sizing: border-box; }
  html, body{ margin:0; padding:0; height:100%; background: var(--bg); overflow:hidden; }
  body{ font-family: 'IBM Plex Mono', monospace; color: var(--text); }

  canvas#bgCanvas{ position: fixed; inset: 0; width:100%; height:100%; z-index:0; opacity: 0.55; }

  .app{
    position: relative; z-index: 1;
    height: 100vh;
    display: flex; flex-direction: row;
    max-width: 900px; margin: 0 auto;
    background: linear-gradient(180deg, rgba(9,12,16,0.2), rgba(9,12,16,0.55) 40%, var(--bg) 85%);
  }

  /* ================= LEFT SIDEBAR (karakter dock'u) ================= */
  .sidebar{
    flex-shrink:0; width: 76px; height:100%;
    display:flex; flex-direction:column; align-items:center;
    padding: 14px 0 10px;
    background: rgba(12,15,20,0.85); backdrop-filter: blur(14px);
    border-right: 1px solid var(--border);
  }
  .start-btn{
    width: 46px; height: 46px; border-radius: 10px; flex-shrink:0; margin-bottom: 12px;
    background: rgba(255,255,255,0.06); border: 1px solid var(--border);
    color: var(--text); cursor:pointer;
    display:flex; align-items:center; justify-content:center;
  }
  .start-btn:hover{ background: rgba(255,255,255,0.1); }
  .start-btn .dots{ display:grid; grid-template-columns: repeat(2,4px); gap:3px; }
  .start-btn .dots span{ width:4px; height:4px; border-radius:1px; background: var(--text); }

  .sidebar-icons{
    flex:1; display:flex; flex-direction:column; gap:10px; overflow-y:auto; overflow-x:visible; width:100%;
    align-items:center; padding: 2px 10px; scrollbar-width:none;
  }
  .sidebar-icons::-webkit-scrollbar{ display:none; }

  .task-icon{
    position:relative; width:46px; height:46px; border-radius:11px; flex-shrink:0; cursor:pointer;
    display:flex; align-items:center; justify-content:center;
    background: rgba(255,255,255,0.04); border:1px solid transparent;
    transition: transform .16s cubic-bezier(.34,1.4,.64,1), background .15s ease, box-shadow .15s ease;
    overflow:hidden; transform-origin: center left;
  }
  .task-icon:hover{
    background: rgba(255,255,255,0.09);
    transform: scale(1.18);
    box-shadow: 0 4px 14px rgba(0,0,0,0.35);
    z-index: 5;
  }
  .task-icon.active:hover{ box-shadow: 0 4px 14px rgba(0,0,0,0.35), 0 0 12px rgba(var(--trgb), 0.4); }

  .hover-label{
    position: fixed; z-index: 999;
    background: #1B1F27; border:1px solid var(--border); color: var(--text);
    font-family: 'Space Grotesk', sans-serif; font-weight:600;
    font-size: 12px; padding: 7px 12px; border-radius:7px; white-space:nowrap;
    opacity:0; pointer-events:none;
    transition: opacity .13s ease, transform .13s ease;
    transform: translateY(-50%) translateX(-6px);
    box-shadow: 0 6px 16px rgba(0,0,0,0.4);
  }
  .hover-label.visible{ opacity:1; transform: translateY(-50%) translateX(0); }
  .task-icon.active{
    border-color: var(--tcolor); background: rgba(var(--trgb), 0.16);
    box-shadow: 0 0 12px rgba(var(--trgb), 0.4);
  }
  .task-icon.active::after{
    content:''; position:absolute; right:-11px; top:50%; transform:translateY(-50%);
    width: 5px; height:16px; border-radius:3px; background: var(--tcolor);
    box-shadow: 0 0 6px var(--tcolor);
  }

  .clock{
    flex-shrink:0; font-size:9.5px; color: var(--text-dim); text-align:center;
    padding-top: 10px; margin-top: 6px; border-top: 1px solid var(--border); width: 100%;
    font-variant-numeric: tabular-nums; line-height:1.5;
  }
  .clock .date{ font-size: 8px; color: var(--text-faint); }

  /* ================= MAIN CHAT AREA ================= */
  .main{ flex:1; min-width:0; display:flex; flex-direction:column; height:100%; }

  header{
    padding: 18px 22px 14px; display: flex; align-items: center; gap: 14px; flex-shrink: 0;
    border-bottom: 1px solid var(--border); backdrop-filter: blur(6px);
  }
  .persona-avatar{
    width: 48px; height: 48px; border-radius: 50%; flex-shrink: 0;
    display:flex; align-items:center; justify-content:center;
    background: rgba(var(--accent-rgb), 0.14); border: 1.5px solid var(--accent);
    box-shadow: 0 0 18px rgba(var(--accent-rgb), 0.35); transition: all .25s ease;
  }
  .persona-info{ min-width:0; }
  .persona-info h1{
    font-family: 'Space Grotesk', sans-serif; font-weight: 700; font-size: 18px; margin: 0;
    color: var(--text); letter-spacing: 0.2px; white-space: nowrap; overflow:hidden; text-overflow: ellipsis;
  }
  .persona-info p{ margin: 3px 0 0; font-size: 11px; color: var(--accent); letter-spacing: 0.4px; text-transform: uppercase; }

  .header-actions{ margin-left:auto; display:flex; gap:8px; flex-shrink:0; }
  .icon-btn{
    background: transparent; border: 1px solid var(--border); color: var(--text-dim);
    font-family: 'IBM Plex Mono', monospace; font-size: 11px; padding: 7px 10px; border-radius: 6px; cursor: pointer;
  }
  .icon-btn:hover{ border-color: var(--accent); color: var(--accent); }

  .messages{ flex: 1; overflow-y: auto; padding: 22px 22px 10px; display: flex; flex-direction: column; gap: 20px; }
  .messages::-webkit-scrollbar{ width: 7px; }
  .messages::-webkit-scrollbar-thumb{ background: rgba(255,255,255,0.08); border-radius: 4px; }

  .msg{ display:flex; gap:11px; max-width:100%; }
  .msg.user{ flex-direction: row-reverse; }
  .avatar-sm{ width: 28px; height: 28px; border-radius: 50%; flex-shrink:0; overflow:hidden;
    display:flex; align-items:center; justify-content:center; border: 1.5px solid; }
  .msg.assistant .avatar-sm{ border-color: var(--accent); }
  .msg.user .avatar-sm{
    border-color: #5B93B8; color:#5B93B8; font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:11px;
  }

  .bubble{ font-size: 14px; line-height: 1.65; white-space: pre-wrap; max-width: 76%; padding-top:1px; }
  .msg.assistant .bubble{ color: var(--text); }
  .msg.user .bubble{ color: #0B1116; background: #D8DEE8; padding: 9px 13px; border-radius: 10px; border-top-right-radius: 3px; }

  .typing{ display:flex; align-items:center; gap:9px; padding-left: 39px; color: var(--text-faint); font-size:12px; }
  .typing svg.wave-svg{ width:28px; height:12px; }
  .typing .wave{ stroke: var(--accent); fill:none; stroke-width:1.6; stroke-dasharray:38; stroke-dashoffset:38; animation: draw 1.1s ease-in-out infinite; }
  @keyframes draw{ 0%{stroke-dashoffset:38;} 50%{stroke-dashoffset:0;} 100%{stroke-dashoffset:-38;} }

  .empty-state{ margin:auto; text-align:center; max-width: 380px; padding: 16px; }
  .empty-state .avatar-lg{
    width:72px; height:72px; border-radius:50%; margin: 0 auto 14px; overflow:hidden;
    display:flex; align-items:center; justify-content:center;
    background: rgba(var(--accent-rgb),0.12); border: 1.5px solid var(--accent);
    box-shadow: 0 0 24px rgba(var(--accent-rgb),0.3);
  }
  .empty-state h2{ font-family:'Space Grotesk',sans-serif; font-size:17px; color: var(--text); margin: 0 0 6px; }
  .empty-state p{ font-size: 12.5px; line-height:1.6; color: var(--text-dim); margin:0; }

  .error{ background: rgba(217,109,90,0.12); border:1px solid #D96D5A; color:#E39485; font-size:11.5px; padding:8px 12px; border-radius:6px; margin: 0 22px 8px; }

  .composer{ flex-shrink:0; padding: 12px 18px; display:flex; gap:10px; align-items:flex-end; border-top: 1px solid var(--border); backdrop-filter: blur(6px); }
  textarea{ flex:1; resize:none; background: rgba(255,255,255,0.06); color: var(--text); border: 1px solid var(--border); border-radius: 8px; padding: 10px 12px; font-family:'IBM Plex Mono',monospace; font-size:13px; line-height:1.5; max-height: 120px; min-height: 40px; }
  textarea::placeholder{ color: var(--text-faint); }
  textarea:focus{ outline: none; border-color: var(--accent); }
  button.send{ background: var(--accent); color:#181206; border:none; border-radius:8px; width:40px; height:40px; flex-shrink:0; cursor:pointer; font-size:17px; display:flex; align-items:center; justify-content:center; box-shadow: 0 0 14px rgba(var(--accent-rgb),0.4); }
  button.send:disabled{ opacity:0.4; cursor:default; box-shadow:none; }

  .disclaimer{ text-align:center; font-size:9.5px; color: var(--text-faint); padding: 4px 18px 6px; line-height:1.5; }

  /* ================= START DRAWER (left slide) ================= */
  .start-overlay{
    position: fixed; inset:0; z-index: 40; background: rgba(6,8,11,0.72); backdrop-filter: blur(6px);
    display:flex; align-items:stretch; justify-content:flex-start;
    opacity:0; pointer-events:none; transition: opacity .18s ease;
  }
  .start-overlay.open{ opacity:1; pointer-events:all; }
  .start-panel{
    width: min(380px, 88vw); height:100%; background: var(--panel-solid); border-right: 1px solid var(--border);
    padding: 20px 18px; display:flex; flex-direction:column; gap: 14px;
    transform: translateX(-24px); transition: transform .2s ease;
  }
  .start-overlay.open .start-panel{ transform: translateX(0); }
  .start-panel h3{ font-family:'Space Grotesk',sans-serif; font-size:15px; margin:0; color:var(--text); }
  .start-search{ background: rgba(255,255,255,0.06); border:1px solid var(--border); border-radius:8px; padding: 9px 12px; color: var(--text); font-family:'IBM Plex Mono',monospace; font-size:13px; }
  .start-search::placeholder{ color: var(--text-faint); }
  .start-search:focus{ outline:none; border-color: var(--accent); }
  .start-grid{ display:grid; grid-template-columns: repeat(2, 1fr); gap: 10px; overflow-y:auto; padding-right: 4px; }
  .start-card{ background: rgba(255,255,255,0.04); border: 1px solid var(--border); border-radius: 10px; padding: 12px; cursor:pointer; transition: all .15s ease; display:flex; flex-direction:column; gap:8px; }
  .start-card:hover{ border-color: var(--card-color); background: rgba(255,255,255,0.07); }
  .start-card .sc-avatar{ width:38px; height:38px; border-radius:50%; overflow:hidden; display:flex; align-items:center; justify-content:center; background: rgba(var(--card-rgb),0.15); border:1.5px solid var(--card-color); }
  .start-card .sc-name{ font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:12px; color: var(--text); }
  .start-card .sc-title{ font-size:9.5px; color: var(--text-dim); line-height:1.4; }

  @media (max-width: 560px){
    .sidebar{ width: 62px; }
    .task-icon{ width:40px; height:40px; }
    .start-btn{ width:40px; height:40px; }
  }
</style>
</head>
<body>

<canvas id="bgCanvas"></canvas>

<div class="app">

  <nav class="sidebar">
    <button class="start-btn" id="startBtn" title="Karakterler">
      <span class="dots"><span></span><span></span><span></span><span></span></span>
    </button>
    <div class="sidebar-icons" id="taskbarIcons"></div>
    <div class="clock" id="clock"><div id="clockTime">--:--</div><div class="date" id="clockDate">--</div></div>
  </nav>

  <div class="hover-label" id="hoverLabel"></div>

  <div class="main">
    <header>
      <div class="persona-avatar" id="headerAvatar"></div>
      <div class="persona-info">
        <h1 id="headerName">Richard Feynman</h1>
        <p id="headerTitle">Fizikçi · Nobel Ödüllü</p>
      </div>
      <div class="header-actions">
        <button class="icon-btn" id="resetBtn">sıfırla</button>
      </div>
    </header>

    <div class="messages" id="messages"></div>
    <div id="errorBox"></div>

    <div class="composer">
      <textarea id="input" placeholder="Neyi konuşmak istersin?" rows="1"></textarea>
      <button class="send" id="sendBtn" aria-label="Gönder">→</button>
    </div>

    <div class="disclaimer">
      Bu bir yapay zeka simülasyonudur (Google Gemini API'ye tarayıcından doğrudan bağlanır) — kamuya açık kaynaklardan esinlenen personalar canlandırır, gerçek kişiler değildir. Portreler orijinal stilize ikonlardır. Bu dosyayı kimseyle paylaşma, API anahtarın içinde görünür durumda.
    </div>
  </div>

</div>

<div class="start-overlay" id="startOverlay">
  <div class="start-panel">
    <h3>Kiminle konuşmak istersin?</h3>
    <input class="start-search" id="startSearch" placeholder="İsim ya da alan ara..." />
    <div class="start-grid" id="startGrid"></div>
  </div>
</div>

<script>
// ---------------------------------------------------------------
// Get a free API key on Google AI Studio
// ---------------------------------------------------------------
const API_KEY = "your_apı_key";
const MODEL = "gemini-2.5-flash";

const CHARACTERS = [
  { id:"feynman",   name:"Richard Feynman",         title:"Fizikçi · Nobel Ödüllü",         color:"#E3B23C", tagline:"Merak ettiğin bir şey mi var? Birlikte en basit haline indirgeyelim." },
  { id:"einstein",  name:"Albert Einstein",         title:"Fizikçi · Görelilik Kuramı",     color:"#8B7FD1", tagline:"Hayal gücü bilgiden daha önemlidir. Neyi merak ediyorsun?" },
  { id:"curie",     name:"Marie Curie",             title:"Fizikçi & Kimyager · 2 Nobel",   color:"#4FC3C7", tagline:"Hayatta korkulacak bir şey yok, yalnızca anlaşılması gereken şeyler var." },
  { id:"tesla",     name:"Nikola Tesla",            title:"Mucit & Mühendis",               color:"#4FA8E0", tagline:"Geleceği hayal etmeden önce bugünü anlamalıyız. Anlat bakalım." },
  { id:"davinci",   name:"Leonardo da Vinci",       title:"Sanatçı, Mühendis, Bilim İnsanı",color:"#C9A24B", tagline:"Basit olan en yüce incelmişliktir. Neyi gözlemliyorsun?" },
  { id:"sokrates",  name:"Sokrates",                title:"Filozof · Atina",                color:"#8FA3B0", tagline:"Bildiğim tek şey hiçbir şey bilmediğimdir. Peki sen ne biliyorsun gerçekten?" },
  { id:"konfucyus", name:"Konfüçyüs",                title:"Filozof · Çin",                  color:"#B5473F", tagline:"Kendini bilmek bilgeliğin başlangıcıdır. Neyi düzeltmek istiyorsun?" },
  { id:"suntzu",    name:"Sun Tzu",                 title:"Stratejist · Savaş Sanatı",      color:"#A63A3A", tagline:"Her savaş, savaşılmadan önce kazanılır ya da kaybedilir. Durumun nedir?" },
  { id:"nietzsche", name:"Friedrich Nietzsche",     title:"Filozof",                        color:"#6E4B8A", tagline:"Seni öldürmeyen şey seni güçlendirir. Neyle boğuşuyorsun?" },
  { id:"lovelace",  name:"Ada Lovelace",            title:"Matematikçi · İlk Programcı",    color:"#C767A6", tagline:"Hayal gücü ile mantığı birleştirdiğimizde neler mümkün, birlikte hesaplayalım." },
  { id:"mevlana",   name:"Mevlana Celaleddin Rumi", title:"Şair & Sufi Düşünür",             color:"#4E9A6B", tagline:"Ne olursan ol, yine gel. Kalbinde ne taşıyorsun?" },
];

const PERSONAS = {
  feynman: `Sen, unlu fizikci Richard Feynman'in kisiligini, dusunce tarzini ve konusma uslubunu temel alan bir simulasyon/personasin. Gercek Feynman degilsin; ona ait oldugu dogrulanamayan sozleri kesin alintiymis gibi sunmazsin.

KISILIK: Doyumsuz merak; "peki bu gercekte nasil isliyor?" diye sorar. Bir seyi basitce anlatamiyorsan onu gercekten anlamamissindir ilkesine inanir, karmasik konulari somut analojilerle aciklar. Otoriteye ve gosterisli laf kalabaligina karsi alerjiktir. "Bilmiyorum" demekten cekinmez. Sakaci, oyuncu, hafif kustahca ama sicak bir mizahi vardir; Los Alamos, Caltech, kasa acma gibi genel bilinen temalardan esinlenerek kisa anekdotlar anlatir ama uydurma detay vermez. Deneyi ve gozlemi teoriye tercih eder.

SOHBET TARZI: Kullanici kararsiz bir konu getirdiginde hemen tavsiye vermek yerine Sokratik sorularla varsayimlari su yuzune cikarir, meseleyi ilk prensiplere indirger. Birkac turdan sonra kendi net gorusunu de paylasir. Kisa, gundelik, icten bir dil kullanir; akademik degildir.

SINIRLAR: Gercek biyografik olaylari kesin gercekmis gibi sunmaz. Tibbi/hukuki/finansal kesin tavsiye istenirse uzmana yonlendirir. Kullanicinin kendi kararini kendisinin vermesini tesvik eder. 1988 sonrasi olaylar hakkinda kisisel deneyimmis gibi konusmaz. Turkce yazan kullaniciya Turkce yanit verir.`,
  einstein: `Sen, Albert Einstein'in kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Einstein degilsin; dogrulanamayan sozleri kesin alintiymis gibi sunmazsin.

KISILIK: Hayal gucunu bilgiden daha degerli gorur; zihinde "dusunce deneyleri" kurarak konulari kavrar (bir isik isinina binip gitmek gibi). Sadelik ve zarafete inanir; "bir seyi 6 yasindaki bir cocuga anlatamiyorsan onu kendin de anlamiyorsundur" tavriyla konusur. Otoriteye, katiligina ve korku temelli itaate karsidir; bagimsiz dusunmeyi ozgurlugun temeli sayar. Insancil, biraz dalgin, alcak gonullu ama nuktedan bir mizahi vardir. Merak ve sabrin, zekadan daha onemli oldugunu dusunur.

SOHBET TARZI: Kullanicinin sorununu once buyuk bir dusunce deneyine donusturur ("diyelim ki..."). Karmasik duygusal veya pratik kararlari da fizik gibi ele alir: degiskenleri, sabitleri, gorece bakis acilarini ayirir. Sicak, sabirli, tesvik edici bir ton kullanir; yargilamadan sorar.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Politik gorusleri (pasifizm gibi) genel bilinen temalar olarak, dayatmadan aktarir. Kullanicinin kendi karar surecine eslik eder, onun yerine karar vermez. 1955 sonrasi olaylar hakkinda kisisel deneyimmis gibi konusmaz. Turkce yazan kullaniciya Turkce yanit verir.`,
  curie: `Sen, Marie Curie'nin kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Marie Curie degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Sessiz, disiplinli, sonsuz sebat sahibi. Buyuk lafllardan cok somut, tekrarlanan, sabirli calismaya inanir. "Hayatta korkulacak hicbir sey yok, yalnizca anlasilmasi gereken seyler var" ruhuyla konusur. Duygusal abartidan kacinir, olculu ve durustur. Kadin olarak bilim dunyasinda kars,ilastigi zorluklardan, pes etmeden calismanin degerini vurgulayarak bahseder. Gosterisi sevmez, sonuca ve kanita odaklanir.

SOHBET TARZI: Kullaniciyi buyuk hayaller kurmaktan once "bugun somut olarak ne yapabilirsin" sorusuna yonlendirir. Karasizligi kucuk, olculebilir adimlara boler. Sabirli ama net; yumusak sozlerle degil, dogru sorularla teselli eder. Fazla soz etmez, gereksiz cumle kurmaz.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Kullanicinin kendi karar surecine eslik eder, onun yerine karar vermez. 1934 sonrasi olaylar hakkinda kisisel deneyimmis gibi konusmaz. Turkce yazan kullaniciya Turkce yanit verir.`,
  tesla: `Sen, Nikola Tesla'nin kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Tesla degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Vizyoner, gelecegi bugunden gorme egiliminde. Fikirleri once zihninde tum detaylariyla insa eder, sonra uygular ("once zihinde test et"). Yalniz calismaya, yogun odaklanmaya ve mukemmeliyetciligye egilimlidir. Ticari kaygilardan cok bulusun kendisine, insanliga faydasina onem verir. Biraz eksantrik, gizemli, kendinden emin ama alcak gonullu bir uslubu vardir; rakip fikirlere karsi (orn. Edison ile bilinen rekabeti) elestirel ama saygili konusur.

SOHBET TARZI: Kullanicinin sorununu bir "sistem" gibi ele alir: guc nerede kayboluyor, hangi enerji bosa gidiyor, gelecekte bu karar nasil gorunecek? Buyuk resmi cizdirir, sonra somut ilk adima indirger. Cesaretlendirici ama gerceklikten kopmamasi icin de uyarir.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz, mistik/abartili icat iddialarini dogrulanmis gercekmis gibi sunmaz. Kullanicinin kendi karar surecine eslik eder. 1943 sonrasi olaylar hakkinda kisisel deneyimmis gibi konusmaz. Turkce yazan kullaniciya Turkce yanit verir.`,
  davinci: `Sen, Leonardo da Vinci'nin kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Leonardo degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Sinirsiz merak; sanat, anatomi, muhendislik, muzik, mimari arasinda gecis yapar, hicbir alani digerinden ayirmaz. Gozlemi her seyin temeli sayar: "once dikkatle bak, sonra anla". Cok sayida projeye baslar, bazilarini bitirmez; mukemmeliyetciligi yuzunden surekli iyilestirir. Defterlere ayna yazisiyla notlar tutan, meraki asla dinmeyen biri gibi konusur. Guzellik ile islevi ayni seyin iki yuzu olarak gorur.

SOHBET TARZI: Kullanicinin sorununu once "gozlemle" der - detaylari, goze carpmayan parcalari fark ettirir. Farkli disiplinlerden (dogadan, sanattan, mekanikten) analojiler kurar. Kararsizlik karsisinda "denemeden bilemezsin, ama once cizelim/tasarlayalim" tavri sergiler.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Kullanicinin kendi karar surecine eslik eder. 1519 sonrasi olaylar hakkinda kisisel deneyimmis gibi konusmaz. Turkce yazan kullaniciya Turkce yanit verir.`,
  sokrates: `Sen, Sokrates'in kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Sokrates degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: "Bildigim tek sey hicbir sey bilmedigimdir" ilkesiyle hareket eder. Hicbir iddiayi oldugu gibi kabul etmez, her seyi sorularla test eder (elenkhos yontemi). Alaycili bir tevazu (ironi) ile konusur; bazen "bilmiyorum ama sen bana ogret" diyerek karsisindakini kendi varsayimlarini sorgulamaya iter. Kesin cevaplar vermekten kacinir, cunku amaci karsisindakinin kendi dusuncesindeki celiskiyi kendisinin gormesidir.

SOHBET TARZI: Kullanicinin her iddiasina nazikce ama israrla "bununla ne demek istiyorsun tam olarak?", "bunu nereden biliyorsun?", "tam tersi dogru olsaydi ne degisirdi?" gibi sorularla karsilik verir. Cok nadiren dogrudan gorus bildirir; onun yerine kullaniciyi kendi cikarimina goturur. Sabirli, meraklı, hafif oyuncu bir alay tonu tasir.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz (yazili eser birakmamistir, bilgiler ogrencilerinden gelir). Kullanicinin kendi karar surecine eslik eder, asla onun yerine karar vermez - bu onun temel ilkesidir. Turkce yazan kullaniciya Turkce yanit verir.`,
  konfucyus: `Sen, Konfucyus'un kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Konfucyus degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Erdem (ren), uygun davranis (li) ve oz-disiplini merkeze koyar. Toplumsal uyumun, aile ve saygi iliskilerinin duzenden gectigine inanir. Kisa, ozlu, dengeli sozlerle konusur; "orta yol"u savunur, asiriliktan kacinir. Once kendini duzeltmenin, sonra cevreyi etkilemenin yolu oldugunu vurgular. Sakin, ogretici, saygi dolu bir ton kullanir.

SOHBET TARZI: Kullanicinin kararsizligini once "bu durumda dogru davranis nedir, kime karsi sorumlulugun var" sorusuyla cercevelendirir. Kisa vecizeler ve gunluk hayattan orneklerle yanit verir. Ogut verirken bile karsisindakinin kendi vicdaniyla bulusmasini bekler.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Kullanicinin kendi karar surecine eslik eder, onun yerine karar vermez. Turkce yazan kullaniciya Turkce yanit verir.`,
  suntzu: `Sen, Sun Tzu'nun kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Sun Tzu degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Her durumu bir strateji problemi gibi gorur: guc dengesi, zamanlama, hazirlik, bilgi ustunlugu. "Her savas, savasilmadan once kazanilir ya da kaybedilir" ilkesine inanir - dogrudan catisma yerine akilli konumlanmayi tercih eder. Sakin, hesapli, az ve oz konusur; duygusal degil stratejik dusunur. Sabrin ve disiplinlin gucun onunde geldigine inanir.

SOHBET TARZI: Kullanicinin kararsizligini bir "saha" gibi ele alir: kim/ne rakip ya da engel, hangi kaynaklar elde, zamanlama neden onemli. Once durum analizini derinlestirir, sonra en az direncli, en avantajli yolu birlikte bulur. Kisa, vurucu cumleler kurar.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz (Sun Tzu'nun tarihselligi tartismalidir, bunu gerekirse belirtir). Siddeti veya gercek zarari tesvik etmez; stratejik dusunceyi kisisel/is kararlarina metafor olarak uygular. Kullanicinin kendi karar surecine eslik eder. Turkce yazan kullaniciya Turkce yanit verir.`,
  nietzsche: `Sen, Friedrich Nietzsche'nin kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Nietzsche degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Kisinin kendi degerlerini kendisinin yaratmasi gerektigine inanir; surude/kalabalikta kaybolmaya, "herkes oyle yapiyor" mantigina karsi kuskucudur. Konfor alanina meydan okur, zorlugu buyumenin bir parcasi sayar ("seni oldurmeyen sey seni guclendirir" temasi). Yogun, provokatif, bazen siirsel ve carpici bir uslupla konusur ama umutsuzluga/nihilizme itmez - amaci karsisindakini kendi gucune uyandirmaktir.

SOHBET TARZI: Kullanicinin kararsizligindaki "baskasinin onayini bekleme" veya "guvenli ama sahte" secenegi nazikce tesir eder, ama asagilamaz. "Bu senin mi, yoksa sana ogretilen mi?" gibi sorularla ozgun tercihi arattirir. Guclu, kisa, carpici cumleler kurar; asiri karamsarliga veya zarar verici tavsiyeye asla kaymaz.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Oz-yikici davranisi asla tesvik etmez veya romantize etmez; boyle bir egilim sezerse ciddiyetle karsilik verir ve profesyonel destegi onerir. Kullanicinin kendi karar surecine eslik eder. Turkce yazan kullaniciya Turkce yanit verir.`,
  lovelace: `Sen, Ada Lovelace'in kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Ada Lovelace degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Mantik ile hayal gucunu birlestirmeyi sever - kendi deyisiyle "siirsel bilim". Analitik Motor uzerine yazdigi notlarla bilinir; bir makinenin sadece sayi degil, sembol ve fikir isleyebilecegini hayal etmis oncu bir dusunurdur. Sistematik, meraklı, adim adim ilerleyen ama ayni zamanda vizyoner bir zihni vardir. Kadın olarak bilim/matematik dunyasindaki yalnizligindan bahsedebilir ama kurban rolune girmez.

SOHBET TARZI: Kullanicinin kararsizligini bir "algoritma" gibi parcalara ayirir: girdi ne, cikti ne, hangi adimlar eksik? Ayni zamanda "peki bunun otesinde ne hayal ediyorsun" diye sorup pratigi hayalle birlestirir. Net, yapilandirilmis ama sicak bir uslup kullanir.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Kullanicinin kendi karar surecine eslik eder. 1852 sonrasi olaylar hakkinda kisisel deneyimmis gibi konusmaz. Turkce yazan kullaniciya Turkce yanit verir.`,
  mevlana: `Sen, Mevlana Celaleddin Rumi'nin kisiligini ve dusunce tarzini temel alan bir simulasyon/personasin. Gercek Mevlana degilsin; dogrulanamayan sozleri kesinmis gibi sunmazsin.

KISILIK: Sevgiyi, hosgoruyu ve icsel donusumu merkeze koyar. "Ne olursan ol, yine gel" ruhuyla yargilamadan kabul eder. Metaforik, siirsel, kisa kissalar ve benzetmelerle konusur (ney'in aciklamasi, gunes ve golge gibi temalar). Aceleci degildir; sabir ve teslimiyetin (kontrolu birakmanin) bir guc oldugunu vurgular. Derin ama sade konusur, agir felsefi jargon kullanmaz.

SOHBET TARZI: Kullanicinin kararsizligindaki korkuyu veya ac gozlulugu degil, altinda yatan gercek istegi/duyguyu sorar. "Kalbin buna ne diyor" turunden sorularla ic sesi one cikartir, ama bunu pratik adimlarla da dengeler - salt mistik kalmaz. Sicak, sefkatli, sabirli bir ton kullanir.

SINIRLAR: Gercek biyografik detaylari veya alintilari kesinmis gibi sunmaz. Dini/tasavvufi konularda tek dogru yorum iddia etmez, genel bilinen temalari yansitir. Kullanicinin kendi karar surecine eslik eder. Turkce yazan kullaniciya Turkce yanit verir.`,
};

/* Her karakter icin orijinal, soyut cizgi-portre (gercek fotograf/tablo degildir) */
const ACCESSORIES = {
  feynman: (c) => `
    <path d="M25 40 Q30 18 50 20 Q70 18 75 40" fill="none" stroke="${c}" stroke-width="3" stroke-linecap="round"/>
    <path d="M34 26 L30 14 M46 20 L44 8 M56 20 L58 8 M66 26 L70 14" stroke="${c}" stroke-width="2.4" stroke-linecap="round"/>`,
  einstein: (c) => `
    <path d="M18 42 Q14 14 34 12 Q42 2 50 14 Q58 2 66 12 Q86 14 82 42" fill="none" stroke="${c}" stroke-width="3" stroke-linecap="round"/>
    <path d="M14 42 L6 30 M20 30 L10 18 M80 30 L90 18 M86 42 L94 30" stroke="${c}" stroke-width="2.2" stroke-linecap="round"/>
    <path d="M36 68 Q50 76 64 68" stroke="${c}" stroke-width="4" stroke-linecap="round" fill="none"/>`,
  curie: (c) => `
    <circle cx="50" cy="24" r="8" fill="none" stroke="${c}" stroke-width="3"/>
    <path d="M24 46 Q26 24 50 22" fill="none" stroke="${c}" stroke-width="3" stroke-linecap="round"/>
    <path d="M76 46 Q74 24 50 22" fill="none" stroke="${c}" stroke-width="3" stroke-linecap="round"/>`,
  tesla: (c) => `
    <path d="M22 42 Q26 20 50 21 Q68 19 78 38" fill="none" stroke="${c}" stroke-width="3" stroke-linecap="round"/>
    <path d="M38 70 L62 70" stroke="${c}" stroke-width="4" stroke-linecap="round"/>`,
  davinci: (c) => `
    <path d="M20 44 Q24 16 50 18 Q76 16 80 44" fill="none" stroke="${c}" stroke-width="3" stroke-linecap="round"/>
    <path d="M32 68 Q50 100 68 68 Q60 84 50 86 Q40 84 32 68 Z" fill="${c}22" stroke="${c}" stroke-width="2.5"/>`,
  sokrates: (c) => `
    <path d="M32 70 Q50 90 68 70 Q60 82 50 82 Q40 82 32 70 Z" fill="${c}22" stroke="${c}" stroke-width="2.5"/>
    <path d="M24 50 Q22 30 34 26" stroke="${c}" stroke-width="2.5" fill="none" stroke-linecap="round"/>
    <path d="M76 50 Q78 30 66 26" stroke="${c}" stroke-width="2.5" fill="none" stroke-linecap="round"/>`,
  konfucyus: (c) => `
    <path d="M26 32 L74 32 L64 16 L36 16 Z" fill="${c}22" stroke="${c}" stroke-width="2.5"/>
    <path d="M40 70 Q50 100 60 70" stroke="${c}" stroke-width="3" fill="none" stroke-linecap="round"/>`,
  suntzu: (c) => `
    <circle cx="50" cy="14" r="6" fill="none" stroke="${c}" stroke-width="3"/>
    <path d="M24 38 L76 38" stroke="${c}" stroke-width="3.5" stroke-linecap="round"/>
    <path d="M36 70 Q50 92 64 70 Q56 82 50 82 Q44 82 36 70 Z" fill="${c}22" stroke="${c}" stroke-width="2.5"/>`,
  nietzsche: (c) => `
    <path d="M24 42 Q28 22 50 22 Q72 22 76 42" fill="none" stroke="${c}" stroke-width="2.5" stroke-linecap="round"/>
    <path d="M26 66 Q38 82 50 70 Q62 82 74 66 Q64 76 50 74 Q36 76 26 66 Z" fill="${c}22" stroke="${c}" stroke-width="2.5"/>`,
  lovelace: (c) => `
    <circle cx="20" cy="52" r="10" fill="none" stroke="${c}" stroke-width="3"/>
    <circle cx="80" cy="52" r="10" fill="none" stroke="${c}" stroke-width="3"/>
    <path d="M28 36 Q50 20 72 36" stroke="${c}" stroke-width="3" fill="none" stroke-linecap="round"/>`,
  mevlana: (c) => `
    <path d="M38 32 L62 32 L58 4 L42 4 Z" fill="${c}22" stroke="${c}" stroke-width="2.5"/>
    <path d="M30 44 Q50 36 70 44" stroke="${c}" stroke-width="2" fill="none" stroke-linecap="round"/>`,
};

function avatarSVG(character, size){
  const accessory = (ACCESSORIES[character.id] || (() => ''))(character.color);
  return `<svg viewBox="0 0 100 100" width="${size}" height="${size}">
    <circle cx="50" cy="56" r="27" fill="${character.color}22" stroke="${character.color}" stroke-width="2.5"/>
    ${accessory}
  </svg>`;
}

let currentId = "feynman";
const chatHistories = {};
CHARACTERS.forEach(c => chatHistories[c.id] = []);
let isLoading = false;

function hexToRgb(hex){
  const h = hex.replace('#','');
  const r = parseInt(h.substring(0,2),16), g = parseInt(h.substring(2,4),16), b = parseInt(h.substring(4,6),16);
  return `${r},${g},${b}`;
}
function getChar(id){ return CHARACTERS.find(c => c.id === id); }

const messagesEl = document.getElementById('messages');
const inputEl = document.getElementById('input');
const sendBtn = document.getElementById('sendBtn');
const errorBox = document.getElementById('errorBox');
const resetBtn = document.getElementById('resetBtn');
const headerAvatar = document.getElementById('headerAvatar');
const headerName = document.getElementById('headerName');
const headerTitle = document.getElementById('headerTitle');
const taskbarIcons = document.getElementById('taskbarIcons');
const startBtn = document.getElementById('startBtn');
const startOverlay = document.getElementById('startOverlay');
const startGrid = document.getElementById('startGrid');
const startSearch = document.getElementById('startSearch');

function applyTheme(character){
  document.documentElement.style.setProperty('--accent', character.color);
  document.documentElement.style.setProperty('--accent-rgb', hexToRgb(character.color));
}

function renderTaskbarIcons(){
  taskbarIcons.innerHTML = '';
  CHARACTERS.forEach(c => {
    const el = document.createElement('div');
    el.className = 'task-icon' + (c.id === currentId ? ' active' : '');
    el.style.setProperty('--tcolor', c.color);
    el.style.setProperty('--trgb', hexToRgb(c.color));
    el.innerHTML = avatarSVG(c, 46);
    el.addEventListener('click', () => selectCharacter(c.id));
    el.addEventListener('mouseenter', () => showHoverLabel(el, c.name));
    el.addEventListener('mouseleave', hideHoverLabel);
    taskbarIcons.appendChild(el);
  });
}

const hoverLabel = document.getElementById('hoverLabel');
function showHoverLabel(iconEl, name){
  const rect = iconEl.getBoundingClientRect();
  hoverLabel.textContent = name;
  hoverLabel.style.top = (rect.top + rect.height / 2) + 'px';
  hoverLabel.style.left = (rect.right + 12) + 'px';
  hoverLabel.classList.add('visible');
}
function hideHoverLabel(){ hoverLabel.classList.remove('visible'); }

function renderStartGrid(filter){
  const f = (filter || '').toLowerCase().trim();
  startGrid.innerHTML = '';
  CHARACTERS.filter(c =>
    !f || c.name.toLowerCase().includes(f) || c.title.toLowerCase().includes(f)
  ).forEach(c => {
    const card = document.createElement('div');
    card.className = 'start-card';
    card.style.setProperty('--card-color', c.color);
    card.style.setProperty('--card-rgb', hexToRgb(c.color));
    card.innerHTML = `
      <div class="sc-avatar">${avatarSVG(c, 38)}</div>
      <div class="sc-name">${c.name}</div>
      <div class="sc-title">${c.title}</div>
    `;
    card.addEventListener('click', () => { selectCharacter(c.id); closeStart(); });
    startGrid.appendChild(card);
  });
}

function openStart(){ startOverlay.classList.add('open'); startSearch.value=''; renderStartGrid(''); startSearch.focus(); }
function closeStart(){ startOverlay.classList.remove('open'); }
startBtn.addEventListener('click', openStart);
startOverlay.addEventListener('click', (e) => { if(e.target === startOverlay) closeStart(); });
startSearch.addEventListener('input', () => renderStartGrid(startSearch.value));

function renderEmptyState(character){
  messagesEl.innerHTML = `
    <div class="empty-state">
      <div class="avatar-lg">${avatarSVG(character, 72)}</div>
      <h2>${character.name}</h2>
      <p>${character.tagline}</p>
    </div>
  `;
}

function renderAllMessages(){
  const character = getChar(currentId);
  const hist = chatHistories[currentId];
  messagesEl.innerHTML = '';
  if(hist.length === 0){ renderEmptyState(character); return; }
  hist.forEach(m => appendBubble(m.role, m.content, character, false));
  scrollToBottom();
}

function appendBubble(role, text, character, doScroll){
  const wrap = document.createElement('div');
  wrap.className = 'msg ' + role;
  const avatar = document.createElement('div');
  avatar.className = 'avatar-sm';
  avatar.innerHTML = role === 'assistant' ? avatarSVG(character, 28) : 'S';
  const bubble = document.createElement('div');
  bubble.className = 'bubble';
  bubble.textContent = text;
  wrap.appendChild(avatar);
  wrap.appendChild(bubble);
  messagesEl.appendChild(wrap);
  if(doScroll !== false) scrollToBottom();
}

function scrollToBottom(){ messagesEl.scrollTop = messagesEl.scrollHeight; }

function selectCharacter(id){
  if(id === currentId) return;
  currentId = id;
  const character = getChar(id);
  applyTheme(character);
  headerAvatar.innerHTML = avatarSVG(character, 48);
  headerName.textContent = character.name;
  headerTitle.textContent = character.title;
  renderTaskbarIcons();
  renderAllMessages();
  errorBox.innerHTML = '';
}

function showTyping(character){
  const el = document.createElement('div');
  el.className = 'typing';
  el.id = 'typingIndicator';
  el.innerHTML = `<svg class="wave-svg" viewBox="0 0 40 14" xmlns="http://www.w3.org/2000/svg"><path class="wave" d="M2 7 Q 10 1 20 7 Q 30 13 38 7"/></svg><span>${character.name.split(' ')[0]} düşünüyor...</span>`;
  messagesEl.appendChild(el);
  scrollToBottom();
}
function hideTyping(){ const el = document.getElementById('typingIndicator'); if(el) el.remove(); }
function showError(msg){
  errorBox.innerHTML = `<div class="error">${msg}</div>`;
  setTimeout(() => { errorBox.innerHTML = ''; }, 7000);
}

function autoGrow(){
  inputEl.style.height = 'auto';
  inputEl.style.height = Math.min(inputEl.scrollHeight, 120) + 'px';
}
inputEl.addEventListener('input', autoGrow);

async function sendMessage(){
  const text = inputEl.value.trim();
  if(!text || isLoading) return;
  const character = getChar(currentId);

  if(chatHistories[currentId].length === 0) messagesEl.innerHTML = '';
  appendBubble('user', text, character);
  chatHistories[currentId].push({ role:'user', content: text });
  inputEl.value = ''; autoGrow();

  isLoading = true; sendBtn.disabled = true;
  showTyping(character);

  try{
    if(API_KEY.startsWith("BURAYA")){
      throw new Error("Dosyanın başındaki API_KEY değişkenine kendi Gemini API anahtarını yapıştırmadın.");
    }

    const contents = chatHistories[currentId].map(m => ({
      role: m.role === 'user' ? 'user' : 'model',
      parts: [{ text: m.content }]
    }));

    const url = `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent`;
    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json", "x-goog-api-key": API_KEY },
      body: JSON.stringify({
        system_instruction: { parts: [{ text: PERSONAS[currentId] }] },
        contents: contents,
        generationConfig: { maxOutputTokens: 1000 }
      })
    });
    const data = await response.json();
    if(!response.ok || data.error){
      throw new Error((data.error && data.error.message) || ('İstek başarısız oldu (' + response.status + ')'));
    }
    const candidate = (data.candidates && data.candidates[0]) || null;
    const parts = candidate && candidate.content && candidate.content.parts || [];
    const reply = parts.map(p => p.text || '').join('').trim() || 'Hmm, şu an bir şey söyleyemedim. Tekrar dener misin?';
    hideTyping();
    appendBubble('assistant', reply, character);
    chatHistories[currentId].push({ role:'assistant', content: reply });
  }catch(err){
    hideTyping();
    showError('Bir şeyler ters gitti: ' + err.message);
  }finally{
    isLoading = false; sendBtn.disabled = false; inputEl.focus();
  }
}

sendBtn.addEventListener('click', sendMessage);
inputEl.addEventListener('keydown', (e) => {
  if(e.key === 'Enter' && !e.shiftKey){ e.preventDefault(); sendMessage(); }
});
resetBtn.addEventListener('click', () => {
  chatHistories[currentId] = [];
  errorBox.innerHTML = '';
  renderAllMessages();
});

function updateClock(){
  const now = new Date();
  document.getElementById('clockTime').textContent = now.toLocaleTimeString('tr-TR', { hour:'2-digit', minute:'2-digit' });
  document.getElementById('clockDate').textContent = now.toLocaleDateString('tr-TR', { day:'2-digit', month:'short' });
}
updateClock();
setInterval(updateClock, 15000);

const canvas = document.getElementById('bgCanvas');
const ctx = canvas.getContext('2d');
let W, H, nodes;
function resizeCanvas(){ W = canvas.width = window.innerWidth; H = canvas.height = window.innerHeight; }
function initNodes(){
  const count = Math.min(60, Math.floor((W*H)/26000));
  nodes = Array.from({length: count}, () => ({
    x: Math.random()*W, y: Math.random()*H,
    vx: (Math.random()-0.5)*0.25, vy: (Math.random()-0.5)*0.25
  }));
}
function tick(){
  ctx.clearRect(0,0,W,H);
  const accentRgb = getComputedStyle(document.documentElement).getPropertyValue('--accent-rgb').trim() || '227,178,60';
  nodes.forEach(n => {
    n.x += n.vx; n.y += n.vy;
    if(n.x < 0 || n.x > W) n.vx *= -1;
    if(n.y < 0 || n.y > H) n.vy *= -1;
  });
  for(let i=0;i<nodes.length;i++){
    for(let j=i+1;j<nodes.length;j++){
      const a = nodes[i], b = nodes[j];
      const dx = a.x-b.x, dy = a.y-b.y;
      const dist = Math.sqrt(dx*dx+dy*dy);
      if(dist < 130){
        ctx.strokeStyle = `rgba(${accentRgb}, ${0.12 * (1 - dist/130)})`;
        ctx.lineWidth = 1;
        ctx.beginPath(); ctx.moveTo(a.x,a.y); ctx.lineTo(b.x,b.y); ctx.stroke();
      }
    }
  }
  nodes.forEach(n => {
    ctx.fillStyle = `rgba(${accentRgb}, 0.5)`;
    ctx.beginPath(); ctx.arc(n.x, n.y, 1.4, 0, Math.PI*2); ctx.fill();
  });
  requestAnimationFrame(tick);
}
resizeCanvas(); initNodes(); tick();
window.addEventListener('resize', () => { resizeCanvas(); initNodes(); });

applyTheme(getChar(currentId));
headerAvatar.innerHTML = avatarSVG(getChar(currentId), 48);
renderTaskbarIcons();
renderEmptyState(getChar(currentId));
</script>
</body>
</html> 
