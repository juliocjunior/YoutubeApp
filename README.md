# YouTube Desktop

Aplicativo Windows para assistir YouTube numa janela própria, sem abrir navegador.

## O que é

- App nativo WinForms (C#) hospedando o **Microsoft Edge WebView2** (mesmo motor do Edge, sem abas/extensões)
- Bloqueio de anúncios (domínios de anúncio + pulo automático, silenciamento e avanço instantâneo dos anúncios do player)
- Watchdog: se a página travar, o app detecta e recarrega sozinho (registro em `%LOCALAPPDATA%\YouTubeApp\app.log`)
- Login Google com fallback de user-agent (`ua_mode=chrome`)
- Tela cheia (F11 + botão do player), zoom, controle de velocidade do vídeo (teclas d/s/r) e atalhos de teclado
- Sempre abre maximizado (tela cheia em modo janela, com barra de título)
- Lembra posição/tamanho/zoom da janela (settings.ini em `%LOCALAPPDATA%\YouTubeApp`)
- O vídeo continua tocando quando a janela é minimizada (como um player de música)
- Instância única, links `target=_blank` abrem na mesma janela

## Estrutura

```
YouTubeApp\
├─ src\
│  ├─ Program.cs            código do aplicativo (C# 5, compatível com csc do .NET Framework 4.8)
│  ├─ app.manifest          DPI awareness (PerMonitorV2)
│  ├─ YouTube.exe.config    WinForms PerMonitorV2 + supportedRuntime 4.8
│  └─ LEIA-ME.txt           instruções para o usuário final (copiado para dist\)
├─ tools\
│  └─ build.ps1             build: baixa DLLs do WebView2 (NuGet), gera icon.ico, compila com csc
├─ lib\                     DLLs do WebView2 baixadas automaticamente (Core, WinForms, WebView2Loader)
├─ assets\                  icon.ico gerado automaticamente pelo build
└─ dist\                    SAÍDA: YouTube.exe + DLLs + LEIA-ME.txt
```

## Como compilar

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File tools\build.ps1
```

Opções:

- `-Clean` — apaga `lib\`, `dist\` e o ícone e refaz tudo do zero
- `-Shortcut` — cria um atalho "YouTube" no Desktop automaticamente

Requisitos: Windows 10/11 x64 com .NET Framework 4.8 (nativo) e o
Microsoft Edge WebView2 Runtime (já vem instalado no Windows 11 e na
maioria dos Windows 10 atualizados). Internet apenas durante o build
(para baixar o pacote NuGet do WebView2).

## Teste automatizado

```powershell
dist\YouTube.exe --selftest
```

Roda, navega até o YouTube, valida carregamento e reprodução de vídeo, grava
o resultado em `%LOCALAPPDATA%\YouTubeApp\selftest.log` e sai sozinho.

## Configuração do usuário

`%LOCALAPPDATA%\YouTubeApp\settings.ini` (criado no primeiro fechamento):

| chave | padrão | descrição |
|---|---|---|
| `adblock` | 1 | bloquear + pular anúncios |
| `hiderecs` | 1 | `1` esconde a coluna lateral de recomendados só em `/watch`, `0` mostra normal (atalho `Ctrl+H` alterna na hora) |
| `maxres` | 0 | `0` = qualidade livre; `360`/`480`/`720`/`1080` = teto de resolução só em `/watch` (exige reiniciar o app; nunca aumenta a qualidade, só limita) |
| `zoom` | 1.0 | zoom salvo |
| `ua_mode` | auto | `chrome` remove o token Edg/ do user-agent (contorno de login Google) |
| `devtools` | 0 | habilita DevTools (F12) |

## Notas técnicas

- Compilado com `csc.exe` do .NET Framework 4.8 (C# 5) — sem Visual Studio / .NET SDK
- WebView2 carregado de `lib\` (net462) + `WebView2Loader.dll` (win-x64)
- Bloqueio por `WebResourceRequested` (doubleclick, googlesyndication, pagead, etc.)
- Pulo de anúncios via script injetado (`AddScriptToExecuteOnDocumentCreatedAsync`)
- `--disable-features=CalculateNativeWinOcclusion` para o vídeo não pausar com a janela minimizada
"# YoutubeApp" 
