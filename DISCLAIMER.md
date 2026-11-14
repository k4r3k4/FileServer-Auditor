# Aviso legal

O FileServerAuditor é uma ferramenta de **auditoria somente leitura** de
permissões NTFS/SMB em file servers Windows. É fornecida **no estado em que
se encontra ("as is")**, sem garantias de qualquer tipo, expressas ou
implícitas, incluindo mas não se limitando a garantias de comercialização,
adequação a um propósito específico ou não violação.

O uso desta ferramenta é por **conta e risco do usuário**. Os autores e
mantenedores não se responsabilizam por danos diretos ou indiretos, perda de
dados, interrupção de negócios ou quaisquer outros prejuízos decorrentes do
uso ou incapacidade de usar o software.

## Garantias de projeto (não são garantias legais)

- Somente leitura: nenhuma API de alteração de ACL, atributos ou conteúdo é
  usada sobre o caminho auditado.
- A senha de domínio fica apenas em memória e é descartada ao fim da varredura.
- Não há telemetria nem envio de dados para a Internet.

## Limitações conhecidas

- DFS, links simbólicos e pontos de junção não são seguidos.
- ACEs de objeto (`ObjectAce`) não são avaliadas.
- A leitura da ACL do share exige permissão administrativa no servidor.
- O resultado é uma avaliação do modelo de ACL; deve ser validado em
  laboratório com contas reais antes de decisões de produção.

## Recomendação

Não utilize esta ferramenta diretamente em ambientes de produção sem antes
testar em laboratório com contas e shares dedicados.
