<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Creative Engine — Gerador de Prompts IA</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');

  :root {
    --bg: #080808;
    --surface: #111;
    --surface2: #181818;
    --border: #1e1e1e;
    --border2: #2a2a2a;
    --accent: #e8ff3d;
    --accent2: #ff6b35;
    --accent3: #3dffb0;
    --text: #efefef;
    --muted: #555;
    --muted2: #888;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.035'/%3E%3C/svg%3E");
    pointer-events: none;
    z-index: 9999;
  }

  /* ── SETUP SCREEN ── */
  #setupScreen {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 24px;
    animation: fadeIn .5s ease;
  }

  .setup-card {
    width: 100%;
    max-width: 500px;
    border: 1px solid var(--border2);
    border-radius: 12px;
    overflow: hidden;
    background: var(--surface);
  }

  .setup-top {
    padding: 32px 32px 24px;
    border-bottom: 1px solid var(--border);
    text-align: center;
    background: linear-gradient(160deg, #111 0%, #0d0d0d 100%);
  }

  .logo-big {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 44px;
    letter-spacing: 4px;
    color: var(--accent);
    line-height: 1;
    margin-bottom: 6px;
  }
  .logo-big span { color: var(--text); }

  .setup-subtitle {
    font-size: 13px;
    color: var(--muted2);
  }

  /* badge Gemini */
  .gemini-badge {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    margin-top: 12px;
    padding: 5px 14px;
    border-radius: 20px;
    background: rgba(66,133,244,0.12);
    border: 1px solid rgba(66,133,244,0.3);
    font-size: 11px;
    font-family: 'DM Mono', monospace;
    color: #7ab4ff;
    letter-spacing: 1px;
  }

  .setup-body { padding: 28px 32px 32px; }

  .setup-label {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    letter-spacing: 2px;
    color: var(--muted2);
    text-transform: uppercase;
    margin-bottom: 8px;
    display: block;
  }

  .setup-input {
    width: 100%;
    background: var(--bg);
    border: 1px solid var(--border2);
    border-radius: 8px;
    padding: 13px 16px;
    color: var(--text);
    font-family: 'DM Mono', monospace;
    font-size: 13px;
    outline: none;
    transition: border-color .2s;
  }
  .setup-input:focus { border-color: var(--accent); }
  .setup-input::placeholder { color: var(--muted); }

  .setup-hint {
    font-size: 11px;
    color: var(--muted);
    margin-top: 8px;
    line-height: 1.6;
  }
  .setup-hint a { color: #7ab4ff; text-decoration: none; }
  .setup-hint a:hover { text-decoration: underline; }

  .setup-btn {
    width: 100%;
    margin-top: 20px;
    padding: 14px;
    background: var(--accent);
    border: none;
    border-radius: 8px;
    color: #000;
    font-family: 'Bebas Neue', sans-serif;
    font-size: 18px;
    letter-spacing: 2px;
    cursor: pointer;
    transition: all .2s;
  }
  .setup-btn:hover { background: #f5ff6b; transform: translateY(-1px); }

  .setup-info {
    margin-top: 16px;
    padding: 12px 14px;
    background: rgba(61,255,176,0.04);
    border: 1px solid rgba(61,255,176,0.15);
    border-radius: 6px;
    font-size: 11px;
    color: var(--muted2);
    line-height: 1.6;
  }

  /* ── APP ── */
  #appScreen { display: none; flex: 1; flex-direction: column; }
  #appScreen.visible { display: flex; }

  header {
    padding: 14px 28px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid var(--border);
    background: rgba(8,8,8,.95);
    backdrop-filter: blur(16px);
    position: sticky;
    top: 0;
    z-index: 100;
    flex-shrink: 0;
  }

  .logo { font-family: 'Bebas Neue', sans-serif; font-size: 22px; letter-spacing: 3px; color: var(--accent); }
  .logo span { color: var(--text); }

  .header-right { display: flex; align-items: center; gap: 10px; }

  .mode-pills { display: flex; gap: 4px; }
  .mode-pill {
    padding: 5px 12px;
    border-radius: 20px;
    border: 1px solid var(--border2);
    font-size: 11px;
    font-family: 'DM Mono', monospace;
    letter-spacing: 1px;
    cursor: pointer;
    color: var(--muted2);
    background: transparent;
    transition: all .2s;
  }
  .mode-pill:hover { color: var(--text); }
  .mode-pill.active { background: var(--accent); color: #000; border-color: var(--accent); }

  .key-badge {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    padding: 5px 10px;
    border: 1px solid var(--border2);
    border-radius: 4px;
    color: var(--muted);
    cursor: pointer;
    transition: all .2s;
  }
  .key-badge:hover { border-color: #7ab4ff; color: #7ab4ff; }

  .app-body { flex: 1; display: flex; overflow: hidden; height: calc(100vh - 57px); }
  .chat-col { flex: 1; display: flex; flex-direction: column; min-width: 0; }

  .messages {
    flex: 1;
    overflow-y: auto;
    padding: 28px 32px;
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .msg { display: flex; gap: 12px; animation: fadeUp .3s ease; }
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

  .avatar {
    width: 30px; height: 30px;
    border-radius: 6px;
    display: flex; align-items: center; justify-content: center;
    font-family: 'Bebas Neue', sans-serif;
    font-size: 11px;
    letter-spacing: 1px;
    flex-shrink: 0;
    margin-top: 2px;
  }
  .avatar.bot { background: var(--accent); color: #000; }
  .avatar.user { background: var(--surface2); color: var(--text); border: 1px solid var(--border2); }

  .msg-inner { flex: 1; min-width: 0; }
  .msg-name { font-family: 'DM Mono', monospace; font-size: 10px; color: var(--muted); letter-spacing: 1.5px; margin-bottom: 6px; }

  .bubble { font-size: 14px; line-height: 1.75; color: var(--text); }
  .bubble p { margin-bottom: 8px; }
  .bubble p:last-child { margin-bottom: 0; }
  .bubble strong { color: var(--accent); font-weight: 600; }
  .bubble em { color: var(--muted2); font-style: italic; }
  .bubble code {
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    background: var(--surface2);
    padding: 2px 6px;
    border-radius: 3px;
    color: var(--accent3);
  }

  .user-bubble {
    background: var(--surface2);
    border: 1px solid var(--border2);
    border-radius: 10px 10px 2px 10px;
    padding: 10px 14px;
    font-size: 14px;
    line-height: 1.6;
    display: inline-block;
    max-width: 85%;
  }

  .prompt-block {
    margin-top: 12px;
    border: 1px solid var(--border2);
    border-radius: 8px;
    overflow: hidden;
    background: var(--surface2);
  }
  .prompt-block-header {
    padding: 9px 14px;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .prompt-type-tag {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    letter-spacing: 2px;
    color: var(--accent);
    text-transform: uppercase;
  }
  .copy-btn {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    padding: 3px 10px;
    border: 1px solid var(--border2);
    border-radius: 3px;
    background: transparent;
    color: var(--muted2);
    cursor: pointer;
    transition: all .2s;
    letter-spacing: 1px;
  }
  .copy-btn:hover { border-color: var(--accent); color: var(--accent); }
  .prompt-block-body {
    padding: 14px;
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    line-height: 1.8;
    color: #bbb;
    white-space: pre-wrap;
    word-break: break-word;
  }

  .typing { display: flex; align-items: center; gap: 5px; padding: 4px 0; }
  .typing span {
    width: 7px; height: 7px;
    border-radius: 50%;
    background: var(--muted);
    animation: pulse 1.3s infinite;
  }
  .typing span:nth-child(2) { animation-delay: .2s; }
  .typing span:nth-child(3) { animation-delay: .4s; }
  @keyframes pulse {
    0%,80%,100% { opacity: .3; transform: scale(.75); }
    40% { opacity: 1; transform: scale(1); }
  }

  .input-area {
    padding: 16px 32px 20px;
    border-top: 1px solid var(--border);
    display: flex;
    gap: 10px;
    align-items: flex-end;
    flex-shrink: 0;
    background: rgba(8,8,8,.9);
  }

  .input-area textarea {
    flex: 1;
    background: var(--surface);
    border: 1px solid var(--border2);
    border-radius: 10px;
    padding: 12px 16px;
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    resize: none;
    outline: none;
    min-height: 48px;
    max-height: 130px;
    line-height: 1.5;
    transition: border-color .2s;
  }
  .input-area textarea:focus { border-color: var(--accent); }
  .input-area textarea::placeholder { color: var(--muted); }

  .send-btn {
    width: 48px; height: 48px;
    background: var(--accent);
    border: none;
    border-radius: 10px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    transition: all .2s;
  }
  .send-btn:hover { background: #f5ff6b; transform: scale(1.05); }
  .send-btn:disabled { opacity: .4; cursor: not-allowed; transform: none; }

  .new-chat-btn {
    margin: 0 32px 14px;
    padding: 10px;
    background: transparent;
    border: 1px dashed var(--border2);
    border-radius: 8px;
    color: var(--muted);
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    letter-spacing: 1px;
    cursor: pointer;
    transition: all .2s;
    flex-shrink: 0;
  }
  .new-chat-btn:hover { border-color: var(--accent2); color: var(--accent2); }

  ::-webkit-scrollbar { width: 3px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 4px; }

  @media (max-width: 600px) {
    header { padding: 12px 16px; }
    .messages { padding: 20px 16px; }
    .input-area { padding: 12px 16px 16px; }
    .new-chat-btn { margin: 0 16px 12px; }
    .mode-pills { display: none; }
  }
</style>
</head>
<body>

<!-- SETUP -->
<div id="setupScreen">
  <div class="setup-card">
    <div class="setup-top">
      <div class="logo-big">Creative<span>Engine</span></div>
      <div class="setup-subtitle">Seu diretor de arte pessoal com IA</div>
      <div class="gemini-badge">⚡ Powered by Google Gemini</div>
    </div>
    <div class="setup-body">
      <label class="setup-label" for="apiKeyInput">Sua chave da API do Gemini</label>
      <input
        class="setup-input"
        id="apiKeyInput"
        type="password"
        placeholder="AIzaSy..."
        autocomplete="off"
      />
      <div class="setup-hint">
        Sua chave começa com <strong style="color:var(--text)">AIzaSy...</strong><br>
        Não tem ainda? Pegue grátis em
        <a href="https://aistudio.google.com/app/apikey" target="_blank">aistudio.google.com/app/apikey</a>
      </div>

      <button class="setup-btn" onclick="startApp()">ENTRAR NO STUDIO ›</button>

      <div class="setup-info">
        🔒 Sua chave fica salva <strong style="color:var(--text)">só no seu navegador</strong>. Ao fechar a aba, ela some automaticamente. Nenhum dado seu é enviado para outros servidores.
      </div>
    </div>
  </div>
</div>

<!-- APP -->
<div id="appScreen">
  <header>
    <div class="logo">Creative<span>Engine</span></div>
    <div class="header-right">
      <div class="mode-pills">
        <button class="mode-pill active" onclick="setMode('imagem', this)">🖼 Imagem</button>
        <button class="mode-pill" onclick="setMode('movimento', this)">🎬 Movimento</button>
        <button class="mode-pill" onclick="setMode('comercial', this)">📣 Comercial</button>
      </div>
      <div class="key-badge" onclick="resetKey()" title="Trocar chave">🔑 Gemini</div>
    </div>
  </header>

  <div class="app-body">
    <div class="chat-col">
      <div class="messages" id="messages"></div>
      <button class="new-chat-btn" onclick="newChat()">↺ Nova conversa</button>
      <div class="input-area">
        <textarea
          id="userInput"
          placeholder="Digite sua mensagem..."
          rows="1"
          onkeydown="handleKey(event)"
          oninput="autoResize(this)"
        ></textarea>
        <button class="send-btn" id="sendBtn" onclick="sendMessage()">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
            <line x1="22" y1="2" x2="11" y2="13"/>
            <polygon points="22 2 15 22 11 13 2 9 22 2"/>
          </svg>
        </button>
      </div>
    </div>
  </div>
</div>

<script>
// ═══════════════════════
// CONFIG
// ═══════════════════════
let API_KEY = '';
let currentMode = 'imagem';
let conversationHistory = [];
let isLoading = false;

// Instruções para cada modo
const SYSTEM_INSTRUCTIONS = {
  imagem: `Você é o Creative Engine, um diretor de arte especialista em prompts para IA generativa de imagens (Midjourney, Stable Diffusion, Flux, Adobe Firefly, etc).

Seu papel é guiar o usuário passo a passo na criação de prompts cinematográficos e fotorrealistas profissionais.

SEMPRE que o usuário iniciar ou pedir um novo prompt, siga este fluxo:
1. Pergunte o PROPÓSITO da imagem (campanha, thumbnail, comercial, post, etc)
2. Peça para descrever a CENA brevemente
3. Ofereça opções de ÂNGULO DE CÂMERA numeradas:
   1. Ângulo baixo — sensação de poder e grandiosidade
   2. Altura dos olhos — naturalidade e conexão
   3. Plano aéreo — contexto e escala
   4. Close extremo — detalhe e intimidade
4. Ofereça opções de ILUMINAÇÃO numeradas:
   1. Golden hour — luz quente, sombras longas e dramáticas
   2. Low-key — sombras profundas, atmosfera misteriosa
   3. Cinematográfico — estilo blockbuster Hollywood
   4. Estúdio limpo — nítido e comercial
5. Gere o PROMPT FINAL dentro de um bloco de código (usando três crases)

No prompt final SEMPRE inclua: cena detalhada, câmera técnica (ex: ARRI Alexa Mini LF, lente anamórfica 40mm, f/2.8), iluminação técnica, paleta de cores, qualidade (ultra realista, 8K, grão de filme 35mm), e termos negativos no final.

Após gerar, explique brevemente o que cada parte técnica do prompt faz. Isso ensina o usuário.

Responda SEMPRE em português brasileiro. Seja direto e didático.`,

  movimento: `Você é o Creative Engine, especialista em prompts de movimento para IA generativa de vídeo (Kling, Runway Gen-3, Sora, Hailuo, Higgsfield, etc).

Guie o usuário passo a passo na criação de prompts de movimentação de câmera cinematográfica.

FLUXO obrigatório:
1. Peça a descrição da CENA ou imagem que será animada
2. Ofereça opções de MOVIMENTO DE CÂMERA numeradas:
   1. Slow Dolly In — avança sutilmente criando tensão
   2. Arc Shot — arco suave ao redor do sujeito
   3. Drone Ascend — sobe revelando o ambiente
   4. Tracking Shot — acompanha o sujeito em movimento
   5. Handheld — leve tremor orgânico, estilo documentário
3. Ofereça opções de DISTÂNCIA numeradas:
   1. Extreme Close-Up — poros e microdetalhes visíveis
   2. Close Médio — expressão + contexto
   3. Plano Médio — corpo inteiro e ação
   4. Plano Aberto — ambiente dominante
4. Pergunte sobre MICROMOVIMENTOS do sujeito (respiração, olhar, etc)
5. Pergunte sobre comportamento da LUZ durante o movimento
6. Gere o PROMPT FINAL dentro de um bloco de código (três crases)

No prompt SEMPRE inclua: duração (4-8 segundos), frame rate (24fps cinemático), qualidade 4K, movimento fluído sem shake.

Explique cada decisão para o usuário aprender a pensar como diretor.

Responda SEMPRE em português brasileiro.`,

  comercial: `Você é o Creative Engine, diretor criativo especialista em comerciais com IA generativa.

Crie conceitos e roteiros completos de comercial com prompts para cada cena.

FLUXO obrigatório:
1. Pergunte: nome da marca ou produto
2. Pergunte: quem é o público-alvo
3. Ofereça opções de EMOÇÃO CENTRAL numeradas:
   1. Ambição e poder — "você pode mais"
   2. Conexão humana — emocional e íntimo
   3. Urgência e ação — CTA direto e forte
   4. Luxo e exclusividade — aspiracional
4. Ofereça opções de DURAÇÃO numeradas:
   1. 15 segundos — reel ou story
   2. 30 segundos — padrão TV e YouTube
   3. 60 segundos — narrativa completa
5. Gere o ROTEIRO COMPLETO dentro de um bloco de código (três crases) com:
   - Número e nome de cada cena
   - Descrição visual detalhada
   - Prompt de imagem para a cena
   - Prompt de movimento sugerido
   - Duração da cena
   - Texto ou narração sugerida

Seja criativo como um diretor de publicidade de alto nível. Explique as escolhas criativas.

Responda SEMPRE em português brasileiro.`
};

// ═══════════════════════
// SETUP
// ═══════════════════════
function startApp() {
  const key = document.getElementById('apiKeyInput').value.trim();
  if (!key || key.length < 10) {
    alert('Cole sua chave do Gemini. Ela começa com AIzaSy...');
    return;
  }
  API_KEY = key;
  sessionStorage.setItem('ce_gemini_key', key);
  document.getElementById('setupScreen').style.display = 'none';
  document.getElementById('appScreen').classList.add('visible');
  newChat();
}

function resetKey() {
  if (!confirm('Trocar a chave da API?')) return;
  sessionStorage.removeItem('ce_gemini_key');
  API_KEY = '';
  conversationHistory = [];
  document.getElementById('appScreen').classList.remove('visible');
  document.getElementById('setupScreen').style.display = 'flex';
  document.getElementById('apiKeyInput').value = '';
}

window.onload = () => {
  const saved = sessionStorage.getItem('ce_gemini_key');
  if (saved) {
    API_KEY = saved;
    document.getElementById('setupScreen').style.display = 'none';
    document.getElementById('appScreen').classList.add('visible');
    newChat();
  }
};

// ═══════════════════════
// MODE
// ═══════════════════════
function setMode(mode, btn) {
  currentMode = mode;
  document.querySelectorAll('.mode-pill').forEach(p => p.classList.remove('active'));
  btn.classList.add('active');
  newChat();
}

// ═══════════════════════
// CHAT
// ═══════════════════════
function newChat() {
  conversationHistory = [];
  document.getElementById('messages').innerHTML = '';
  document.getElementById('userInput').value = '';

  const welcomes = {
    imagem: '👋 Olá! Sou o **Creative Engine**.\n\nVou te guiar na criação de **prompts de imagem cinematográfica profissional** — com câmera, lente, iluminação e composição detalhados, igual um diretor de arte de verdade.\n\nPara começar: **para que você vai usar essa imagem?**\n(Ex: campanha de marca, thumbnail de vídeo, post no Instagram, comercial...)',
    movimento: '🎬 Olá! Sou o **Creative Engine**.\n\nVou te ajudar a criar **prompts de movimentação de câmera** para animar suas imagens com IA — com movimentos profissionais de cinema.\n\nPara começar: **descreva a imagem ou cena que você quer animar.**',
    comercial: '📣 Olá! Sou o **Creative Engine**.\n\nVou criar um **roteiro completo de comercial** com prompts de imagem e vídeo para cada cena — do jeito que agências profissionais fazem.\n\nPara começar: **qual é a marca ou produto que você quer anunciar?**'
  };

  addBotMessage(welcomes[currentMode]);
  conversationHistory.push({ role: 'model', parts: [{ text: welcomes[currentMode] }] });
}

// ═══════════════════════
// GEMINI API
// ═══════════════════════
async function sendMessage() {
  const input = document.getElementById('userInput');
  const text = input.value.trim();
  if (!text || isLoading) return;

  input.value = '';
  input.style.height = 'auto';
  isLoading = true;
  document.getElementById('sendBtn').disabled = true;

  addUserMessage(text);
  conversationHistory.push({ role: 'user', parts: [{ text }] });

  const typingId = addTyping();

  try {
    // Gemini 2.0 Flash — rápido e gratuito
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`;

    const body = {
      system_instruction: {
        parts: [{ text: SYSTEM_INSTRUCTIONS[currentMode] }]
      },
      contents: conversationHistory,
      generationConfig: {
        temperature: 0.85,
        maxOutputTokens: 2048
      }
    };

    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    });

    removeTyping(typingId);

    if (!response.ok) {
      const err = await response.json();
      const msg = err?.error?.message || 'Erro na API. Verifique sua chave do Gemini.';
      addBotMessage(`❌ **Erro:** ${msg}\n\nDica: confirme que sua chave está correta em aistudio.google.com`);
      isLoading = false;
      document.getElementById('sendBtn').disabled = false;
      return;
    }

    const data = await response.json();
    const reply = data?.candidates?.[0]?.content?.parts?.[0]?.text || 'Não consegui gerar uma resposta. Tente novamente.';

    conversationHistory.push({ role: 'model', parts: [{ text: reply }] });
    addBotMessage(reply);

  } catch (e) {
    removeTyping(typingId);
    addBotMessage(`❌ **Erro de conexão.** Verifique sua internet.\n\n*Detalhe: ${e.message}*`);
  }

  isLoading = false;
  document.getElementById('sendBtn').disabled = false;
}

// ═══════════════════════
// RENDER
// ═══════════════════════
function addBotMessage(text) {
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.className = 'msg';

  const rendered = renderMarkdown(text);

  div.innerHTML = `
    <div class="avatar bot">CE</div>
    <div class="msg-inner">
      <div class="msg-name">CREATIVE ENGINE</div>
      <div class="bubble">${rendered}</div>
    </div>
  `;
  msgs.appendChild(div);

  // Transforma blocos de código em prompt-blocks visuais
  div.querySelectorAll('pre code').forEach(block => {
    const code = block.textContent;
    const wrapper = document.createElement('div');
    wrapper.className = 'prompt-block';
    const safeCode = code.replace(/`/g, '&#96;');
    wrapper.innerHTML = `
      <div class="prompt-block-header">
        <span class="prompt-type-tag">✦ PROMPT GERADO</span>
        <button class="copy-btn" id="cbtn-${Date.now()}">⎘ Copiar</button>
      </div>
      <div class="prompt-block-body">${escHtml(code)}</div>
    `;
    // Adiciona evento de copiar
    const btn = wrapper.querySelector('.copy-btn');
    btn.addEventListener('click', () => {
      navigator.clipboard.writeText(code).then(() => {
        btn.textContent = '✓ Copiado!';
        btn.style.color = 'var(--accent3)';
        setTimeout(() => { btn.textContent = '⎘ Copiar'; btn.style.color = ''; }, 2000);
      });
    });
    block.closest('pre').replaceWith(wrapper);
  });

  scrollBottom();
}

function addUserMessage(text) {
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.className = 'msg';
  div.style.justifyContent = 'flex-end';
  div.innerHTML = `
    <div class="msg-inner" style="display:flex;flex-direction:column;align-items:flex-end">
      <div class="msg-name" style="text-align:right">VOCÊ</div>
      <div class="user-bubble">${escHtml(text)}</div>
    </div>
    <div class="avatar user">EU</div>
  `;
  msgs.appendChild(div);
  scrollBottom();
}

let typingCount = 0;
function addTyping() {
  const id = 'typing-' + (++typingCount);
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.className = 'msg';
  div.id = id;
  div.innerHTML = `
    <div class="avatar bot">CE</div>
    <div class="msg-inner">
      <div class="msg-name">CREATIVE ENGINE</div>
      <div class="typing"><span></span><span></span><span></span></div>
    </div>
  `;
  msgs.appendChild(div);
  scrollBottom();
  return id;
}

function removeTyping(id) {
  const el = document.getElementById(id);
  if (el) el.remove();
}

// ═══════════════════════
// MARKDOWN SIMPLES
// ═══════════════════════
function renderMarkdown(text) {
  let t = text
    .replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')
    .replace(/```[\w]*\n?([\s\S]*?)```/g, '<pre><code>$1</code></pre>')
    .replace(/`([^`]+)`/g, '<code>$1</code>')
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    .replace(/\*(.*?)\*/g, '<em>$1</em>')
    .replace(/^### (.+)$/gm, '<p><strong style="font-size:14px;color:var(--accent2)">$1</strong></p>')
    .replace(/^## (.+)$/gm, '<p><strong style="font-size:15px;color:var(--accent)">$1</strong></p>')
    .replace(/^# (.+)$/gm, '<p><strong style="font-size:17px;color:var(--accent)">$1</strong></p>')
    .replace(/^---+$/gm, '<hr style="border:none;border-top:1px solid var(--border2);margin:12px 0">')
    .replace(/^\d+\. (.+)$/gm, '<p style="padding-left:16px">• $1</p>')
    .replace(/^[-*] (.+)$/gm, '<p style="padding-left:16px">· $1</p>')
    .replace(/\n\n/g, '</p><p>')
    .replace(/\n/g, '<br>');
  // wrap em parágrafo se não tiver tag
  if (!t.startsWith('<')) t = '<p>' + t + '</p>';
  return t;
}

function escHtml(t) {
  return t.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}

function scrollBottom() {
  const m = document.getElementById('messages');
  m.scrollTop = m.scrollHeight;
}

function handleKey(e) {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMessage(); }
}

function autoResize(el) {
  el.style.height = 'auto';
  el.style.height = Math.min(el.scrollHeight, 130) + 'px';
}
</script>
</body>
</html>
