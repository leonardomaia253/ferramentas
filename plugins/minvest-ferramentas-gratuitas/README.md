# MInvest Ferramentas Gratuitas

Versão 1.8.83: rótulos e perguntas gerados para o MCP evitam repetir a unidade, facilitando a leitura pelos agentes.

Serviço MCP público para agentes compatíveis, com pacotes de plugin para Codex e Claude Code. Oferece oito calculadoras financeiras, comparação de 2 a 4 cenários e pesquisa inicial de mercado na web com fontes citadas, sem login e sem acesso a dados de uma conta MInvest.

Cada cálculo operacional retorna uma próxima validação específica para suas premissas e limitações. Use essa etapa como checagem, não como decisão automática.

As oito calculadoras são: preço e margem, ponto de equilíbrio, capital de giro, fôlego de caixa, viabilidade de expansão, diluição societária, dimensionamento de mercado e valuation indicativo. Há ainda o checklist de preparação para investimento, `comparar_cenarios` e `pesquisar_mercado`. Preço e margem também converte a contribuição atual em unidades e faturamento mensais necessários para cobrir os custos fixos informados, e sinaliza quando contribuição nula ou negativa impede equilíbrio por volume. Os cálculos são educativos e retornam fórmulas, premissas, limitações e uma leitura do cenário.

`pesquisar_mercado` busca sinais de demanda, concorrentes/substitutos, tendências, oportunidades e riscos em fontes públicas e pode direcionar a síntese a uma decisão informada. Exige consentimento explícito para enviar tema, região e os campos opcionais de público/decisão à OpenAI, bloqueia padrões comuns de dados identificáveis, retorna citações clicáveis e diferencia fatos de inferências. A busca é exploratória: não representa amostra estatística, benchmark de mercado nem monitoramento contínuo. Não inventa TAM/SAM/SOM nem transforma estimativas de terceiros em dados auditados. A chamada usa `store=false`, mas a política padrão da OpenAI pode reter logs de segurança por até 30 dias. O limite é de três pesquisas por dia por IP; chamadas recusadas por falta de consentimento ou por campos sensíveis não consomem essa cota específica.

Na simulação de diluição, pre-money e aporte precisam ser positivos. Aporte zero não representa uma rodada de investimento; a participação individual de um sócio continua opcional para a simulação geral.

As calculadoras aceitam os valores disponíveis em etapas. Se faltarem premissas, o agente recebe a lista exata de campos e perguntas para reuni-los numa única resposta, sem repetir o que já foi informado nem calcular com lacunas.

Em preço e margem, custo unitário, taxas e meta de contribuição bastam para a referência sem rateio. O preço praticado é opcional para analisar a margem atual; custos fixos e volume mensal esperado são opcionais como par para calcular rateio, preço carregado e equilíbrio. Se um deles vier sem o outro, o agente pergunta somente o campo que falta. O rateio pressupõe que o volume informado seja vendido e não estima demanda, descontos, mix ou disposição a pagar. Os preços de referência são arredondados para cima ao centavo e nunca ficam abaixo de R$ 0,01; quando o custo total informado é zero, esse é o menor preço positivo representável, não uma recomendação comercial.

Quando a contribuição unitária é nula ou negativa, preço e margem ou ponto de equilíbrio retornam as métricas de equilíbrio com `value: null` e `valueStatus: no_break_even_with_nonpositive_contribution`, inclusive quando custos fixos são zero. A contribuição calculada continua visível. Cada venda não cobre os próprios custos variáveis, então aumentar o volume não resolve o problema. Não trate `null` como zero ou falha de cálculo. O comparador propaga esse estado e não calcula um delta enganoso. Essa orientação também aparece na descrição da ferramenta MCP para clientes que leem apenas `tools/list`.

Quando os custos fixos informados são zero e a contribuição é positiva, o resultado de zero unidades significa apenas que não há custos fixos informados para cobrir. Isso não comprova lucro ou sustentabilidade nem considera despesas que não entraram no cálculo; o MCP marca essas métricas com `valueStatus: no_fixed_costs_to_cover`. Se a contribuição por unidade for nula ou negativa, o estado `zero_fixed_costs_nonpositive_contribution` destaca que zero unidades/faturamento não representa uma operação sustentável; compare a contribuição, não o zero de custos fixos.

No fôlego de caixa, cada cenário mostra o fluxo líquido mensal (positivo = consumo; negativo = geração) junto do prazo estimado. O estresse reduz entradas pelo percentual informado e a alternativa de ajuste reduz saídas em 10%. Saldo inicial zero ou negativo significa ausência de reserva, mesmo que o fluxo mensal esteja equilibrado. Fôlego ilimitado significa somente que as premissas mensais não mostram consumo líquido; vencimentos, atrasos e saldo bancário ainda precisam ser conferidos.

As reduções de entradas e saídas são opcionais. O retorno não descreve cenário que não foi solicitado, usa exatamente o percentual de redução informado e alerta também quando o fôlego atual (sem estresse) fica abaixo de um mês.

Nas respostas MCP de fôlego, prazos ilimitados ou acima da faixa numérica usam `value: null` e um `valueStatus` explícito; o agente deve apresentar o campo `formatted` e não reconstruir prazo a partir de sentinelas numéricos.

`comparar_cenarios` permite colocar de duas a quatro alternativas lado a lado nas calculadoras operacionais, dimensionamento de mercado e valuation, com variações frente ao cenário-base. No dimensionamento, compare clientes, gasto, percentuais de atendimento/conquista e horizonte; o SOM continua sendo oportunidade anualizada derivada dessas premissas, não previsão de vendas. A ferramenta não seleciona uma alternativa nem supre premissas ausentes; receita zero continua sendo ausência de base, não valuation zero.

No comparador, `baselineValueStatus` e `valueStatus` identificam o estado tanto do cenário-base quanto de cada alternativa. Em métricas de valuation, `no_revenue_basis` significa que não há base de receita para calcular aquela faixa; use `formatted` e não substitua `null` por zero. `analysisNotRun` indica que aquela métrica opcional não foi executada no cenário.

O comparador também aceita cenários parciais: preserva os valores conhecidos e retorna as premissas faltantes separadas por alternativa. Não calcula até que todas estejam completas.

Capital de giro exige receita faturada a prazo, prazo de recebimento, caixa livre e reserva mínima. CMV + dias de estoque são opcionais como par; compras a prazo + prazo de pagamento também. Blocos não enviados ficam fora da estimativa e são identificados como cobertura parcial — não são tratados como zero ou inexistentes. A necessidade externa calculada cobre apenas os componentes enviados. Usa mês de 30 dias e não considera inadimplência, sazonalidade, impostos ou vencimentos em datas específicas. Se apenas parte das vendas for a prazo, informe só esse valor de receita.

No MCP, `calculationScope` identifica os componentes incluídos, os opcionais não informados e os fatores fora do modelo. O mesmo campo é retornado em cada cenário de `comparar_cenarios`; use-o para qualificar a necessidade externa estimada sem presumir que os itens omitidos não se aplicam.

O servidor limita cada endereço IP público a 120 chamadas a ferramentas por hora, incluindo chamadas com dados incompletos; agentes que compartilham a mesma saída de rede também compartilham o limite. `comparar_cenarios` conta como uma chamada mesmo ao comparar até quatro cenários. Quando atingido, a resposta informa o tempo para tentar novamente. A chave de limite é um hash HMAC do IP confiável fornecido pela borda Supabase; o endereço bruto não é armazenado pela ferramenta.

Na expansão, informe entradas e saídas mensais em regime e rampas lineares separadas de 0 a 120 meses (zero significa regime integral desde o mês 1); o horizonte continua de 1 a 600 meses inteiros. Payback ocorre no fechamento do mês, e atraso inicial, marcos irregulares e valor residual não entram no modelo. O schema MCP e a calculadora web aplicam os mesmos limites. No valuation, SaaS e modelos recorrentes usam receita recorrente anual (ARR) como base; serviços e comércio lucrativos usam EBITDA anual; os demais casos usam receita anual. Marketplace usa receita líquida reconhecida (take rate), nunca volume bruto transacionado, e não exige percentual de receita recorrente. A saída agora separa base do múltiplo, valor operacional antes da dívida líquida, equity central após a dívida e faixa inferior/superior; a comparação e os agentes recebem os mesmos componentes. Crescimento acima de 80% é aceito até 200%, mas destacado para validação. As faixas são heurísticas sem comparáveis verificados.

No MCP, payback não atingido até o horizonte retorna `value: null` com `valueStatus: not_reached_within_horizon`; isso descreve o período analisado, não todos os prazos possíveis. O comparador propaga o estado e não calcula uma diferença numérica quando pelo menos um cenário não recupera o investimento no horizonte escolhido.

No valuation, `methods` lista apenas o múltiplo realmente aplicado; `multipleFormula`, múltiplo-base e fator de qualidade permitem conferir a conta. A concentração do maior cliente é ajustada uma única vez. A interface e os agentes mostram premissas coletadas que não alteram o ramo escolhido e deixam explícito que DCF, comparáveis, cohorts e outras abordagens não foram calculadas. Os fatores 0,72 e 1,38 formam uma sensibilidade fixa, não um intervalo estatístico nem evidência de mercado.

Todas as saídas monetárias das calculadoras, inclusive valuation, são arredondadas a centavos e usam a mesma regra na página e no MCP. Se a escala ultrapassar a precisão segura em centavos, a ferramenta interrompe o cálculo com `precision_limit`; não reduza a escala nem substitua a resposta por uma estimativa manual sem confirmar os valores e as unidades.

O valuation pede o modelo primeiro e então só os campos que afetam o ramo escolhido. SaaS, marketplace e deep tech não pedem lucro nem margem EBITDA; serviços e comércio usam perguntas condicionais para escolher entre receita e EBITDA. `notUsedInputs` identifica campos que não participam do ramo, sem exigir que a pessoa estime números irrelevantes.

Na comparação, os deltas monetários também são arredondados a centavos. Se `absoluteChangeStatus` retornar `precision_limit`, a diferença não é apresentada como número; confira unidade e escala de cada cenário antes de comparar novamente. Se retornar `analysis_not_run`, a métrica não foi calculada dos dois lados e os deltas absoluto e relativo ficam indisponíveis. Estados não comparáveis, como `nonpositive_contribution`, também omitem ambos os deltas em vez de exibir zero enganoso.

No checklist de investimento, a ferramenta separa itens concluídos, em andamento, pendentes, declarados fora de escopo e sem status. Fora de escopo exige justificativa de 12 a 240 caracteres e continua autodeclarado, sem verificação externa. O percentual concluído exclui itens fora de escopo do denominador, mantém itens sem status no denominador e não estima chance de investimento. `completionMetricStatus` identifica o percentual como autodeclarado ou informa que não há status; a API não chama essa métrica de confirmada.

No checklist, o percentual é uma autoavaliação com peso igual por item, não uma medida de prontidão. Para itens concluídos, pode-se informar um localizador genérico de evidência; a ferramenta mostra a cobertura dessas referências, mas não abre nem verifica arquivos. Não inclua arquivos, dados pessoais, links privados ou conteúdo confidencial. O endpoint recusa URLs, e-mails, telefones e alguns formatos comuns de credencial; essa checagem não detecta todo dado sensível, então mantenha a referência genérica. Itens fora de escopo são autodeclarados; itens sem status continuam no denominador e não viram pendências nem conclusões.

Para cada item, o checklist agora oferece ação e evidência esperada. O plano operacional separa trabalho de pendências confirmadas de passos para descobrir o status dos itens ainda desconhecidos.

## Dados enviados

Os números informados são enviados ao endpoint MCP da MInvest para processamento e devolvidos ao mesmo agente que fez a chamada. Ao usar um atalho (`prompts/get`), o objetivo e o contexto genérico também são enviados para montar a orientação. A função não grava esses inputs em banco de negócio nem os registra em log explícito; o tráfego ainda passa pela infraestrutura Supabase e pelo provedor do agente, cujas políticas de retenção se aplicam. Informe isso antes do envio e não inclua nomes, contatos, documentos, URLs privadas, credenciais ou informações confidenciais de clientes.

## Instalação

### Qualquer agente compatível com MCP

Configure um servidor MCP remoto por HTTP (Streamable HTTP) no cliente:

```
https://azobnryexzyeqbgtowyc.supabase.co/functions/v1/minvest-free-tools
```

Depois da conexão, o agente descobre as calculadoras com `tools/list` e as executa com `tools/call`. Um plugin ou skill é opcional: a conexão MCP direta já disponibiliza todas as ferramentas e as instruções operacionais do servidor.

### Codex CLI

```sh
codex mcp add minvest-free-tools --url https://azobnryexzyeqbgtowyc.supabase.co/functions/v1/minvest-free-tools
```

Confirme a conexão com `codex mcp list`.

No Codex CLI, adicione o marketplace público: `codex plugin marketplace add https://github.com/leonardomaia253/ferramentas.git --sparse .agents/plugins`. Depois, instale **MInvest Ferramentas** no diretório de plugins do Codex desktop. O pacote Codex usa o formato portátil `plugin.json` + `mcp.json`.

### Claude Code

O plugin já instala a skill e configura o servidor MCP:

```sh
claude plugin marketplace add leonardomaia253/ferramentas
claude plugin install minvest-ferramentas-gratuitas@minvest-ferramentas
```

Confirme a conexão com `/mcp` dentro do Claude Code. Para conectar somente o servidor MCP, use `claude mcp add --transport http --scope user minvest-free-tools https://azobnryexzyeqbgtowyc.supabase.co/functions/v1/minvest-free-tools`.

O plugin gratuito está publicado no marketplace público. A listagem universal do Codex/ChatGPT e a submissão ao diretório Anthropic ainda dependem dos respectivos portais e aprovações.

Para gerar os pacotes ZIP de release para os portais, execute `npm run plugins:sync-codex` e depois `npm run plugins:package`. Codex e Claude devem compartilhar a mesma versão no manifesto. Os arquivos e hashes SHA-256 aparecem em `output/plugins/`; eles não incluem credenciais.

## Limites

As ferramentas não substituem análise financeira profissional, não verificam os dados informados e não devem ser apresentadas como laudo, garantia de retorno ou recomendação de investimento. O valuation exige receita para produzir uma faixa; sem ela, o resultado informa que não há base suficiente.
