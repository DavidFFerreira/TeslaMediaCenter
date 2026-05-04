# 🚗 Tesla Video Bypass — Drive Mode Demo

Um site de demonstração que mostra como reproduzir vídeo no browser do Tesla enquanto o carro está em andamento.

**[🔗 Ver Demo ao Vivo](https://SEU-USUARIO.github.io/tesla-video)**

---

## Como Funciona

O browser do Tesla (baseado em Chromium) usa o evento `visibilitychange` para pausar vídeos quando o carro entra em marcha. Esta técnica interceta esse evento e força a retoma.

### Técnicas usadas

| Técnica | Descrição |
|---|---|
| `visibilitychange` | Deteta quando o Tesla "esconde" o browser ao entrar em marcha |
| `AudioContext API` | Mantém o pipeline de áudio ativo via Web Audio API |
| `Wake Lock API` | Impede o ecrã de adormecer |
| Pause intercept | Interceta pausas não intencionais e retoma o vídeo |

### Código principal

```javascript
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'hidden') {
    if (audioCtx?.state === 'suspended') audioCtx.resume();
    video.play().catch(() => {});
  }
});

// Interceta pausa não intencional
video.addEventListener('pause', () => {
  if (!userPaused) setTimeout(() => video.play(), 80);
});
```

## Funcionalidades do Site

- **Tab YouTube** — Embed do YouTube com bypass via `postMessage` / YT API
- **Tab Upload** — Carrega qualquer vídeo local (MP4, WebM, MOV) e reproduz com bypass ativo
- **Tab Código** — Código completo comentado, copiável
- **Simulação** — Botão para simular o comportamento do Tesla em andamento
- **Status em tempo real** — Mostra estado do AudioContext, Wake Lock e visibilidade

## Publicar no GitHub Pages

1. Faz fork ou clona este repositório
2. Vai a **Settings → Pages**
3. Em **Source**, escolhe `Deploy from a branch`
4. Escolhe `main` / `root`
5. O site fica disponível em `https://SEU-USUARIO.github.io/tesla-video`

## ⚠️ Aviso Legal

Este projeto é apenas para fins educativos e de demonstração técnica.
**Não uses o ecrã do Tesla para ver vídeo enquanto conduzes.** Em Portugal e na maioria dos países, é ilegal ter ecrãs de vídeo visíveis ao condutor em andamento.

---

Desenvolvido com ❤️ — Ficheiro único HTML, sem dependências, sem servidor.
