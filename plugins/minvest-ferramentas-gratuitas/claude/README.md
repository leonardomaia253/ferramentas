# MInvest Ferramentas Gratuitas

Oito calculadoras financeiras gratuitas, comparação de cenários e pesquisa exploratória de mercado com fontes públicas clicáveis, sem login e sem acesso a dados de uma conta. A pesquisa pode ser orientada a uma decisão, exige consentimento explícito para enviar tema, região e campos opcionais de público/decisão à OpenAI, e não equivale a benchmark representativo ou monitoramento contínuo. Cada resultado de cálculo traz fórmula, premissas, leitura do cenário e limitações.

Os números são enviados ao endpoint MCP da MInvest para processamento e devolvidos ao mesmo agente. Ao usar um atalho (`prompts/get`), o objetivo e o contexto genérico também são enviados para montar a orientação. A função não grava esses inputs em banco de negócio nem os registra em log explícito; a chamada também passa pela infraestrutura Supabase e pelo provedor do agente, cujas políticas de retenção se aplicam. Informe isso antes do envio e não inclua nomes, contatos, documentos, URLs privadas, credenciais ou dados confidenciais de clientes.

## Conectar ao Claude Code

O plugin já inclui a configuração do servidor MCP. A instalação abaixo conecta as ferramentas e adiciona a skill de orientação:

```sh
claude plugin marketplace add leonardomaia253/ferramentas
claude plugin install minvest-ferramentas-gratuitas@minvest-ferramentas
```

Confirme a conexão com `/mcp` dentro do Claude Code.

Para conectar somente o servidor MCP, sem instalar a skill do marketplace, use:

```sh
claude mcp add --transport http --scope user minvest-free-tools https://azobnryexzyeqbgtowyc.supabase.co/functions/v1/minvest-free-tools
```

O plugin está no marketplace público MInvest. A submissão ao diretório Anthropic ainda depende do portal de desenvolvedor e da aprovação da Anthropic.

Ferramentas: preço e margem, ponto de equilíbrio, capital de giro, fôlego de caixa, viabilidade de expansão (horizonte de 1 a 600 meses inteiros e rampas lineares de 0 a 120 meses), diluição societária, preparação para investimento e valuation indicativo. Na expansão, entradas e saídas mensais representam o regime integral; zero meses significa esse valor desde o mês 1. O modelo calcula VPL e payback mês a mês, sem atraso inicial ou marcos irregulares. Preço e margem mostra uma referência sem rateio fixo e outra com custos fixos distribuídos pelo volume mensal esperado; ambas arredondam para cima ao centavo e deixam explícito que o volume é uma premissa. Capital de giro separa recebíveis, estoque e compras a prazo de fornecedores, compara a necessidade operacional com caixa atual e reserva mínima e mostra quanto faltaria financiar; usa mês de 30 dias. No valuation, SaaS e outros modelos recorrentes usam ARR; serviços e comércio lucrativos usam EBITDA; marketplace usa receita líquida/take rate e não exige percentual de receita recorrente. A saída separa valor operacional antes da dívida líquida e equity central após dívida; agentes recebem os mesmos valores e faixas. Crescimento acima de 80% aparece como premissa que precisa de comprovação, e a faixa continua heurística sem comparáveis verificados. O comparador aceita cenários parciais, identifica lacunas por alternativa e só calcula quando todos os dados necessários estiverem presentes.

No valuation, a ferramenta lista somente o múltiplo que efetivamente aplicou, retorna a fórmula, o múltiplo-base e o fator de qualidade, e explica quais respostas coletadas não alteraram aquele ramo. A concentração do maior cliente é aplicada uma única vez. Não são calculados DCF, comparáveis, cohorts, economia unitária, ativos ou marcos tecnológicos; os limites 0,72/1,38 são sensibilidade fixa e não intervalo estatístico.

No checklist, o percentual é autoavaliação com peso igual por item, não medida de prontidão. Para itens marcados como organizados, pode-se informar localizador genérico; a referência não é aberta nem verificada. Não solicite documentos nem dados confidenciais. Itens sem status seguem desconhecidos; a pontuação não representa chance de captação.

O limite do MCP é de 120 chamadas a ferramentas por hora por IP público, inclusive chamadas com dados incompletos; agentes na mesma saída de rede compartilham essa cota. O comparador conta como uma chamada mesmo ao comparar quatro cenários, e a resposta informa quando e em quanto tempo tentar novamente após atingir o limite.

Os cálculos são educativos e não constituem laudo ou recomendação financeira.

Cada calculadora operacional também retorna uma próxima validação específica. Use-a para conferir as premissas e limitações, sem tratar o resultado como decisão automática.

No fôlego de caixa, fluxo mensal positivo representa consumo e negativo representa geração líquida. O estresse reduz entradas pelo percentual informado; outro cenário reduz saídas em 10%. O fôlego é medido até saldo zero, sem calendário de vencimentos.

No checklist de investimento, itens sem status informado continuam não confirmados. A ferramenta não os presume pendentes. Itens fora de escopo exigem justificativa de 12 a 240 caracteres; a classificação é autodeclarada, não verificada, e o percentual exclui esses itens sem excluir itens desconhecidos.
