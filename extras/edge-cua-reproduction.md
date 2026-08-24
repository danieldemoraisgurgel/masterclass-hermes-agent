# Reprodução validável: Edge + CUA + DevTools remoto

Este roteiro reproduz o fluxo do Edge controlado pelo MCP `windows-cua` a partir de um gateway Linux. Ele usa um perfil separado, mas o CUA o anexa como `existing_profile`.

> **Segurança:** este roteiro inicia o Edge com `--remote-debugging-address=0.0.0.0`. A porta TCP 9222 **deve** ter uma regra de firewall limitada ao IP Tailscale do gateway autorizado. Não exponha DevTools à internet ou a toda a rede: o endpoint pode acessar abas, cookies e sessões.

## Pré-requisitos

- O `cua-driver serve` roda na sessão gráfica do Windows com `--permission-mode standard --grant existing-profile`.
- O usuário autorizou explicitamente o binding `existing_profile`.
- O gateway Linux alcança o Windows pela tailnet.
- O Microsoft Edge oficial está instalado em `Program Files` e a pasta de perfil separada será `C:\Temp\HermesEdgeCDP`.

## 1. Iniciar o Edge na sessão gráfica

O `launch_app` do CUA bloqueia flags Chromium diretas. Quando o agente só tiver o MCP `windows-cua`, inicie o PowerShell via CUA usando `-EncodedCommand` UTF-16LE com este payload exato:

```powershell
Start-Process -FilePath 'C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe' -ArgumentList @('--remote-debugging-address=0.0.0.0','--remote-debugging-port=9222','--user-data-dir=C:\Temp\HermesEdgeCDP')
```

Para execução manual na sessão gráfica:

```powershell
New-Item -ItemType Directory -Force 'C:\Temp\HermesEdgeCDP' | Out-Null
Start-Process -FilePath 'C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe' `
  -ArgumentList '--remote-debugging-address=0.0.0.0','--remote-debugging-port=9222','--user-data-dir=C:\Temp\HermesEdgeCDP'
```

Confirme o processo e o listener antes de testar a rede:

```powershell
Get-CimInstance Win32_Process -Filter "Name='msedge.exe'" |
  Where-Object { $_.CommandLine -like '*HermesEdgeCDP*' } |
  Select-Object ProcessId, CommandLine

Get-NetTCPConnection -LocalPort 9222 -State Listen |
  Select-Object LocalAddress, LocalPort, OwningProcess

Invoke-RestMethod http://127.0.0.1:9222/json/version
Invoke-RestMethod http://127.0.0.1:9222/json/list
```

A escuta esperada é `0.0.0.0:9222` (ou equivalente IPv6). O teste HTTP local usa loopback e deve retornar JSON; se falhar, corrija o processo Edge antes de investigar Tailscale, firewall ou portproxy.

## 2. Restringir o acesso de rede

No PowerShell elevado do Windows, crie ou valide uma regra que permita apenas o gateway. Substitua os exemplos pelos IPs Tailscale reais do seu ambiente:

```powershell
New-NetFirewallRule `
  -DisplayName 'Edge DevTools via Tailscale' `
  -Direction Inbound `
  -Action Allow `
  -Protocol TCP `
  -LocalAddress 100.116.151.102 `
  -LocalPort 9222 `
  -RemoteAddress 100.79.185.92 `
  -Profile Any

Get-NetFirewallRule -DisplayName 'Edge DevTools via Tailscale'
```

Como o Edge escuta em todas as interfaces, não é necessário `netsh interface portproxy` para este modo. Se houver um portproxy antigo, ele é redundante e pode ser removido depois de confirmar que o acesso direto pelo IP Tailscale funciona.

## 3. Validar o endpoint a partir do gateway

```bash
curl --fail --silent --show-error --max-time 10 \
  http://100.116.151.102:9222/json/list | \
  jq -r '.[] | select(.type == "page") | [.title, .url, .webSocketDebuggerUrl] | @tsv'
```

A resposta precisa ter ao menos uma entrada `type: page` com título, URL e `webSocketDebuggerUrl`. Isso prova somente a conectividade Edge/CDP; não substitui a autorização do CUA.

## 4. Fazer o binding pelo CUA

No **mesmo transporte MCP**:

1. execute `list_windows` e obtenha `pid` e `window_id` atuais da janela Edge;
2. chame `browser_prepare` com `strategy: {kind: existing_profile}` e os identificadores exatos;
3. chame imediatamente `get_browser_state`;
4. prossiga apenas se `binding_quality=exact`, `mutation_allowed=true` e `endpoint_access_class=existing_profile_approved`;
5. depois de cada mutação, capture um novo estado; se o Edge reiniciar, redescubra PID e janela.

A sequência verificável é:

```text
list_windows → browser_prepare → get_browser_state → browser_navigate → novo get_browser_state
```

## Diagnóstico

- Endpoint remoto falha e o endpoint local falha: Edge não iniciou com os argumentos corretos ou há um perfil antigo bloqueando `C:\Temp\HermesEdgeCDP`.
- Endpoint local responde e o remoto falha: valide firewall, IP Tailscale e a rota de rede.
- `browser_binding_stale`: execute `list_windows` de novo; não reutilize PID/window_id antigos.
- `browser_consent_required`: reinicie o daemon com `serve --permission-mode standard --grant existing-profile` e repita a autorização/binding.
- `browser_route_unavailable`: não use `isolated_new`; use a instância Edge já iniciada com o perfil separado e `existing_profile` autorizado.
