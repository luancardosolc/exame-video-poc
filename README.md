# exame-video-poc

POC visual/técnica para validar experiência de vídeo estilo Instagram/TikTok/Twitch usando apenas HTML + CSS + JS puro.

## Como rodar

Abra `index.html` diretamente no browser:

```bash
open index.html
# ou
python3 -m http.server 8080
# depois acesse http://localhost:8080
```

> **Nota:** O YouTube IFrame API pode bloquear quando aberto via `file://`. Prefira um servidor local (python ou Live Server do VSCode).

## O que esta POC valida

| Validação | Status |
|-----------|--------|
| Fake fullscreen (sem `requestFullscreen()`) | ✅ |
| Vídeo como background layer | ✅ |
| Overlays por cima do vídeo | ✅ |
| Overlays clicáveis | ✅ (com `pointer-events: none` no iframe) |
| Experiência mobile vertical | ✅ |
| Badge AO VIVO com pulse | ✅ |
| Reactions animados | ✅ |
| Troca dinâmica entre modos (sem reload) | ✅ |
| Safe areas (notch / Dynamic Island) | ✅ |
| `100dvh` para evitar bug Safari iOS | ✅ |
| Debug panel | ✅ |
| Autoplay fallback | ✅ |

## Modos disponíveis

- **YT Vídeo** — YouTube IFrame API, vídeo normal
- **YT Live** — YouTube IFrame API, live stream
- **Vimeo** — iframe Vimeo com `background=1`

## Arquitetura de camadas (z-index)

```
APP LAYER     (z: 10)  — fake fullscreen container
VIDEO LAYER   (z: 20)  — iframe como background
OVERLAY LAYER (z: 30)  — UI social (header, footer, sidebar, reactions)
GESTURE LAYER (z: 40)  — captura gestos (pointer-events: none)
DEBUG PANEL   (z: 50)  — sempre no topo
```

## Limitações conhecidas

### iframe vs player nativo

- Qualquer clique dentro do iframe é capturado por ele, não pelo HTML pai
- Solução: `pointer-events: none` no iframe + controles via YouTube IFrame API
- YouTube ToS restringe overlays sobre o player
- autoplay exige `muted=1` em todos os browsers modernos
- iOS Safari exige `playsinline=1` ou abre em fullscreen nativo

### Por que apps de produção usam player nativo

| Iframe | ExoPlayer (Android) / AVPlayer (iOS) |
|--------|--------------------------------------|
| Overlays com restrições cross-origin | Overlay nativo, sem restrições |
| Autoplay limitado por política do browser | Autoplay total |
| Sem controle de qualidade granular | Controle total de ABR/qualidade |
| Dependente de ToS YouTube/Vimeo | Independente |

## Próximos passos / migração para React Native

1. Substituir iframe do YouTube por `react-native-youtube-iframe`
2. Substituir iframe do Vimeo por `react-native-video` ou `expo-av`
3. Overlays → componentes React Native com `position: absolute`
4. Reactions → Animated API do RN ou `react-native-reanimated`
5. Safe areas → `react-native-safe-area-context`
6. Gestos → `react-native-gesture-handler`
