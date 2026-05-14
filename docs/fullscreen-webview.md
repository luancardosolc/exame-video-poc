# Fullscreen em WebView

Este documento descreve o comportamento esperado do botão de fullscreen da POC quando ela roda dentro do app Exame Plus via `react-native-webview`.

## Contexto

A POC usa um fullscreen "fake", parecido com experiências de Instagram, TikTok e players sociais:

- o conteúdo ocupa `100vw` x `100dvh`;
- o vídeo fica como camada de fundo;
- os controles, textos, reações e CTAs ficam em overlays HTML por cima;
- não é usado `requestFullscreen()`;
- no iOS, os players usam `playsinline=1` para evitar o fullscreen nativo automático.

Esse modo funciona bem para vídeos verticais. Para vídeos em paisagem, porém, manter a WebView em portrait deixa o vídeo pequeno, com muito espaço acima e abaixo.

## Contrato com o app nativo

O botão de fullscreen da POC não tenta girar a tela diretamente pela Web API, porque `screen.orientation.lock()` não é confiável em WebViews iOS.

Em vez disso, a POC envia mensagens para o app React Native:

```js
window.ReactNativeWebView.postMessage('enterFullscreen');
window.ReactNativeWebView.postMessage('exitFullscreen');
```

No app Exame Plus, `WebViewOverlay` recebe essas mensagens no `onMessage` e delega a rotação para `expo-screen-orientation`:

- `enterFullscreen`: trava a orientação em landscape;
- `exitFullscreen`: sai do modo fullscreen e deve voltar para o comportamento de portrait do app;
- `close`: fecha a WebView e também deve limpar qualquer lock de orientação ativo.

## Comportamento esperado por plataforma

- Android: ao tocar no botão, a tela gira para landscape; ao tocar novamente, volta para portrait.
- iPhone: ao tocar no botão, a tela deve girar para landscape; ao tocar novamente, deve voltar para portrait.
- iPad: ao tocar no botão, a tela deve girar para landscape quando o binário iOS suportar lock de orientação em iPad.

## Observações conhecidas

- A POC mantém um estado interno `isFullscreen` apenas para alternar o botão e escolher qual mensagem enviar.
- A rotação real é responsabilidade do app nativo.
- No iPad, o lock de orientação via Expo/iOS depende da configuração nativa do app. Se o app permitir Split View, o iPad pode ignorar locks de orientação.
- Ao sair do fullscreen, apenas "desbloquear" a orientação pode não forçar retorno visual para portrait no iOS; o app pode precisar aplicar explicitamente um lock portrait antes de liberar ou manter o app em portrait.

