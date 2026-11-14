# FileServerAuditor

[![Release](https://img.shields.io/github/v/release/k4r3k4/FileServer-Auditor?sort=semver)](https://github.com/k4r3k4/FileServer-Auditor/releases)
[![License: MIT](https://img.shields.io/github/license/k4r3k4/FileServer-Auditor?color=blue)](LICENSE)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6?logo=windows)
![.NET Framework](https://img.shields.io/badge/.NET%20Framework-4.8-512BD4?logo=dotnet)

Antes de continuar, saiba que esse e um projeto desenvolvido 100% com auxilio
de ferramentas de IA. VIBE CODING. Avalie o uso dele em cenários controlados.

Auditor **somente leitura** de permissões de acesso em compartilhamentos de
arquivos Windows (NTFS + SMB), com interface gráfica e modo de linha de comando.
O foco é responder “quem acessa o quê?” **sem alterar nada** no ambiente
auditado.

> A ferramenta é **somente leitura**. Valide o resultado em laboratório com
> contas e shares reais antes de usá-lo em decisões de produção.

O programa responde a duas perguntas:

1. **Usuário → pastas:** a quais pastas e arquivos um usuário tem acesso efetivo?
2. **Pasta → usuários:** quais ACEs (Allow/Deny) existem em cada pasta e,
   opcionalmente, qual é o acesso efetivo de um usuário foco?

O resultado pode ser navegado por uma árvore de pastas, ordenado na grade e
exportado em **CSV**, **HTML** ou **JSON**.

## Índice

- [Download e instalação](#download-e-instalação)
- [Recursos](#recursos)
- [Garantias de segurança](#garantias-de-segurança)
- [Requisitos](#requisitos)
- [Uso rápido (interface)](#uso-rápido-interface)
- [Uso rápido (linha de comando)](#uso-rápido-linha-de-comando)
- [Documentação](#documentação)
- [Limitações conhecidas](#limitações-conhecidas)
- [Licença](#licença)
- [Avisos e responsabilidade](#avisos-e-responsabilidade)

## Download e instalação

A forma mais rápida é baixar o executável pronto na aba
**[Releases](https://github.com/k4r3k4/FileServer-Auditor/releases/latest)**.

| Asset | Descrição |
|---|---|
| `FileServerAuditorV4.exe` | Aplicativo portátil para Windows (.NET Framework 4.8) |
| `SHA256SUMS.txt` | Hash SHA-256 do executável, para verificação de integridade |

1. Baixe **os dois arquivos** da mesma release.
2. Confira a integridade (recomendado):

   ```powershell
   Get-FileHash .\FileServerAuditorV4.exe -Algorithm SHA256
   ```

   ```cmd
   certutil -hashfile FileServerAuditorV4.exe SHA256
   ```

   O hash calculado deve ser **idêntico** ao registrado em `SHA256SUMS.txt`.
3. Execute `FileServerAuditorV4.exe`. Não há instalador: o programa é portátil.
4. Se o Windows SmartScreen exibir um aviso, isso ocorre porque o executável
   **não é assinado digitalmente**. Use “Mais informações → Executar assim
   mesmo” apenas se confiar na origem.

**Requisitos:** Windows 10/11 ou Windows Server com **.NET Framework 4.8**.

## Recursos

- Dois modos de auditoria: **usuário → pastas** e **pasta → usuários**.
- Cálculo de **acesso efetivo** combinando NTFS e a ACL do share (quando acessível).
- Modelo de avaliação próprio: Deny prevalece, herança, `InheritOnly`,
  DACL `NULL`/vazia e direitos implícitos do proprietário.
- **Cadeia de acesso**: mostra quais ACEs contribuíram para o acesso efetivo.
- Grupos do AD: `tokenGroups` em busca **Base** no objeto, grupo primário,
  `sIDHistory`, grupos locais do servidor e grupos citados nas ACEs; fallback
  recursivo por `memberOf` quando `tokenGroups` não está disponível.
- Leitura de grupos grandes do AD por faixas (`member;range=...`), sem o limite
  de ~1.500 membros.
- Resolução de identidades em **florestas multi-domínio** via LDAP/Global Catalog.
- Varredura em **streaming**, com cancelamento e progresso.
- Destaques de risco: acesso total, erro de leitura, acesso efetivo e
  “acesso sem pasta pai” (com ancestrais marcados).
- Notas de **junção/DFS**: pontos de reanálise são registrados sem serem seguidos.
- Exportação **CSV**, **HTML** e **JSON** com observações e resumo.
- **Diff de relatórios**: botão “Comparar auditorias” na interface (e função
  `ReportDiff`) para comparar dois exports CSV/JSON, com aviso quando as
  identidades forem diferentes. Lista adicionados, removidos e alterados.
- **Credenciais de domínio** opcionais, com opção de mantê-las criptografadas
  em memória via **DPAPI** (não são gravadas em disco).
- **Modo console/CLI** para automação e Task Scheduler.

## Garantias de segurança

- **Somente leitura:** nenhuma API de alteração de ACL, atributos ou conteúdo é
  usada sobre o caminho auditado.
- Escritas permitidas apenas em locais sob controle do usuário:
  - exportação CSV/HTML/JSON no caminho escolhido no diálogo “Salvar”;
  - comparação de auditorias salva em `.txt` no caminho escolhido;
  - log de falhas em `%LOCALAPPDATA%\FileServerAuditor`.
- A senha de domínio fica apenas em memória e é descartada ao fim da varredura.
  Quando “Lembrar credenciais” está ativo, usuário e senha são mantidos
  criptografados com DPAPI durante a sessão.
- CSV neutraliza fórmulas (`=`, `+`, `-`, `@`, TAB/CR/LF) contra CSV injection.
- Nenhum dado é enviado para a Internet; não há telemetria.
- Executável declarado como `asInvoker` no manifesto (não força elevação).
- **Não requer variáveis de ambiente** nem arquivos de configuração/segredos.

## Requisitos

- Windows 10/11 ou Windows Server com **.NET Framework 4.8**.
- Conta com permissão de leitura das ACLs NTFS dos caminhos auditados.
- Para ler a ACL do **share** (`NetShareGetInfo` nível 502), é necessário ser
  administrador no servidor. Sem isso, o NTFS continua sendo auditado e o share
  aparece como “Não verificado”.
- Opcional: credenciais de domínio alternativas
  (`LogonUser` + `LOGON32_LOGON_NEW_CREDENTIALS` com impersonation na thread de
  varredura).

## Uso rápido (interface)

1. Informe a raiz: `\\SERVIDOR\Compartilhamento` (preferível) ou uma unidade
   mapeada (`X:\Pasta`).
2. Escolha o modo e a identidade (`DOMINIO\usuario`). Em máquina ingressada no
   domínio, o domínio é detectado automaticamente.
3. Opções: “Somente com acesso”, “Expandir grupos”, “Incluir arquivos”,
   credenciais de domínio e “Lembrar credenciais nesta sessão (DPAPI)”.
4. Clique em **Auditar**. O botão **Cancelar** interrompe a varredura.
5. Clique em um nó da árvore para filtrar a grade; clique no cabeçalho para
   ordenar; duplo clique copia o caminho.
6. Exporte em **CSV**, **HTML** ou **JSON**.

## Uso rápido (linha de comando)

Executar sem argumentos abre a interface gráfica. Com argumentos, o programa
roda em modo console (útil para automação e Task Scheduler):

```cmd
FileServerAuditorV4.exe /root:\\SERVIDOR\Dados /user:EMPRESA\joao.silva ^
  /mode:usuario /export:resultado.csv /format:csv
```

```cmd
FileServerAuditorV4.exe /root:\\SERVIDOR\Dados /mode:pasta ^
  /identity:EMPRESA\joao.silva /export:resultado.json /format:json
```

| Opção | Descrição |
|---|---|
| `/root:<caminho>` | Raiz da auditoria (`\\SERVIDOR\Share` ou `X:\Pasta`). Obrigatório no modo console. |
| `/user:<identidade>` | Usuário no formato `DOMINIO\usuario` (modo usuário). |
| `/mode:usuario\|pasta` | Modo de consulta (padrão: `usuario`). |
| `/identity:<id>` | Usuário foco no modo pasta. |
| `/export:<caminho>` | Caminho do arquivo de saída. |
| `/format:csv\|html\|json` | Formato de saída (padrão: `csv`). |
| `/creds` | Solicita senha e usa credenciais de domínio. |
| `/remember` | Mantém as credenciais desta sessão criptografadas com DPAPI. |
| `/?` ou `/help` | Exibe a ajuda. |

O código de saída é `0` em caso de sucesso e `1` em caso de erro.

## Documentação

- [Manual de uso](docs/LEIA-ME.txt)
- [Política de segurança](SECURITY.md)
- [Aviso legal](DISCLAIMER.md)
- [Changelog](CHANGELOG.md)
- [Licença](LICENSE)

## Limitações conhecidas

- DFS, links simbólicos e pontos de junção **não** são seguidos; pontos de
  reanálise são registrados, mas ignorados para evitar loops e fuga do escopo.
- ACEs de objeto (`ObjectAce`) não são avaliadas; a linha registra a limitação.
- Shares administrativos (`C$`, `ADMIN$`, `IPC$`) podem retornar descritor nulo.
- A leitura da ACL do share depende de permissão administrativa no servidor.
- O acesso efetivo é uma avaliação do modelo de ACL; deve ser validado com uma
  conta real em laboratório antes de decisões de produção.

## Licença

Este projeto é licenciado sob a licença **MIT**. Consulte [LICENSE](LICENSE).

## Avisos e responsabilidade

- O FileServerAuditor é fornecido **no estado em que se encontra**, sem
  garantias de qualquer tipo. O uso é por conta e risco do usuário.
- Não substitui uma auditoria oficial de segurança. Valide o resultado em
  laboratório com contas reais antes de decisões de produção.
- É uma ferramenta de auditoria **somente leitura**: não altera ACLs,
  atributos, conteúdo ou configurações de compartilhamento.
- A senha de domínio fica apenas em memória e é descartada ao fim da varredura.
- Não há telemetria nem envio de dados para a Internet.

Veja [DISCLAIMER.md](DISCLAIMER.md) para o aviso legal completo.
