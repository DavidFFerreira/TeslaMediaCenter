# 🚗 Tesla Media Center v2.0

Site de ficheiro único (`index.html`) que permite reproduzir **YouTube, Netflix, Upload local, Plex, Jellyfin e IPTV** no browser Chromium do Tesla — com bypass de condução ativo, dados do veículo em tempo real e zero dependências externas.

**[🔗 Ver Demo ao Vivo](https://davidfferreira.github.io/TeslaMediaCenter)**

---

## ☕ Apoiar o Projeto

Podes poupar em gasolina, mas o café do developer ainda custa dinheiro!

**[→ Pagar um café via PayPal](https://paypal.me/Davidfferreira1986?locale.x=pt_PT&country.x=PT)**

> _"O Tesla carrega sozinho. O developer não."_ 🔋

---

## Como usar no Tesla (passo a passo)

### 1. Abrir o site
No browser do Tesla, navega para:
```
https://davidfferreira.github.io/TeslaMediaCenter
```
Guarda nos favoritos para acesso rápido.

### 2. Escolher a fonte de vídeo
Seleciona a aba pretendida — YouTube, Netflix, Upload, Plex, Jellyfin ou IPTV — e inicia a reprodução **antes de arrancar**.

### 3. Dar play e arrancar
Com o vídeo a tocar e o carro estacionado, arranca normalmente. O browser muda automaticamente para modo condução — o bypass interceta esse evento e o vídeo **continua sem interrupção**.

> **O botão "Simular"** existe apenas para testar o bypass no PC ou telemóvel. No Tesla real o bypass ativa-se sozinho ao entrar em marcha — não precisas de fazer nada.

### 4. Durante a viagem
O vídeo e o áudio continuam ativos. O Wake Lock impede o ecrã de adormecer. Se o vídeo pausar por qualquer motivo, o Pause Intercept retoma-o automaticamente em menos de 120 ms.

---

## Funcionalidades por Tab

| Tab | Descrição |
|---|---|
| **▶ YouTube** | Pesquisa integrada via API Invidious + embed com bypass ativo. Fallback para colar URL/ID diretamente. |
| **🎞 Netflix** | Instruções para abrir a Netflix numa nova tab com bypass sempre ativo nesta página. |
| **⬆ Upload** | Carrega um ficheiro MP4/WebM local — ideal para testar o bypass no PC ou telemóvel. |
| **🎬 Plex** | Liga ao teu Plex Media Server via token manual ou autenticação PIN OAuth. Biblioteca completa com grid de capas. |
| **🪼 Jellyfin** | Liga ao teu servidor Jellyfin com utilizador/palavra-passe. Direct stream sem transcodificação. |
| **📡 IPTV** | Cola um URL de lista M3U e reproduz canais ao vivo. Filtros por grupo e nome, navegação de canal sem sair do player. |
| **🚗 Tesla** | Dados do veículo em tempo real: velocidade, marcha, bateria, autonomia, temperaturas, localização, potência, odómetro. Atualiza a cada 2 segundos. |
| **</> Código** | Código-fonte do bypass (tesla-bypass.js) para copiar e usar noutras páginas. |

---

## HUD e Status Bar

No topo da página existe um **HUD** com:
- **Modo** — PARKED / DRIVING / REVERSE (lido diretamente do gear do veículo)
- **Bypass** — estado do motor de bypass
- **Hora** — relógio em tempo real

Abaixo do HUD a **Status Bar** mostra em tempo real:
- **Visibilidade** — estado do `document.visibilityState`
- **AudioCtx** — se o pipeline WebAudio está ativo
- **Wake Lock** — se o ecrã está bloqueado contra adormecimento
- **Plex / Jellyfin / IPTV** — estado de ligação de cada serviço
- **Botão Simular** — simula o modo condução para testes

---

## Como funciona o Bypass (v2.0)

O browser do Tesla (Chromium) dispara o evento `visibilitychange → hidden` quando o carro entra em marcha. Esta versão usa **quatro camadas independentes** para garantir a retoma em todos os cenários:

### Camada 1 — Polling do gear do veículo (nova em v2.0)
```javascript
// Corre de 500ms em 500ms — lê o ShiftState diretamente do Chromium do Tesla
function pollTeslaGear() {
  const gear =
    window?.tesla?.ShiftState ??
    window?.TeslaApp?.shiftState ??
    window?.shiftState ?? null;

  if (gear !== 'P' && gear !== null) {
    // Entrou em marcha: ativa bypass imediatamente, independente do visibilitychange
    if (audioCtx?.state === 'suspended') audioCtx.resume();
    resumeAllVideos();
  }
}
setInterval(pollTeslaGear, 500);
```

### Camada 2 — visibilitychange com retry loop
```javascript
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'hidden') {
    if (audioCtx?.state === 'suspended') audioCtx.resume();
    resumeAllVideos();
    // Retry até 10x a cada 150ms — o Tesla pode atrasar a pausa até 500ms
    let retries = 0;
    const retryInterval = setInterval(() => {
      if (document.visibilityState !== 'hidden' || retries++ > 10) {
        clearInterval(retryInterval); return;
      }
      resumeAllVideos();
    }, 150);
  } else {
    grabWakeLock(); // Re-solicita Wake Lock ao voltar ao ecrã
  }
});
```

### Camada 3 — Eventos adicionais (fallback para versões antigas de Chromium)
```javascript
document.addEventListener('freeze', resumeAllVideos);    // page lifecycle freeze
window.addEventListener('pagehide', resumeAllVideos);    // navegação/swap de tab
window.addEventListener('blur', resumeAllVideos);        // perda de foco
```

### Camada 4 — Pause intercept por vídeo (debounced)
```javascript
// Instalado em cada elemento <video> individualmente
videoEl.addEventListener('pause', () => {
  if (!isPausedRef.value) {
    // 60ms se em condução, 120ms caso contrário — evita loops play/pause
    debouncedResume(isDriving ? 60 : 120);
  }
});
// Também interceta stalled, waiting e canplay
```

### Técnicas combinadas

| Técnica | Função |
|---|---|
| `pollTeslaGear` (500ms) | Lê `window.tesla.ShiftState` e ativa bypass ao detetar marcha |
| `visibilitychange` + retry | Interceta o evento principal do Tesla com 10 tentativas |
| `freeze` / `pagehide` / `blur` | Camadas extra para versões de firmware mais antigas |
| `AudioContext API` | Mantém o pipeline de áudio ativo em segundo plano |
| `Wake Lock API` | Impede o ecrã de adormecer; re-solicitado automaticamente |
| Pause intercept (debounced) | Retoma cada vídeo individualmente em 60–120 ms |
| `postMessage` ao iframe YT | Envia `playVideo` ao embed do YouTube via postMessage |

---

## Tab Tesla — Dados do Veículo

A tab **🚗 Tesla** lê diretamente as variáveis JavaScript que o Chromium do Tesla expõe no `window`. Diferentes versões de firmware usam namespaces diferentes — o código tenta todos:

```
window.tesla.VehicleSpeed / ShiftState / BatteryLevel / EstBatteryRange
window.tesla.InsideTemp / OutsideTemp / Latitude / Longitude / Power
window.TeslaApp.vehicleSpeed / shiftState / batteryLevel / ...
window.vehicleSpeed / shiftState / ... (namespace plano)
```

**Dados mostrados:**
- Velocidade (km/h)
- Marcha (P / D / R / N)
- Bateria (%) com barra visual com alerta a < 40% e crítico a < 20%
- Autonomia estimada (km)
- Temperatura interior e exterior (°C)
- Potência instantânea (kW)
- Localização GPS (latitude/longitude)
- Odómetro (km)
- Estado e taxa de carregamento
- Versão do software

> Fora do Tesla os valores aparecem como **n/d** — é o comportamento esperado. No Tesla real atualizam a cada 2 segundos.

---

## YouTube — Pesquisa integrada

A pesquisa usa a **API pública Invidious** (frontend alternativo ao YouTube com CORS aberto) com failover automático por 5 instâncias independentes:

1. Pesquisa na primeira instância disponível (timeout de 5 s por tentativa)
2. Se falhar, passa automaticamente para a próxima instância
3. Se todas falharem, mostra um campo para colar o URL ou ID do YouTube diretamente

**Usar URL/ID direto:**
- URL completo: `https://www.youtube.com/watch?v=dQw4w9WgXcQ`
- URL curto: `https://youtu.be/dQw4w9WgXcQ`
- ID direto: `dQw4w9WgXcQ`

---

## Netflix

A Netflix usa DRM Widevine L3, suportado pelo Chromium do Tesla. Como não existe app nativa para o Tesla, o método recomendado é:

1. Na aba **Netflix**, clica em **Abrir Netflix** — abre em nova tab
2. Faz login e dá play num conteúdo **antes de arrancar**
3. Volta a esta tab — o bypass fica ativo em segundo plano
4. Arranca — o AudioContext + visibilitychange mantém o áudio/vídeo ativo

> Qualidade pode variar conforme o plano e a cobertura de rede.

---

## IPTV — Listas M3U

### Como configurar
1. Vai à aba **📡 IPTV**
2. Cola o URL da tua lista M3U (ex: `http://servidor.com/lista.m3u`)
3. Clica em **Carregar Lista**
4. Filtra por grupo ou nome
5. Clica num canal — o player abre com o bypass ativo

### Formatos suportados
- Listas `.m3u` e `.m3u8`
- Streams HLS (nativamente suportado pelo Chromium do Tesla)
- Streams MP4/TS diretos
- Metadados `group-title` e `tvg-logo`

### Navegação durante a condução
Com o vídeo a tocar usa os botões **◀ Anterior** e **Próximo ▶** para trocar de canal sem voltar à lista. O bypass reinstala-se automaticamente em cada mudança de canal.

> **CORS:** O site tenta primeiro o acesso direto ao servidor. Se falhar por CORS, usa o proxy `corsproxy.io` automaticamente. Para listas na rede local do carro (Wi-Fi hotspot ou rede doméstica) funciona sempre sem restrições.

---

## Plex Media Server

### Método 1 — Token manual (mais simples)
1. Abre [app.plex.tv](https://app.plex.tv) num browser normal
2. Navega até um filme → clica com o botão direito → **Ver XML** (ou abre as definições de conta)
3. No URL da nova tab, copia o valor de `X-Plex-Token=...`
4. No site: cola o URL do servidor (ex: `http://192.168.1.100:32400`) e o token
5. Clica em **Ligar**

### Método 2 — Autenticação PIN OAuth (sem token manual)
1. Clica em **"Autenticar via plex.tv (PIN)"**
2. É gerado um código de 4 letras
3. Abre [plex.tv/link](https://plex.tv/link) no telemóvel e introduz o código
4. O site liga automaticamente, descobre o servidor e carrega a biblioteca

### O que está disponível após ligar
- Biblioteca completa com grid de capas
- Navegação por secções (filmes, séries, música, etc.)
- Direct play via `/library/parts` — sem transcodificação
- Now playing com título, ano e duração

> O Plex Server precisa de estar acessível via HTTP/HTTPS do browser do Tesla. Em rede local (Wi-Fi do carro) funciona sem restrições de CORS.

---

## Jellyfin

1. Na aba **🪼 Jellyfin**, coloca o URL do servidor (ex: `http://192.168.1.100:8096`)
2. Introduz o utilizador e a palavra-passe
3. Clica em **Ligar ao Jellyfin**
4. Navega pela biblioteca e clica num item para reproduzir

### Endpoint de stream usado
```
/Videos/{id}/stream?Static=true&MediaSourceId={sourceId}&api_key={token}
```
Direct stream sem transcodificação — desde que o formato seja compatível com o Chromium do Tesla (MP4/H.264 recomendado).

### Instalar o Jellyfin (se não tiveres)
1. Descarrega em [jellyfin.org/downloads](https://jellyfin.org/downloads) (Windows/Mac/Linux/NAS)
2. Instala e configura em `http://localhost:8096`
3. Adiciona as tuas pastas de filmes/séries como biblioteca
4. Para acesso fora de casa configura o *Remote Access* nas definições

---

## Compatibilidade

| Contexto | Estado |
|---|---|
| Tesla Chromium (qualquer modelo) | ✅ Funciona |
| Chrome / Edge / Firefox (PC) | ✅ Funciona (exceto dados Tesla que ficam n/d) |
| Safari / iOS | ⚠ Wake Lock não suportado; resto funciona |
| Android Chrome | ✅ Funciona |

---

## ⚠️ Aviso

Este projeto é apenas para fins educativos e de entretenimento como passageiro. Não uses o ecrã do Tesla para ver vídeo enquanto és tu a conduzir — é ilegal e perigoso.

