# Tutorial: controlar o Windows pelo Hermes remoto com CUA via MCP sobre SSH

> **Cenário:** gateway Hermes em um VPS Linux e área de trabalho Windows acessada pela tailnet Tailscale.
>
> **Resultado:** o gateway chama o `cua-driver` no Windows por SSH, mas as ações de interface são executadas na sessão gráfica interativa do usuário Windows — não na Session 0 do serviço SSH.

---

## 1. Arquitetura final

```text
Hermes Gateway (Linux / VPS)
  └─ MCP stdio: windows-cua
       └─ wrapper /root/.hermes/bin/windows-cua-mcp
            └─ SSH via Tailscale → danie@100.116.151.102
                 └─ cua-driver mcp --socket \\.\pipe\cua-driver
                      └─ cua-driver serve na sessão gráfica Windows (Session 1+)
```

O gateway não usa o `computer_use` nativo para controlar o Windows, pois esse tool nativo permanece no host Linux. O MCP `windows-cua` expõe os tools do CUA que rodam no Windows: `list_windows`, `get_window_state`, `click`, `type_text`, `press_key`, `launch_app` e outros.

---

## 2. Pré-requisitos

- VPS Linux com Hermes Agent configurado.
- Máquina Windows conectada à mesma tailnet.
- Usuário Windows com uma sessão gráfica aberta (console ou RDP).
- `cua-driver` instalado no Windows.
- OpenSSH Server habilitado no Windows.
- Chave pública do VPS autorizada para o usuário Windows.

> **Importante:** o OpenSSH no Windows executa comandos na Session 0. Sem o daemon do CUA iniciado na sessão gráfica, `list_windows` retorna uma lista vazia.

---

## 3. Preparar o CUA no Windows

No PowerShell da sessão gráfica do Windows:

```powershell
cua-driver autostart enable
cua-driver autostart kick
query session
cua-driver status
cua-driver doctor
```

A validação esperada inclui:

```text
interactive session: session 1 has an attached interactive desktop
UI Automation: CoCreateInstance(CUIAutomation) succeeded
```

A sessão `danie` foi confirmada como Session 1 ativa. O daemon ficou disponível no pipe:

```text
\\.\pipe\cua-driver
```

### Corrigir incompatibilidade de versões do daemon

Se o cliente CUA reportar algo semelhante a:

```text
incompatible daemon: contract version 0.6.0 does not match SDK 0.7.0
```

recrie a tarefa autostart usando a versão atual do binário:

```powershell
$driver = 'C:\Users\danie\.cua-driver\packages\releases\0.21.0-x86_64-pc-windows-msvc\cua-driver.exe'
& $driver autostart disable
& $driver stop
& $driver autostart enable
& $driver autostart kick
& $driver status
```

---

## 4. Habilitar SSH no Windows

Em um PowerShell elevado (Administrador):

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -ErrorAction SilentlyContinue | Enable-NetFirewallRule
```

No VPS, valide a porta pela tailnet:

```bash
timeout 5 bash -c '</dev/tcp/100.116.151.102/22'
```

---

## 5. Autorizar a chave SSH do VPS

A chave pública do VPS deve ser adicionada ao OpenSSH do Windows.

### Conta Windows comum

Para uma conta que **não** é administradora, use:

```powershell
New-Item -ItemType Directory -Force "$HOME\.ssh" | Out-Null
# Adicione a chave pública do VPS em $HOME\.ssh\authorized_keys
```

Em seguida, aplique ACLs restritas:

```powershell
$user = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
icacls "$HOME\.ssh" /inheritance:r /grant:r "$($user):(OI)(CI)F" "SYSTEM:(OI)(CI)F"
icacls "$HOME\.ssh\authorized_keys" /inheritance:r /grant:r "$($user):F" "SYSTEM:F"
```

### Conta Windows administradora

O usuário usado nesta instalação (`danie`) pertence ao grupo local de administradores. Por causa deste bloco padrão do `sshd_config`:

```text
Match Group administrators
    AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

O OpenSSH ignora `C:\Users\danie\.ssh\authorized_keys` e exige o arquivo global. Em PowerShell elevado:

```powershell
$keyPath = Join-Path $env:ProgramData 'ssh\administrators_authorized_keys'
# Grave a chave pública do VPS em $keyPath
icacls $keyPath /inheritance:r /grant:r '*S-1-5-32-544:F' '*S-1-5-18:F'
Restart-Service sshd
```

Os SIDs usados acima são:

- `S-1-5-32-544`: grupo local Administradores;
- `S-1-5-18`: SYSTEM.

### Validar do VPS

```bash
ssh -i /root/.ssh/id_rsa -o IdentitiesOnly=yes -o BatchMode=yes \
  danie@100.116.151.102 exit
```

---

## 6. Criar o wrapper MCP no VPS

Arquivo criado no VPS: `/root/.hermes/bin/windows-cua-mcp`

```bash
#!/usr/bin/env bash
# MCP stdio bridge: VPS → SSH/Tailscale → Windows interactive CUA daemon.
exec ssh \
  -i /root/.ssh/id_rsa \
  -o IdentitiesOnly=yes \
  -o BatchMode=yes \
  -o ConnectTimeout=10 \
  -o ServerAliveInterval=30 \
  -o StrictHostKeyChecking=yes \
  danie@100.116.151.102 \
  'powershell.exe -NoProfile -Command "& \"C:\Users\danie\.cua-driver\packages\releases\0.21.0-x86_64-pc-windows-msvc\cua-driver.exe\" mcp --socket \"\\.\pipe\cua-driver\""'
```

Depois:

```bash
chmod 700 /root/.hermes/bin/windows-cua-mcp
bash -n /root/.hermes/bin/windows-cua-mcp
```

> Ajuste o caminho do binário se uma nova versão do CUA instalar em outro diretório.

---

## 7. Registrar o MCP no Hermes do VPS

```bash
hermes mcp add windows-cua \
  --command /root/.hermes/bin/windows-cua-mcp \
  --connect-timeout 30
```

Na pergunta de seleção, habilite todos os tools (`Y`).

Valide a integração:

```bash
hermes mcp test windows-cua
hermes mcp list
```

Resultado validado nesta instalação:

```text
✓ Connected
✓ Tools discovered: 57
windows-cua  /root/.hermes/bin/windows...  all  ✓ enabled
```

O Hermes informa que uma **nova sessão** deve ser iniciada para que os tools MCP apareçam no chat.

---

## 8. Teste técnico direto pelo SSH

Exemplo para confirmar que o processo SSH encaminha chamadas ao daemon da sessão gráfica:

```bash
ssh -i /root/.ssh/id_rsa -o IdentitiesOnly=yes -o BatchMode=yes \
  danie@100.116.151.102 \
  'powershell.exe -NoProfile -Command "& \"C:\Users\danie\.cua-driver\packages\releases\0.21.0-x86_64-pc-windows-msvc\cua-driver.exe\" call list_windows --socket \"\\.\pipe\cua-driver\""'
```

A validação retornou janelas reais da sessão Windows, incluindo Hermes, PowerShell, Edge, WhatsApp e Explorador de Arquivos.

---

## 9. Prompts executados para validar o MCP

### Abrir a Calculadora

Prompt enviado ao Hermes em uma sessão nova:

```text
Use exclusivamente o MCP windows-cua para abrir o aplicativo Calculadora no Windows remoto. Em seguida, use windows-cua list_windows para confirmar que uma janela da Calculadora está aberta. Responda em português com a confirmação e o título da janela.
```

Resultado validado:

```text
Calculadora aberta.
Título: “Calculadora”
window_id: 592718
Estado: visível e não minimizada.
```

### Calcular 2 + 2 pela interface da Calculadora

Prompt enviado ao Hermes em outra sessão nova:

```text
Use exclusivamente o MCP windows-cua na janela existente da Calculadora do Windows remoto. Realize 2 + 2 pela interface da Calculadora e leia/verifique o resultado exibido. Responda apenas com o resultado.
```

Resultado validado pela interface do aplicativo:

```text
4
```

---

## 10. Uso diário

Após abrir uma nova sessão do Hermes conectada ao gateway VPS, use instruções explícitas, por exemplo:

```text
Use o MCP windows-cua para abrir a Calculadora no meu Windows.
```

```text
Use o MCP windows-cua para listar as janelas abertas no Windows.
```

```text
Use o MCP windows-cua para abrir o Explorador de Arquivos e navegar até C:\temp.
```

---

## 11. Diagnóstico rápido

### O MCP conecta, mas não encontra janelas

No Windows:

```powershell
query session
cua-driver status
cua-driver doctor
```

Confirme que o daemon está em Session 1+ e que há desktop interativo anexado.

### A conexão SSH falha

No VPS:

```bash
ssh -vvv -i /root/.ssh/id_rsa -o IdentitiesOnly=yes danie@100.116.151.102 exit
```

Para contas administradoras, confirme que a chave está em:

```text
C:\ProgramData\ssh\administrators_authorized_keys
```

### Erro de contrato/versionamento do CUA

Pare e recrie o daemon/tarefa com a versão atual do driver, conforme a seção 3.

### O MCP não aparece no chat

```bash
hermes mcp test windows-cua
hermes mcp list
```

Em seguida, abra uma nova sessão Hermes. A descoberta de tools MCP acontece no início de cada sessão.

---

## Referência

- CUA: https://cua.ai/docs/how-to-guides/driver/windows-ssh
- Hermes MCP: https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp
- Hermes Computer Use: https://hermes-agent.nousresearch.com/docs/user-guide/features/computer-use
