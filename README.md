# 🚗 Tesla Media Center v2.0

Site de demonstração com bypass de condução para reprodução de vídeo no browser do Tesla — com suporte a **YouTube**, **upload local** e **Plex Media Server**.

**[🔗 Ver Demo ao Vivo](https://davidfferreira.github.io/TeslaMediaCenter)**

---

## Funcionalidades

| Tab | Descrição |
|---|---|
| **YouTube** | Embed com bypass de condução ativo |
| **Upload** | Carrega qualquer MP4/WebM local (fica em memória, sem upload) |
| **Plex** | Liga ao teu Plex Media Server e reproduz filmes/séries com bypass |

## Como funciona o Bypass

O browser do Tesla (Chromium) dispara o evento `visibilitychange` com `hidden` quando o carro entra em marcha. Este site interceta esse evento e força a retoma do vídeo.

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
- `visibilitychange` listener — deteta quando o Tesla esconde o browser
- `AudioContext API` — mantém o pipeline de áudio ativo
- `Wake Lock API` — impede o ecrã de adormecer
- Pause intercept — retoma vídeo pausado involuntariamente

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

## Publicar no GitHub Pages

```
Settings → Pages → Deploy from branch → main / (root) → Save
```

URL: `https://SEU-USERNAME.github.io/TeslaMediaCenter`

## ⚠️ Aviso

Este projeto é apenas para fins educativos. Não uses o ecrã do Tesla para ver vídeo enquanto conduzes — é ilegal e perigoso.

---

Ficheiro único HTML · Zero dependências · Zero servidor
