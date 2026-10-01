<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="theme-color" content="#0D1B2A">
<title>ADVISOR</title>
<style>
  :root {
    --night: #0D1B2A;
    --deep: #14293F;
    --lamp: #F4B942;
    --lamp-soft: #FFD98A;
    --teal: #46B1A8;
    --mist: #9FB3C8;
    --text: #EAF0F6;
  }
  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body { height: 100%; margin: 0; }
  body {
    background: radial-gradient(120% 80% at 50% 28%, var(--deep) 0%, var(--night) 70%);
    color: var(--text);
    font-family: "Noto Sans Arabic", "Segoe UI", system-ui, -apple-system, Roboto, sans-serif;
    display: flex; flex-direction: column;
    height: 100dvh;
    padding: env(safe-area-inset-top) 0 env(safe-area-inset-bottom);
  }
  header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 14px 18px 0;
  }
  header h1 { font-size: 1.05rem; font-weight: 600; letter-spacing: .14em; margin: 0; color: var(--lamp-soft); }
  header nav { display: flex; gap: 6px; }
  .link {
    background: none; border: 0; color: var(--mist); font: inherit; font-size: .9rem;
    padding: 10px 12px; border-radius: 10px; cursor: pointer;
  }
  .link:focus-visible, #orb:focus-visible, button:focus-visible, select:focus-visible, input:focus-visible {
    outline: 2px solid var(--teal); outline-offset: 2px;
  }

  main { flex: 1; display: flex; flex-direction: column; align-items: center; min-height: 0; }
  .stage { padding: 4vh 0 1.5vh; display: grid; place-items: center; }

  /* The orb: the one memorable element */
  #orb {
    position: relative; width: 168px; height: 168px; border: 0; padding: 0; border-radius: 50%;
    background: transparent; cursor: pointer; display: grid; place-items: center;
  }
  #orb .core {
    width: 124px; height: 124px; border-radius: 50%;
    background: radial-gradient(circle at 38% 32%, #FFF1CC 0%, var(--lamp) 55%, #C98A1E 100%);
    box-shadow: 0 0 46px 6px rgba(244,185,66,.38), inset 0 -8px 18px rgba(120,70,0,.35);
    transition: background .4s, box-shadow .4s, transform .3s;
  }
  #orb::before, #orb::after {
    content: ""; position: absolute; inset: 8px; border-radius: 50%;
    border: 2px solid transparent; pointer-events: none;
  }
  #orb[data-state="idle"] .core { animation: breathe 5.5s ease-in-out infinite; }
  #orb[data-state="listening"] .core {
    background: radial-gradient(circle at 38% 32%, #D8FFFA 0%, var(--teal) 58%, #22756F 100%);
    box-shadow: 0 0 50px 8px rgba(70,177,168,.45), inset 0 -8px 18px rgba(0,60,55,.4);
  }
  #orb[data-state="listening"]::before { border-color: rgba(70,177,168,.7); animation: ring 1.6s ease-out infinite; }
  #orb[data-state="listening"]::after  { border-color: rgba(70,177,168,.5); animation: ring 1.6s ease-out .8s infinite; }
  #orb[data-state="thinking"] .core { transform: scale(.88); box-shadow: 0 0 30px 2px rgba(244,185,66,.25), inset 0 -8px 18px rgba(120,70,0,.35); }
  #orb[data-state="thinking"]::before {
    inset: 4px; border: 3px solid transparent; border-top-color: var(--lamp-soft);
    animation: spin 1.1s linear infinite;
  }
  #orb[data-state="speaking"] .core { animation: speak .9s ease-in-out infinite; }
  #orb[data-state="speaking"]::before { border-color: rgba(244,185,66,.55); animation: ring 1.8s ease-out infinite; }

  @keyframes breathe { 0%,100% { transform: scale(1); } 50% { transform: scale(1.045); } }
  @keyframes ring { from { transform: scale(.85); opacity: 1; } to { transform: scale(1.45); opacity: 0; } }
  @keyframes spin { to { transform: rotate(360deg); } }
  @keyframes speak { 0%,100% { transform: scale(1); } 50% { transform: scale(1.09); } }
  @media (prefers-reduced-motion: reduce) {
    #orb .core, #orb::before, #orb::after { animation: none !important; }
  }

  #status { margin: 6px 20px 0; min-height: 1.6em; color: var(--mist); font-size: .95rem; text-align: center; }
  #live { margin: 4px 24px 0; min-height: 1.4em; color: var(--text); text-align: center; font-size: 1rem; }

  #log {
    flex: 1; width: 100%; max-width: 560px; min-height: 0; overflow-y: auto;
    padding: 12px 18px 18px; display: flex; flex-direction: column; gap: 10px;
    mask-image: linear-gradient(to bottom, transparent 0, #000 22px);
  }
  .msg { max-width: 88%; padding: 10px 14px; border-radius: 16px; line-height: 1.5; font-size: 1rem; white-space: pre-wrap; }
  .msg.me { align-self: flex-end; background: var(--deep); border: 1px solid rgba(159,179,200,.18); border-end-end-radius: 4px; }
  .msg.ai { align-self: flex-start; background: rgba(244,185,66,.1); border: 1px solid rgba(244,185,66,.22); border-end-start-radius: 4px; }
  .empty { color: var(--mist); text-align: center; margin: auto; padding: 0 28px; line-height: 1.6; }

  /* Settings sheet */
  #sheet {
    position: fixed; inset: 0; background: rgba(5,12,20,.72); display: none;
    align-items: flex-end; justify-content: center; z-index: 5;
  }
  #sheet.open { display: flex; }
  .panel {
    width: 100%; max-width: 560px; background: var(--deep); border-radius: 20px 20px 0 0;
    padding: 20px 18px calc(20px + env(safe-area-inset-bottom)); max-height: 88dvh; overflow-y: auto;
  }
  .panel h2 { margin: 0 0 14px; font-size: 1.1rem; }
  label { display: block; margin: 14px 0 6px; font-size: .9rem; color: var(--mist); }
  input, select {
    width: 100%; padding: 12px; border-radius: 10px; border: 1px solid rgba(159,179,200,.3);
    background: var(--night); color: var(--text); font: inherit; font-size: 1rem;
  }
  .hint { font-size: .82rem; color: var(--mist); margin: 6px 0 0; line-height: 1.45; }
  .row { display: flex; gap: 10px; margin-top: 20px; }
  .btn { flex: 1; padding: 13px; border-radius: 12px; border: 0; font: inherit; font-weight: 600; cursor: pointer; }
  .btn.primary { background: var(--lamp); color: #2A1C00; }
  .btn.quiet { background: transparent; color: var(--text); border: 1px solid rgba(159,179,200,.35); }
</style>
</head>
<body>
  <header>
    <h1>ADVISOR</h1>
    <nav>
      <button class="link" id="newChat">New chat</button>
      <button class="link" id="openSettings">Settings</button>
    </nav>
  </header>

  <main>
    <div class="stage">
      <button id="orb" data-state="idle" aria-label="Talk to ADVISOR"><span class="core"></span></button>
    </div>
    <div id="status" role="status" aria-live="polite">Tap the orb and start talking</div>
    <div id="live" dir="auto"></div>
    <div id="log" aria-live="polite"></div>
  </main>

  <div id="sheet" role="dialog" aria-modal="true" aria-label="Settings">
    <div class="panel">
      <h2>Settings</h2>

      <label for="lang">I speak</label>
      <select id="lang">
        <option value="en-US">English</option>
        <option value="ar-SA">العربية (Modern Arabic)</option>
        <option value="ar-EG">العربية المصرية (Egyptian Arabic)</option>
        <option value="fr-FR">Français</option>
        <option value="es-ES">Español</option>
        <option value="de-DE">Deutsch</option>
        <option value="tr-TR">Türkçe</option>
        <option value="it-IT">Italiano</option>
        <option value="pt-BR">Português</option>
        <option value="hi-IN">हिन्दी</option>
        <option value="ur-PK">اردو</option>
        <option value="ru-RU">Русский</option>
      </select>
      <p class="hint">This sets what the microphone listens for. ADVISOR answers in the language you speak.</p>

      <label for="key">Gemini API key</label>
      <input id="key" type="password" autocomplete="off" placeholder="AIza...">
      <p class="hint">Get a free key at aistudio.google.com. It is stored only on this phone. Fine for your own use. Never share the key or post it online.</p>

      <label for="proxy">Server address (optional)</label>
      <input id="proxy" type="url" inputmode="url" placeholder="https://your-worker.workers.dev">
      <p class="hint">Leave empty for now. Later, a private server can hold the key so it is never inside the app.</p>

      <label for="model">Model</label>
      <input id="model" type="text" autocomplete="off">
      <p class="hint">Keep the default unless the app says the model was not found.</p>

      <div class="row">
        <button class="btn quiet" id="closeSettings">Cancel</button>
        <button class="btn primary" id="saveSettings">Save</button>
      </div>
    </div>
  </div>

<script>
(() => {
  const $ = s => document.querySelector(s);
  const orb = $('#orb'), statusEl = $('#status'), liveEl = $('#live'), logEl = $('#log'), sheet = $('#sheet');

  const SYSTEM = [
    "You are ADVISOR, a warm, thoughtful friend the person talks to by voice, like a phone call.",
    "Speak naturally in short turns: usually one to three sentences. Never use lists, markdown, emojis or symbols, because your words are read aloud.",
    "Always reply in the language and dialect the person is using. If they speak Egyptian Arabic, answer in Egyptian Arabic. If they speak Modern Standard Arabic, answer in that.",
    "Listen first. Ask at most one question at a time. Remember what was said earlier in the conversation.",
    "You can explain things simply, help people think through life questions, and teach languages: say a short phrase, explain it briefly, then invite the person to repeat it.",
    "Give honest, balanced advice and do not just tell people what they want to hear. For medical, legal, money or safety questions, give general information, say plainly what you cannot know, and suggest a qualified professional.",
    "If someone seems to be in danger or deeply distressed, respond with care and encourage them to reach someone they trust or local emergency services."
  ].join(" ");

  const DEFAULTS = { key: '', proxy: '', lang: 'en-US', model: 'gemini-flash-latest' };
  let cfg = { ...DEFAULTS };
  try { cfg = { ...DEFAULTS, ...JSON.parse(localStorage.getItem('advisor-cfg') || '{}') }; } catch (e) {}
  const saveCfg = () => { try { localStorage.setItem('advisor-cfg', JSON.stringify(cfg)); } catch (e) {} };

  let mode = 'idle', history = [], rec = null, voices = [];

  const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
  const STATUS = {
    idle: 'Tap the orb and start talking',
    listening: 'Listening… tap again when you finish',
    thinking: 'Thinking…',
    speaking: 'Speaking… tap to interrupt'
  };

  function setMode(m, text) {
    mode = m;
    orb.dataset.state = m;
    statusEl.textContent = text || STATUS[m];
    if (m !== 'listening') liveEl.textContent = '';
  }

  function addMsg(role, text) {
    const empty = logEl.querySelector('.empty'); if (empty) empty.remove();
    const d = document.createElement('div');
    d.className = 'msg ' + (role === 'user' ? 'me' : 'ai');
    d.dir = 'auto';
    d.textContent = text;
    logEl.appendChild(d);
    logEl.scrollTop = logEl.scrollHeight;
  }

  function showEmpty() {
    logEl.innerHTML = '';
    const d = document.createElement('div');
    d.className = 'empty';
    d.textContent = 'Say hello, ask for advice, or ask to practise a language.';
    logEl.appendChild(d);
  }

  /* ---------- Listening ---------- */
  function startListening() {
    if (!SR) { setMode('idle', 'This browser cannot listen. Open ADVISOR in Chrome on Android.'); return; }
    if (!cfg.key && !cfg.proxy) { openSettings(); setMode('idle', 'Add your API key in Settings first'); return; }
    stopSpeaking();
    rec = new SR();
    rec.lang = cfg.lang;
    rec.interimResults = true;
    rec.continuous = false;
    let finalText = '', failed = false;

    rec.onresult = e => {
      let interim = '';
      for (let i = e.resultIndex; i < e.results.length; i++) {
        const r = e.results[i];
        if (r.isFinal) finalText += r[0].transcript; else interim += r[0].transcript;
      }
      liveEl.textContent = finalText + interim;
    };
    rec.onerror = e => {
      failed = true;
      const msg = {
        'not-allowed': 'Microphone is blocked. Allow it in Chrome site settings, then try again.',
        'service-not-allowed': 'Microphone is blocked. Allow it in Chrome site settings, then try again.',
        'no-speech': "I didn't hear anything. Tap the orb and try again.",
        'audio-capture': 'No microphone found.',
        'network': 'Speech recognition needs an internet connection.',
        'language-not-supported': 'This language is not supported for listening on this phone.'
      }[e.error];
      if (e.error !== 'aborted') setMode('idle', msg || 'Listening stopped. Tap to try again.');
    };
    rec.onend = () => {
      if (failed || mode !== 'listening') return;
      const text = finalText.trim() || liveEl.textContent.trim();
      if (text) send(text); else setMode('idle', "I didn't hear anything. Tap the orb and try again.");
    };
    try { rec.start(); setMode('listening'); }
    catch (e) { setMode('idle', 'Could not start the microphone. Tap to try again.'); }
  }

  function stopListening() { if (rec) { try { rec.stop(); } catch (e) {} } }

  /* ---------- Thinking ---------- */
  async function send(text) {
    addMsg('user', text);
    history.push({ role: 'user', content: text });
    history = history.slice(-16);
    while (history.length && history[0].role !== 'user') history.shift();
    setMode('thinking');

    try {
      const model = (cfg.model && !/^claude/i.test(cfg.model)) ? cfg.model : DEFAULTS.model;
      const url = cfg.proxy || ('https://generativelanguage.googleapis.com/v1beta/models/' + encodeURIComponent(model) + ':generateContent');
      const headers = { 'content-type': 'application/json' };
      if (!cfg.proxy) headers['x-goog-api-key'] = cfg.key;
      const payload = {
        systemInstruction: { parts: [{ text: SYSTEM }] },
        contents: history.map(m => ({ role: m.role === 'assistant' ? 'model' : 'user', parts: [{ text: m.content }] }))
      };
      if (cfg.proxy) payload.model = model;
      const res = await fetch(url, { method: 'POST', headers, body: JSON.stringify(payload) });
      const data = await res.json();
      if (!res.ok) {
        const e = new Error((data && data.error && data.error.message) || ('Error ' + res.status));
        e.status = res.status;
        throw e;
      }
      const parts = (data.candidates && data.candidates[0] && data.candidates[0].content && data.candidates[0].content.parts) || [];
      const reply = parts.map(p => p.text || '').join(' ').trim();
      if (!reply) throw new Error('Empty reply');
      history.push({ role: 'assistant', content: reply });
      addMsg('assistant', reply);
      speak(reply);
    } catch (err) {
      history.pop();
      let msg = 'Something went wrong. Tap the orb to try again.';
      const m = String(err.message || err);
      if (err.status === 429 || /quota|rate limit|RESOURCE_EXHAUSTED/i.test(m)) msg = 'Too many requests for the free limit. Wait a minute and try again.';
      else if (err.status === 400 && /API key/i.test(m) || err.status === 401 || err.status === 403) msg = 'The API key was rejected. Check it in Settings.';
      else if (err.status === 404) msg = 'That model name was not found. Check the model in Settings.';
      else if (/Failed to fetch|NetworkError/i.test(m)) msg = 'No connection to the AI. Check your internet.';
      setMode('idle', msg);
    }
  }

  /* ---------- Speaking ---------- */
  function loadVoices() { voices = window.speechSynthesis ? speechSynthesis.getVoices() : []; }
  if (window.speechSynthesis) { loadVoices(); speechSynthesis.onvoiceschanged = loadVoices; }

  function pickLang(text) {
    const arabic = /[\u0600-\u06FF]/.test(text);
    const sel = cfg.lang;
    if (arabic) return sel.startsWith('ar') ? sel : 'ar-SA';
    return sel.startsWith('ar') ? 'en-US' : sel;
  }

  function pickVoice(lang) {
    const norm = v => v.lang.replace('_', '-').toLowerCase();
    const l = lang.toLowerCase(), p = l.split('-')[0];
    return voices.find(v => norm(v) === l) || voices.find(v => norm(v).split('-')[0] === p) || null;
  }

  function speak(text) {
    if (!window.speechSynthesis) { setMode('idle', 'This browser cannot speak aloud.'); return; }
    stopSpeaking();
    const lang = pickLang(text);
    const u = new SpeechSynthesisUtterance(text);
    u.lang = lang;
    const v = pickVoice(lang);
    if (v) u.voice = v;
    u.rate = 1;
    u.onend = () => { if (mode === 'speaking') setMode('idle'); };
    u.onerror = () => { if (mode === 'speaking') setMode('idle'); };
    setMode('speaking', v ? undefined : 'Speaking… (no voice installed for this language: add one in Android Settings > Text-to-speech)');
    speechSynthesis.speak(u);
  }

  function stopSpeaking() { if (window.speechSynthesis) speechSynthesis.cancel(); }

  /* ---------- Controls ---------- */
  orb.addEventListener('click', () => {
    if (mode === 'idle') startListening();
    else if (mode === 'listening') stopListening();
    else if (mode === 'speaking') { stopSpeaking(); setMode('idle'); startListening(); }
  });

  $('#newChat').addEventListener('click', () => {
    stopSpeaking(); stopListening(); history = []; showEmpty(); setMode('idle');
  });

  function openSettings() {
    $('#lang').value = cfg.lang; $('#key').value = cfg.key; $('#proxy').value = cfg.proxy; $('#model').value = cfg.model;
    sheet.classList.add('open');
  }
  function closeSettings() { sheet.classList.remove('open'); }
  $('#openSettings').addEventListener('click', openSettings);
  $('#closeSettings').addEventListener('click', closeSettings);
  sheet.addEventListener('click', e => { if (e.target === sheet) closeSettings(); });
  $('#saveSettings').addEventListener('click', () => {
    cfg.lang = $('#lang').value;
    cfg.key = $('#key').value.trim();
    cfg.proxy = $('#proxy').value.trim().replace(/\/+$/, '');
    cfg.model = $('#model').value.trim() || DEFAULTS.model;
    saveCfg(); closeSettings(); setMode('idle');
  });

  /* ---------- Start ---------- */
  showEmpty();
  if (!window.isSecureContext) setMode('idle', 'The microphone only works on a secure (https) page.');
  else if (!cfg.key && !cfg.proxy) openSettings();
})();
</script>
</body>
</html>
