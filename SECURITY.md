# Política de segurança

## Escopo

O FileServerAuditor é uma ferramenta de **auditoria somente leitura** de
permissões NTFS/SMB. Ela não altera ACLs, atributos, conteúdo ou configurações
de compartilhamento.

## Garantias de projeto

- **Sem escrita no alvo:** nenhuma chamada a `SetAccessControl`, `SetAccessRule`,
  `SetOwner`, `NetShareSetInfo` ou equivalente é feita sobre o caminho auditado.
- **Escritas permitidas:**
  - exportações CSV/HTML/JSON no caminho escolhido explicitamente no diálogo;
  - comparação de auditorias salva em `.txt` no caminho escolhido;
  - log de falhas em `%LOCALAPPDATA%\FileServerAuditor`.
- **Credenciais:** a senha de domínio fica apenas em memória, é usada para
  `LogonUser`/impersonation e é descartada no `finally` da varredura. Não é
  gravada em disco nem em log. Quando a opção "Lembrar credenciais nesta
  sessão" está marcada, usuário e senha são armazenados criptografados com
  DPAPI (Data Protection API, escopo CurrentUser) enquanto a sessão estiver
  ativa, evitando que o texto claro permaneça exposto nos campos do objeto.
- **Sem rede não solicitada:** os únicos acessos remotos são os caminhos
  auditados e consultas AD/SMB necessárias à resolução de identidades e ACLs.
  Não há telemetria nem envio de dados para a Internet.
- **CSV injection:** células iniciadas por `=`, `+`, `-`, `@`, TAB/CR/LF são
  prefixadas com `'` para não serem interpretadas como fórmula no Excel.
- **Princípio do menor privilégio:** o manifesto declara `asInvoker`; a
  elevação é decisão do operador (necessária apenas para ACL de share).

## Dados e privacidade

As exportações podem conter nomes de usuários, grupos, SIDs e caminhos. Trate
os relatórios como informação sensível e armazene-os de acordo com a política
da organização.

## Relatando vulnerabilidades

Não abra issue pública para falhas de segurança. Use o recurso de
**Security advisory** do GitHub (aba *Security* → *Report a vulnerability*)
ou entre em contato diretamente com o mantenedor (`k4r3k4`). Inclua:

- versão afetada;
- cenário e impacto;
- passos de reprodução;
- sugestão de correção, se houver.

## Versões suportadas

| Versão | Suporte |
|---|---|
| 4.x | Sim |
| 3.x e anteriores | Não |
