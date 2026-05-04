# 🚗 Tesla Media Center v2.0

Site de demonstração com bypass de condução para reprodução de vídeo no browser do Tesla — com suporte a **YouTube**, **upload local**, **Plex Media Server** e **IPTV (M3U)**.

**[🔗 Ver Demo ao Vivo](https://davidfferreira.github.io/TeslaMediaCenter)**

---

## Como usar no Tesla (passo a passo)

### 1. Abrir o site
No browser do Tesla, navega para:
```
https://davidfferreira.github.io/TeslaMediaCenter
```
Guarda nos favoritos para acesso rápido.

### 2. Escolher a fonte de vídeo
Seleciona a aba que queres usar — YouTube, Upload, Plex ou IPTV — e inicia a reprodução **antes de arrancar**.

### 3. Carregar play e arrancar
Dá play no vídeo enquanto o carro ainda está estacionado. Quando arrancares, o browser muda automaticamente para modo condução — o bypass interceta esse evento e o vídeo continua sem parar.

> **O botão "Simular" é apenas para testar fora do carro** (no PC ou telemóvel). No Tesla real, o bypass ativa-se sozinho assim que o carro entra em marcha — não precisas de fazer nada.

### 4. Durante a viagem
O vídeo/áudio continua a tocar normalmente. Se o ecrã do Tesla desligar momentaneamente, o Wake Lock tenta mantê-lo ativo. Se o vídeo pausar por qualquer motivo, o Pause Intercept retoma-o automaticamente em menos de 100ms.

---

## Funcionalidades

| Tab | Descrição |
|---|---|
| **YouTube** | Pesquisa de vídeos integrada + embed com bypass de condução ativo |
| **Upload** | Carrega qualquer MP4/WebM local (fica em memória, sem upload) |
| **Plex** | Liga ao teu Plex Media Server e reproduz filmes/séries com bypass |
| **JellyFin** | Liga ao teu JellyFin Media Server e reproduz filmes/séries com bypass |
| **IPTV** | Cola um URL de lista M3U e reproduz canais ao vivo com bypass |

---

## Como funciona o Bypass

O browser do Tesla (Chromium) dispara o evento `visibilitychange` com `hidden` automaticamente quando o carro entra em marcha. Este site interceta esse evento e força a retoma do vídeo — **sem qualquer ação manual**.

```javascript
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'hidden') {
    if (audioCtx?.state === 'suspended') audioCtx.resume();
    video.play().catch(() => {});
  }
});

// Interceta pausa involuntária
video.addEventListener('pause', () => {
  if (!userPaused) setTimeout(() => video.play(), 80);
});
```

**Técnicas combinadas:**
- `visibilitychange` listener — deteta quando o Tesla esconde o browser ao entrar em marcha
- `AudioContext API` — mantém o pipeline de áudio ativo em segundo plano
- `Wake Lock API` — impede o ecrã de adormecer
- Pause intercept — retoma vídeo pausado involuntariamente em ~80ms

---

## YouTube — Pesquisa integrada

1. Na aba **YouTube**, usa a barra de pesquisa para procurar vídeos
2. Seleciona o resultado desejado — carrega automaticamente no player
3. Dá play e arranca — o bypass mantém o vídeo ativo

> Se a pesquisa falhar por restrições CORS, aparece um campo para colar diretamente um URL ou ID do YouTube.

---

## IPTV — Listas M3U

### Como configurar
1. Vai à aba **IPTV**
2. Cola o URL da tua lista M3U (ex: `http://servidor.com/lista.m3u`)
3. Clica em **Carregar Lista**
4. Escolhe o grupo (desporto, filmes, PT, etc.) ou filtra pelo nome
5. Clica num canal — o player abre com o bypass ativo

### Navegação durante a condução
Com o vídeo a tocar, podes usar os botões **◀ Anterior** e **Próximo ▶** para trocar de canal sem voltar à lista.

> **Nota CORS:** Alguns servidores IPTV bloqueiam pedidos externos. O site tenta primeiro o acesso direto e, se falhar, usa um proxy automático. Para listas na rede local do carro (Wi-Fi), funciona diretamente sem restrições.

---

## Configurar o Plex

### Método 1 — Token manual
1. Abre [app.plex.tv](https://app.plex.tv) no browser
2. Clica com botão direito num filme → **Ver XML**
3. No URL da nova tab, copia o valor de `X-Plex-Token=...`
4. Cola no campo do site + URL do teu servidor (ex: `http://192.168.1.100:32400`)

### Método 2 — Autenticação PIN
1. Clica em **"Autenticar via plex.tv (PIN)"**
2. Vai a [plex.tv/link](https://plex.tv/link) no telemóvel
3. Introduz o código de 4 letras mostrado
4. O site liga automaticamente

> **Nota:** O Plex Server precisa de ter CORS permissivo ou estar acessível via HTTPS para funcionar no GitHub Pages. Para uso local na rede Wi-Fi do carro funciona sem restrições.

---
## ⚠️ Aviso

Este projeto é apenas para fins educativos. Não uses o ecrã do Tesla para ver vídeo enquanto conduzes — é ilegal e perigoso.

