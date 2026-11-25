# Guia prático de performance para apps web

> Documento elaborado a partir de um reel com 31 recomendações de performance e expandido com orientação técnica. Não é uma lista para aplicar de uma vez: medir o gargalo real, mudar uma coisa por vez e comparar o antes e o depois é parte essencial do trabalho.

## Princípio de trabalho

Desempenho não é apenas a velocidade do servidor. Uma experiência lenta pode resultar de imagens pesadas, JavaScript excessivo, renderização bloqueada, chamadas repetidas, consultas ineficientes, distância de rede, dependências de terceiros ou má percepção de carregamento.

Antes de otimizar:

1. Defina a jornada importante: por exemplo, abrir a home, pesquisar, concluir uma compra ou gerar conteúdo.
2. Meça em dispositivo e rede realistas; um notebook rápido no Wi-Fi costuma esconder problemas.
3. Localize o maior gargalo com métricas, traces e perfil de renderização.
4. Faça uma mudança, publique-a com segurança e compare métricas reais.
5. Preserve a correção: uma otimização que mostra dados desatualizados, duplica uma cobrança ou prejudica acessibilidade não é uma melhoria.

## 1. Colocar skeletons de carregamento

Skeletons são representações provisórias do layout final: blocos para título, imagem e linhas de texto. Eles não encurtam a consulta ao servidor, mas ajudam a pessoa a entender que o conteúdo está chegando e reduzem a percepção de espera.

**Como aplicar:** reserve no CSS o espaço final de imagens, textos e cards; apresente um skeleton apenas enquanto a operação exceder um pequeno limiar; substitua-o sem mudar bruscamente a geometria da página.

**Cuidados:** não use skeleton como desculpa para uma tela permanentemente lenta. Evite animações pesadas ou excesso de brilho em aparelhos modestos. Para respostas quase instantâneas, o skeleton pode aparecer e desaparecer rápido demais, causando uma piscada pior que a espera.

## 2. Servir imagens em WebP ou AVIF

Imagens geralmente representam a maior parcela dos bytes baixados. WebP e AVIF reduzem o peso com pouca perda visual; AVIF tende a comprimir melhor, enquanto WebP é um excelente formato geral.

**Como aplicar:** gere variantes no upload ou no pipeline de mídia; ofereça dimensões responsivas com `srcset` e `sizes`; comprima de acordo com o uso; entregue miniaturas para cards e versões maiores apenas para visualização ampliada.

**Cuidados:** não envie uma imagem de 3000 px para uma área de 300 px. Preserve um fallback se houver público em navegadores legados. Defina largura e altura, ou `aspect-ratio`, para a página não saltar quando a imagem chegar.

## 3. Evitar chamadas duplicadas à API

É comum que cabeçalho, página e modal peçam o mesmo perfil ou configuração em paralelo. Isso multiplica custo, latência e disputa por recursos sem melhorar a interface.

**Como aplicar:** centralize o estado remoto em uma camada de consulta/cache; use uma chave consistente para cada recurso; deduplique requisições em andamento e reaproveite respostas recentes. Cancele ou invalide a busca quando uma navegação a tornar obsoleta.

**Cuidados:** cache não pode esconder mudanças importantes indefinidamente. Escolha uma janela de reuso compatível com cada dado: configurações públicas toleram mais tempo que saldo, estoque ou permissões.

## 4. Aplicar debounce em busca e autocomplete

Sem debounce, cada tecla pode disparar uma requisição. Ao digitar uma palavra de sete letras, sete buscas podem chegar ao servidor, embora só a última importe.

**Como aplicar:** mantenha a digitação responsiva, mas espere em geral 250–400 ms após a última tecla antes de iniciar a busca. Cancele a requisição anterior ou descarte a resposta se uma consulta mais nova já existir.

**Cuidados:** não use debounce para toda interação. Clique em “Salvar”, por exemplo, não deve ficar artificialmente atrasado. Para entradas críticas, apresente estado de carregamento e mensagem quando não houver resultados.

## 5. Paginar resultados da API

Uma lista pode ter milhares de registros, mas a pessoa vê poucos inicialmente. Entregar todos desperdiça banco, rede, memória e tempo de renderização.

**Como aplicar:** use limite e cursor para listas grandes ou que mudam com frequência. O servidor retorna um conjunto pequeno de itens e um cursor para a próxima página. Paginação por `offset` é simples, mas pode ficar lenta e instável em tabelas grandes.

**Cuidados:** evite calcular uma contagem total cara em toda requisição. Garanta ordenação estável para não repetir ou pular itens entre páginas, especialmente quando novos registros entram na lista.

## 6. Entregar estáticos por CDN

Uma CDN mantém cópias de imagens, fontes, CSS e JavaScript em pontos de presença próximos de quem acessa. Isso reduz latência e tira carga do servidor que atende a aplicação.

**Como aplicar:** entregue assets públicos por CDN, use nomes com hash de conteúdo, como `app.a1b2c3.js`, e defina cache longo para esses arquivos imutáveis.

**Cuidados:** CDN não substitui uma API bem localizada nem uma consulta eficiente. Não faça cache público de conteúdo autenticado, personalizado ou sensível sem uma chave e política corretas.

## 7. Enviar tarefas lentas para uma fila

Relatórios, e-mails, conversão de mídia, importações e processamento de arquivos não precisam bloquear uma resposta HTTP. A API pode aceitar a solicitação e enviar a tarefa para um worker processar em segundo plano.

**Como aplicar:** responda rapidamente com o estado do trabalho; use uma fila durável, workers com concorrência limitada e uma forma de o usuário acompanhar conclusão ou falha.

**Cuidados:** jobs precisam ser idempotentes: uma tentativa repetida não pode cobrar duas vezes ou enviar duplicatas. Configure retries com atraso progressivo, fila de erros e observabilidade.

## 8. Corrigir consultas N+1

N+1 ocorre quando a aplicação busca uma lista e faz outra consulta para cada item. Cem pedidos podem virar 101 consultas; o problema cresce com o volume de dados.

**Como aplicar:** use joins, pré-carregamento, buscas em lote ou um DataLoader. Observe a contagem de queries por endpoint e valide o plano de execução.

**Cuidados:** não troque N+1 por um join gigante que multiplica linhas e traz colunas inúteis. O objetivo é reduzir viagens ao banco sem produzir respostas desnecessariamente enormes.

## 9. Ativar compressão Brotli ou gzip

HTML, CSS, JavaScript, JSON e SVG comprimidos transferem menos bytes. Brotli costuma proporcionar melhor compressão; gzip funciona como alternativa amplamente suportada.

**Como aplicar:** configure a compressão no CDN, proxy reverso ou servidor HTTP e valide os cabeçalhos `Content-Encoding` nas respostas de texto.

**Cuidados:** não comprima novamente WebP, AVIF, JPEG, MP4, ZIP e outros formatos já comprimidos. Isso quase não reduz tamanho e desperdiça CPU.

## 10. Fazer cache dos dados mais consultados no Redis

Redis é útil para respostas repetidas e relativamente estáveis: catálogos, rankings, agregados e configurações públicas. Assim, o banco não precisa recomputar tudo a cada acesso.

**Como aplicar:** use cache-aside: tente o cache, consulte o banco em caso de ausência e armazene a resposta com TTL. Defina chaves com escopo claro e invalide ou atualize-as quando o dado de origem mudar.

**Cuidados:** invalidação é o ponto difícil. Evite expiração simultânea de muitas chaves importantes, que pode causar um pico no banco. Não guarde segredos ou dados pessoais em chaves previsíveis e sem a proteção adequada.

## 11. Remover dependências não usadas

Cada pacote aumenta manutenção, risco de segurança, tempo de instalação e, no front-end, potencialmente o bundle enviado ao navegador.

**Como aplicar:** use um analisador de bundle, procure imports sem uso e prefira imports específicos. Confirme se a ferramenta consegue eliminar código morto por tree-shaking.

**Cuidados:** não remova por busca textual apenas. Uma dependência pode ser carregada dinamicamente, exigida por build ou usada em runtime. Faça testes de build e das jornadas afetadas.

## 12. Definir timeout para APIs externas

Serviços de pagamento, IA, geocodificação ou e-mail podem ficar lentos. Sem timeout, conexões se acumulam e uma dependência externa pode degradar o app inteiro.

**Como aplicar:** defina prazos de conexão e leitura apropriados; mostre erro compreensível; use retries com backoff e jitter somente em operações seguras ou idempotentes.

**Cuidados:** retry de uma operação de escrita pode duplicar efeitos. Para serviços instáveis, implemente circuit breaker: temporariamente interrompa chamadas que estão falhando e ofereça modo degradado.

## 13. Transmitir respostas de IA token a token

Streaming mostra o início da resposta assim que ela é gerada, em vez de esperar a conclusão inteira. Isso melhora o tempo percebido até a primeira informação útil.

**Como aplicar:** use um protocolo de streaming adequado, como SSE; atualize o texto incrementalmente; permita cancelamento e persista a resposta final quando o fluxo terminar.

**Cuidados:** trate conexões interrompidas, duplicação de trechos e conteúdo parcial. Streaming melhora percepção, não reduz por si só custo nem tempo total de inferência.

## 14. Atualizar a interface sem esperar o servidor

Atualização otimista mostra o novo estado imediatamente — por exemplo, “curtido” ou “tarefa concluída” — e sincroniza em segundo plano.

**Como aplicar:** atualize o cache/interface local, envie a operação e reverta com uma mensagem clara se ela falhar. Use identificadores de operação para tornar reenvios seguros.

**Cuidados:** é ideal para ações simples e reversíveis. Para pagamento, estoque limitado, transferência ou permissões, o servidor continua sendo a fonte de verdade e a interface precisa comunicar o estado pendente.

## 15. Usar lazy loading em imagens fora da tela

Imagens que não estão visíveis não precisam disputar banda e processamento com a primeira dobra da página.

**Como aplicar:** use `loading="lazy"` ou observação de interseção para conteúdo abaixo da dobra. Dê prioridade à imagem principal e aos elementos visíveis inicialmente.

**Cuidados:** lazy loading no elemento principal pode piorar LCP. Reserve o espaço da imagem para evitar layout shift e faça teste em redes móveis reais.

## 16. Disparar em paralelo requisições independentes

Buscar perfil, pedidos e notificações em sequência soma tempos de espera. Se não houver dependência, elas podem começar juntas.

**Como aplicar:** agrupe chamadas independentes com execução concorrente; trate cada falha de forma adequada; imponha limites de concorrência para lotes grandes.

**Cuidados:** não paralelize passos que dependem do resultado anterior. Concorrência excessiva pode saturar navegador, API, banco ou serviços externos.

## 17. Criar índices para consultas comuns

Índices permitem que o banco encontre dados sem percorrer uma tabela inteira. Eles são especialmente úteis em filtros, ordenações e joins recorrentes.

**Como aplicar:** observe queries reais, rode `EXPLAIN ANALYZE` ou equivalente e crie índices alinhados à ordem dos filtros e da ordenação. Para múltiplas condições, considere um índice composto.

**Cuidados:** índices consomem disco e deixam escritas mais caras. Cada índice deve justificar seu custo por uma consulta importante e medida.

## 18. Rastrear requisições lentas com OpenTelemetry

Tracing conectado mostra onde o tempo foi gasto: navegador, servidor, banco, cache ou serviço externo. Ele transforma “a tela é lenta” em uma evidência acionável.

**Como aplicar:** crie spans para cada etapa relevante e propague o contexto de trace entre cliente, API e workers. Combine traces com logs e métricas de latência por percentil.

**Cuidados:** não envie tokens, senhas, conteúdo privado ou identificadores pessoais desnecessários para a plataforma de observabilidade. Controle cardinalidade de atributos para evitar custo excessivo.

## 19. Manter servidor perto do banco de dados

Cada consulta paga uma ida e volta de rede. Se API e banco estão em regiões distantes, múltiplas consultas por página amplificam a latência.

**Como aplicar:** mantenha serviços que conversam frequentemente na mesma região, usando zonas diferentes apenas quando a estratégia de disponibilidade exigir.

**Cuidados:** isso não significa colocar tudo perto de um único usuário. Usuários globais ainda se beneficiam de CDN e arquitetura regional; escolha conforme dados de acesso e requisitos de residência de dados.

## 20. Renderizar no servidor ou no build

SSR entrega HTML inicial pronto no servidor; SSG constrói páginas antecipadamente. Ambos reduzem a tela em branco enquanto o JavaScript baixa e executa.

**Como aplicar:** use SSG para conteúdo estável, SSR para páginas dinâmicas importantes para o primeiro carregamento e componentes no cliente apenas quando houver interação que o exija.

**Cuidados:** SSR adiciona custo e pode aumentar TTFB se mal implementado. Reduza hidratação e JavaScript desnecessário; renderizar HTML no servidor não resolve sozinho um bundle pesado.

## 21. Carregar scripts de terceiros depois da página

Chat, analytics, pixels, anúncios e widgets externos frequentemente bloqueiam renderização e executam tarefas longas no celular.

**Como aplicar:** use `defer`, carregamento após a primeira interação ou após a página estar utilizável. Audite quais scripts realmente geram valor e avalie cada fornecedor individualmente.

**Cuidados:** respeite consentimento antes de carregar rastreadores. Scripts externos podem mudar sem aviso; monitore seu impacto e tenha um plano se o fornecedor ficar lento ou indisponível.

## 22. Dividir o JavaScript por página

Code splitting evita enviar o código de todas as áreas do app para uma pessoa que abriu somente uma rota.

**Como aplicar:** divida por rota e por recursos pesados, como editor, mapa, gráfico ou área administrativa. Carregue o módulo sob demanda e, quando fizer sentido, faça prefetch da próxima navegação provável.

**Cuidados:** chunks demais criam cascata de requisições. Analise o grafo do bundle; a meta é diminuir o caminho crítico, não gerar fragmentos arbitrários.

## 23. Virtualizar listas longas

Renderizar milhares de linhas no DOM torna a rolagem e as interações lentas. Virtualização mantém somente itens visíveis e uma margem pequena ao redor deles.

**Como aplicar:** use uma biblioteca ou componente de janela virtual; estime ou meça alturas; adicione overscan moderado para evitar espaços em branco durante a rolagem.

**Cuidados:** valide foco por teclado, leitores de tela, seleção, busca e alturas variáveis. Uma lista virtualizada precisa manter a experiência acessível, não apenas rápida.

## 24. Pedir somente as colunas necessárias

`SELECT *` e respostas genéricas podem enviar campos grandes, sensíveis ou inúteis para a tela. Isso desperdiça banco, rede, serialização e memória.

**Como aplicar:** crie DTOs ou projeções por uso: o card recebe os campos do card, a página detalhada recebe o conjunto completo. Faça a seleção diretamente na consulta quando possível.

**Cuidados:** não faça uma chamada separada para cada campo. Agrupe campos coerentes por jornada e mantenha contratos de API claros e versionáveis.

## 25. Usar pool de conexões

O pool mantém conexões com o banco reutilizáveis, reduzindo o custo de abrir uma conexão a cada requisição e protegendo o banco de sobrecarga.

**Como aplicar:** defina limite por instância, timeout de espera e monitoramento de conexões ocupadas. Em ambientes serverless, use proxy/pooler apropriado.

**Cuidados:** some o máximo de conexões de todas as instâncias; um pool aparentemente pequeno pode exceder o limite do banco quando o app escala. Pool não conserta consultas lentas.

## 26. Evitar re-renderizações desnecessárias

Em front-ends reativos, atualizar estado no topo da árvore pode redesenhar muito mais do que o necessário. O efeito aparece em tabelas, filtros, formulários e painéis densos.

**Como aplicar:** mantenha o estado perto de quem o utiliza, separe componentes, estabilize props e use o profiler do framework para localizar o custo real. Memoize apenas componentes que demonstraram ser caros.

**Cuidados:** `memo`, `useMemo` e `useCallback` não são solução universal. Usá-los em toda parte pode aumentar complexidade, memória e bugs de dependência.

## 27. Fazer cache de estáticos no navegador

O navegador pode reutilizar CSS, JavaScript, logos e fontes entre visitas e rotas, evitando downloads repetidos.

**Como aplicar:** para arquivos versionados por hash, use cache longo e `immutable`. Para recursos que mudam sem hash, use validação com ETag ou `stale-while-revalidate` quando adequado.

**Cuidados:** não armazene respostas personalizadas em cache público. Teste atualização e rollback: cache agressivo sem versionamento pode deixar usuários presos a um app antigo.

## 28. Distribuir tráfego entre várias instâncias

Um balanceador encaminha acessos a várias cópias da aplicação. Isso aumenta capacidade, reduz ponto único de falha e permite escalar por demanda.

**Como aplicar:** mantenha a aplicação stateless; guarde sessão, arquivos e tarefas em serviços compartilhados; configure health checks, autoscaling e desligamento gracioso.

**Cuidados:** escalar a API não elimina gargalo de banco, cache, fila ou serviço externo. Teste falhas de instância e garanta que requisições e jobs podem ser retomados.

## 29. Hospedar fontes e limitar pesos

Cada família, estilo e peso de fonte adiciona arquivos e pode atrasar a renderização de texto. Hospedar fontes localmente dá mais controle de cache e elimina uma dependência adicional.

**Como aplicar:** use WOFF2, carregue somente pesos empregados no design, faça subset de caracteres quando possível e use uma estratégia como `font-display: swap`.

**Cuidados:** fonte demais cria requests, reflow e consumo de memória. Evite fonte de ícones quando SVG atende; considere a troca visual temporária de fonte ao escolher a estratégia de carregamento.

## 30. Monitorar desempenho de usuários reais

Testes de laboratório são necessários, mas não representam toda a variedade de celulares, redes, regiões e extensões. RUM mostra o que acontece nas visitas reais.

**Como aplicar:** acompanhe Core Web Vitals, erros e tempos de jornadas por rota, tipo de dispositivo, versão do app e qualidade de rede. Prefira percentis como p75 e p95 a médias simples.

**Cuidados:** colete o mínimo de dados necessário, com privacidade e consentimento adequados. Uma métrica sem segmentação pode esconder que apenas usuários móveis estão sofrendo.

## 31. Testar no celular e em rede lenta

Computador de desenvolvimento e Wi-Fi rápido escondem problemas. A experiência precisa funcionar em dispositivos reais, CPU limitada e conexão instável.

**Como aplicar:** teste carregamento inicial, navegação, rolagem, formulários, imagens e recuperação após perda de conexão. Use limitação de banda/latência e, sempre que possível, aparelhos físicos representativos do público.

**Cuidados:** teste também estados de erro, cancelamento e retorno da conectividade. A meta não é apenas uma nota alta em ferramenta de laboratório: é uma interface compreensível, estável e recuperável para pessoas reais.

## Ordem sugerida de implementação

1. Instrumente a aplicação: métricas reais, tracing e logs úteis.
2. Ataque peso inicial: imagens, fontes, JavaScript e scripts de terceiros.
3. Elimine trabalho repetido: chamadas duplicadas, N+1, dados excessivos e renderizações desnecessárias.
4. Melhore resiliência: timeouts, filas, cache, pools e degradação controlada.
5. Escale infraestrutura somente quando as métricas mostrarem que é necessário.
6. A cada mudança, compare os resultados e mantenha testes de regressão de desempenho.

## Checklist curto para revisar uma tela

- A primeira dobra entrega conteúdo útil rapidamente?
- A imagem principal tem tamanho e prioridade corretos?
- Há JavaScript, fontes ou scripts externos que não são necessários naquele momento?
- A API retorna apenas o necessário, em poucas chamadas e com consultas eficientes?
- O estado de carregamento, falha e lentidão é claro para a pessoa usuária?
- A experiência foi validada em celular, com CPU e rede limitadas?
- Existe evidência — métrica ou trace — de que a mudança melhorou o cenário?
