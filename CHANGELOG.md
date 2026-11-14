# Changelog

Todas as mudanças relevantes deste projeto são registradas aqui.
O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/)
e o projeto usa [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [4.0.10] - 2026-11-13

### Corrigido
- Resolução de grupos aninhados no AD: `tokenGroups` passou a ser lido com
  escopo **Base** sobre o objeto do usuário, após localizar a entrada por
  `sAMAccountName`. Quando a consulta de `tokenGroups` falha, o código recorre
  à expansão recursiva de `memberOf`. Corrige o falso negativo de acesso
  concedido por grupo aninhado em NTFS e no modo Pasta.
- Consultas LDAP de grupos agora respeitam `UseGlobalCatalog` (`GC://`).

## [4.0.9] - 2026-11-12

### Adicionado
- Comparação de auditorias na interface: o botão "Comparar auditorias"
  seleciona dois exports CSV/JSON, extrai a identidade dos arquivos, avisa
  (com confirmação) quando são de usuários diferentes e exibe o diff em uma
  janela com opção de salvar o resultado em `.txt`.
- Download do executável portátil e verificação de integridade SHA-256 no
  README.

### Corrigido
- Diff de CSV real: os cabeçalhos gerados na exportação agora são reconhecidos,
  evitando campos vazios e alterações não detectadas na comparação.
- Destaque de "acesso sem pasta pai" com a opção "Somente com acesso":
  ancestrais sem linha na grade continuam na árvore e recebem o destaque roxo;
  a subpasta acessível recebe o laranja.

## [4.0.8] - 2026-11-11

### Adicionado
- Seção de uso por linha de comando no README.

### Alterado
- README reescrito e documentação alinhada aos recursos atuais (JSON, CLI,
  DPAPI, diff e multi-domínio).

### Corrigido
- Verificação de diff de relatórios em JSON.

## [4.0.7] - 2026-11-11

### Adicionado
- Suporte a multi-domínio via LDAP/Global Catalog: resolução de identidades em
  domínios adicionais confiáveis quando o domínio local não resolve.

## [4.0.6] - 2026-11-11

### Adicionado
- Diff de relatórios: compara dois exports CSV ou JSON e lista adicionados,
  removidos e alterados campo a campo.

## [4.0.5] - 2026-11-11

### Adicionado
- Modo console/CLI: execução via linha de comando para automação e Task
  Scheduler, sem abrir a interface. Suporta `/root`, `/user`, `/mode`,
  `/identity`, `/export`, `/format` (csv|html|json), `/creds`, `/remember` e
  `/?`. Retorna código de saída (0 = sucesso, 1 = erro).

## [4.0.4] - 2026-11-11

### Adicionado
- Coluna "Via/cadeia" de acesso: mostra quais ACEs contribuíram para o acesso
  efetivo (principal, tipo Allow/Deny, direitos) no modo usuário.

## [4.0.3] - 2026-11-11

### Adicionado
- Notas de junção/DFS: pontos de reanálise (junção, link simbólico, ponto de
  reanálise genérico) são detectados durante a varredura e aparecem como linha
  na grade com nota explicativa, sem serem seguidos (mantém a
  não-destrutividade).

## [4.0.2] - 2026-11-11

### Adicionado
- Exportação JSON com escape correto de caracteres especiais e estrutura
  `geradoEm`/`raiz`/`identidade`/`resumo`/`totalLinhas`/`linhas`.

### Corrigido
- Botão "Exportar JSON" com habilitação/desabilitação sincronizada com CSV e
  HTML; ajuste de layout do painel de ações.

## [4.0.1] - 2026-11-11

### Adicionado
- Criptografia de credenciais de domínio com DPAPI: usuário e senha podem ser
  mantidos criptografados durante a sessão, sem gravação em disco.

### Corrigido
- Sobreposição/corte de itens da interface em escalas de DPI elevadas (125%,
  150%); larguras de colunas, altura de linha e cabeçalho ajustados ao DPI.

## [4.0.0] - 2026-11-10

### Adicionado
- Direitos implícitos do proprietário (`READ_CONTROL` e `WRITE_DAC`) no
  cálculo de acesso efetivo, sem implicar leitura/escrita de conteúdo.
- Leitura de grupos grandes do AD por faixas (`member;range=...`), com limite
  de segurança por grupo.
- Teto de expansão de grupos: 200.000 linhas por varredura, com aviso único.

### Alterado
- Varredura em streaming: a barra de progresso fica indeterminada e o status
  mostra a contagem de itens.

### Corrigido
- Grade vazia, normalização de raiz UNC/unidade mapeada, ordenação por
  cabeçalho e destaque de "acesso sem pasta pai" propagado aos ancestrais.
- Log de falhas movido para `%LOCALAPPDATA%\FileServerAuditor`.
- Senha descartada ao fim da varredura.
- Resumo honesto no modo pasta sem foco (conta ACEs, não "acesso efetivo: 0").

### Segurança
- Neutralização de fórmulas no CSV (CSV injection).
- Avisos limitados a 20.000 itens.
- Nenhuma escrita em ACL, atributos ou conteúdo do caminho auditado.

## [3.1.0] - 2026-11-09

### Alterado
- Estabilização do protótipo: mapeamento de direitos genéricos, leitura de
  pipes, timeout, escape de argumentos e avisos de enumeração.
