# Oficina IPD · Meu app está lento. Por onde começo?

Escolha uma ação que alguém faz no seu app. Ao terminar esta rodada, você terá uma medida inicial, uma hipótese e uma decisão sobre a mudança testada.

## Comece pelo sintoma

| O que a pessoa percebe | Primeira observação | Onde procurar no guia |
| --- | --- | --- |
| A página demora a mostrar o produto | Abra Rede/Network nas ferramentas do navegador, recarregue e veja quando a foto principal começa a baixar, seu tamanho e sua duração. | Imagens (2 e 15), scripts (21 e 22), fontes (29) |
| A busca trava ou mostra o termo anterior | Digite uma palavra, conte os pedidos de busca e veja em que ordem as respostas chegam. | Buscas (3 e 4), listas (5 e 23) |
| A lista demora mesmo com poucos itens na tela | Compare o tempo até a resposta da API com o tempo até aparecer a lista. Se a API demora, conte as consultas ao banco por pedido. | Consultas (8, 17 e 24), renderização (26) |
| Gerar um relatório deixa a tela parada | Cronometre separadamente até a confirmação do pedido e até o arquivo ficar disponível. | Fila (7), dependências externas (12), rastreamento (18) |

Os números correspondem às seções do [guia com as 31 práticas](GUIA.md).

## Faça uma rodada

1. Escreva a jornada em uma frase verificável, como “digitar caneca na busca e ver os resultados de caneca”.
2. Copie a [ficha](FICHA.md). Registre aparelho, rede, dados, versão e estado do cache. Repita a ação cinco vezes e anote as medidas individuais e a mediana, o valor do meio depois de ordenar.
3. Escolha uma hipótese compatível com a observação. Faça a menor mudança que permita testá-la em uma cópia de desenvolvimento.
4. Repita a medição nas mesmas condições. Confira que a ação continua correta, inclusive em caso de erro. Se a diferença couber na variação das tentativas, registre resultado inconclusivo.
5. Decida se mantém, reverte ou mede novamente. Ao compartilhar, conte o que a pessoa fazia, o que mudou e qual foi o resultado.

## Faça com seu assistente de código

Abra o projeto no seu editor, disponibilize GUIA.md e FICHA.md e cole o texto abaixo, trocando os colchetes pela sua jornada.

> Quero investigar [o que faço e onde percebo a demora]. Leia o guia e inspecione o caminho dessa ação no projeto. Identifique os arquivos envolvidos e separe o que o código mostra do que precisa ser medido. Sugira uma métrica, explique como coletá-la e aguarde os valores se você não conseguir medir. Com a medida inicial, indique uma única mudança e o teste que mostraria se a hipótese está errada. Quando eu escolher a mudança, aplique-a localmente, valide a jornada e preencha a ficha usando apenas medições observadas.

## Exemplo preenchido · dados fictícios

Na busca do catálogo, digitar “caneca” disparava seis consultas. A hipótese era que cada tecla iniciava uma nova busca. Repetimos cinco vezes a mesma sequência de teclas, com intervalo de 100 ms. Antes, as contagens foram 6, 6, 6, 6 e 6. Com uma espera de 250 ms após a última tecla, foram 1, 1, 1, 1 e 1. A lista final correspondia a “caneca”. A mudança reduziu consultas nesse cenário. Para decidir se a espera ficou confortável, ainda falta medir o tempo entre a última tecla e a exibição da lista.
