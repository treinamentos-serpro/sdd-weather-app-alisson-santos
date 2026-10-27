# Discovery — Aplicação de previsão do tempo

## Contexto

A empresa solicitou uma aplicação para consultar informações meteorológicas de cidades. A experiência deve permitir encontrar uma cidade, consultar as condições atuais e a previsão para cinco dias, alternar entre Celsius e Fahrenheit e funcionar em dispositivos móveis. O briefing ainda não define público-alvo, fonte dos dados, plataformas além de dispositivos móveis nem o nível de detalhe meteorológico.

## Revisão da Classificação

| Item da lista atual | Classificação | Avaliação |
| --- | --- | --- |
| RF1 — Buscar cidades | Funcional | Correto: descreve uma ação oferecida ao usuário. |
| RF2 — Clima atual | Funcional | Correto: define informação que o sistema deve apresentar. |
| RF3 — Previsão de cinco dias | Funcional | Correto: define informação que o sistema deve apresentar. |
| RF4 — Alternar unidade | Funcional | Correto: define uma interação e seu resultado. |
| RF5 — Estados da consulta | Funcional | Correto: define respostas observáveis a carregamento, ausência de resultados e falha. |
| RNF1 — Responsividade | Não-funcional | Correto: descreve adaptação da qualidade da interface; faltava definir tamanhos de tela e critérios verificáveis. |
| RNF2 — Usabilidade | Não-funcional | Correto: é atributo de qualidade, mas “compreensível” é subjetivo e precisa de validação. |
| RNF3 — Acessibilidade | Não-funcional | Correto: é atributo de qualidade, mas os critérios estavam genéricos. |
| RNF4 — Resiliência | Não-funcional | Correto como atributo de qualidade; a exibição de estados e mensagens é o comportamento funcional correspondente, já coberto em RF5. |
| RNF5 — Consistência entre cidade/unidade e dados exibidos | Funcional | **Classificado incorretamente:** determina como o sistema deve atualizar os dados após uma seleção. Foi movido para RF6. |

## Requisitos Funcionais

- **RF1 — Buscar cidades:** permitir que o usuário pesquise uma cidade e selecione o local desejado.
- **RF2 — Clima atual:** apresentar as condições meteorológicas atuais da cidade selecionada.
- **RF3 — Previsão de cinco dias:** apresentar a previsão para os cinco dias solicitados.
- **RF4 — Alternar unidade:** permitir alternar entre Celsius e Fahrenheit e refletir a unidade escolhida nos valores de temperatura exibidos.
- **RF5 — Estados da consulta:** informar quando uma busca ou consulta estiver em andamento, não encontrar resultados ou falhar, com opção de tentar novamente quando aplicável.
- **RF6 — Atualizar dados da seleção:** apresentar clima atual e previsão para a cidade selecionada e aplicar a unidade escolhida a todas as temperaturas exibidas.

## Requisitos Não-Funcionais

- **RNF1 — Responsividade:** manter as funções principais utilizáveis em larguras de 320 px a 1440 px, sem rolagem horizontal da página.
- **RNF2 — Usabilidade:** tornar busca, leitura da previsão e alternância de unidade compreensíveis; validar com usuários ou testes de usabilidade, pois o critério de sucesso ainda precisa ser acordado.
- **RNF3 — Acessibilidade:** buscar conformidade com WCAG 2.2 nível AA, incluindo operação por teclado, foco visível, nomes acessíveis para controles e contraste adequado.
- **RNF4 — Resiliência:** tratar falhas de conexão e indisponibilidade da fonte de dados sem deixar a interface em estado indefinido; permitir recuperação ou nova tentativa quando possível.
- **RNF5 — Performance (proposto):** buscar carregamento do conteúdo inicial em até 2 segundos e conclusão de uma consulta em até 3 segundos no percentil 95, sob condições de rede de referência a definir.
- **RNF6 — Disponibilidade (proposto):** definir meta mensal de disponibilidade de pelo menos 99,5% para a aplicação; monitorar separadamente a disponibilidade do provedor meteorológico, que é uma dependência externa.

> RNF5 e RNF6 são metas iniciais para discussão, não compromissos aprovados. RNF1 e RNF3 já constavam na lista anterior, mas receberam critérios mais verificáveis; os quatro pontos solicitados são, portanto, cobertos como lacunas ou refinamentos.

## Riscos

As avaliações abaixo são preliminares: a probabilidade e o impacto podem mudar quando as perguntas em aberto forem respondidas.

| Categoria | Risco | Probabilidade | Impacto | Estratégia de mitigação |
| --- | --- | --- | --- | --- |
| Técnico — dependência externa | Indisponibilidade, limite de requisições ou mudança no serviço meteorológico impedir ou degradar consultas. | Média | Alto: usuários ficam sem dados e a experiência principal falha. | Avaliar limites, SLA, cobertura e política de mudança antes da escolha; implementar timeout, tratamento de erros, cache apropriado e nova tentativa controlada. |
| Técnico — qualidade dos dados | Dados defasados, incompletos ou com diferenças de fuso/localidade serem apresentados como atuais. | Média | Alto: decisões do usuário podem se basear em informação incorreta. | Exibir horário da atualização e fuso; validar respostas e documentar a semântica dos dados do provedor. |
| Produto — busca de cidades | Cidades homônimas ou resultados de busca pouco claros levarem à localidade errada. | Média | Alto: toda a previsão exibida pode ser irrelevante. | Mostrar país e subdivisão regional nos resultados e permitir confirmar a cidade selecionada. |
| Produto — escopo da previsão | “Cinco dias” e os campos da previsão serem interpretados de maneiras diferentes por negócio e desenvolvimento. | Alta | Médio: retrabalho de interface e divergência na aceitação da entrega. | Fechar a janela de datas, granularidade e campos antes do plano técnico; incluir exemplos nos critérios de aceite. |
| Técnico — conversão de unidades | Valor convertido ou rótulo Celsius/Fahrenheit incorreto ou inconsistente entre telas. | Baixa | Alto: reduz confiança e pode induzir decisão errada. | Centralizar a conversão, testar valores-limite e arredondamento e validar unidade e rótulo em todos os componentes. |
| Produto — experiência móvel | Layout ou controles não funcionarem bem nos dispositivos e condições de uso prioritários. | Média | Alto: impede o uso no contexto principal declarado pelo briefing. | Definir viewport e dispositivos suportados; validar tarefas essenciais em telas pequenas e em navegadores móveis reais. |
| Produto — acessibilidade | Barreiras de teclado, leitor de tela, foco ou contraste excluírem parte dos usuários. | Média | Alto: parte do público não consegue consultar ou interpretar a previsão. | Adotar critério de conformidade acordado, verificar componentes e fluxos com ferramentas e testes manuais assistivos. |
| Técnico — performance | Consultas lentas, especialmente em rede móvel, causarem abandono ou aparência de falha. | Média | Médio: degrada a conclusão da tarefa e a percepção de confiabilidade. | Definir condições de medição e metas; medir carregamento e consultas, reduzir payloads e apresentar estado de carregamento sem bloquear a interface. |
| Técnico/operacional — disponibilidade | Meta de disponibilidade não considerar falhas do provedor externo ou não corresponder à expectativa do negócio. | Média | Alto: não há previsibilidade operacional nem tratamento acordado para indisponibilidade. | Definir SLO para a aplicação e dependências separadamente; monitorar falhas e comunicar indisponibilidade de forma clara. |
| Produto/financeiro — provedor | Custos, limites, licença, atribuição ou cobertura geográfica do provedor inviabilizarem o uso previsto. | Média | Alto: pode exigir troca de fornecedor ou limitar regiões e funcionalidades após o desenvolvimento. | Comparar provedores com cenários de uso e volume; revisar termos, custos e cobertura antes de fixar a integração. |
| Produto/privacidade — localização | Geolocalização ser adicionada sem consentimento claro ou sem necessidade confirmada. | Baixa | Alto: pode reduzir confiança e criar exposição de dados e obrigações de privacidade. | Não solicitar localização por padrão sem decisão de produto; caso aprovada, pedir permissão contextual, explicar o uso e minimizar retenção. |
| Produto — expectativa sobre alertas | Usuários esperarem alertas de tempo severo apesar de não estarem no escopo definido. | Média | Alto: a ausência pode ser interpretada como garantia de que não há risco meteorológico. | Confirmar explicitamente se alertas fazem parte do produto; se não fizerem, delimitar o escopo e evitar comunicar a previsão como serviço de alerta. |

## Perguntas em Aberto

O briefing descreve quatro capacidades, mas não define público, limites do produto nem critérios de aceitação. As perguntas abaixo registram as principais decisões que precisam ser tomadas antes de fechar a especificação; os impactos indicam o custo de avançar sem resposta.

| Área | Pergunta em aberto | Impacto de seguir sem resposta |
| --- | --- | --- |
| Objetivo | Qual problema de negócio a aplicação deve resolver e como será medido o sucesso (por exemplo, consultas concluídas, retorno de usuários ou satisfação)? | Não será possível priorizar funcionalidades nem avaliar se o produto atende ao objetivo da empresa. |
| Público | Quem são os usuários prioritários e em quais situações consultarão o clima? | A interface pode privilegiar informações ou fluxos inadequados para o público real. |
| Plataformas | “Usar em dispositivos móveis” significa site responsivo, PWA ou aplicativos nativos? Desktop também está no escopo? | A arquitetura, os custos, a distribuição e o plano de testes podem mudar significativamente. |
| Cobertura geográfica | A busca deve aceitar cidades de qualquer país ou somente regiões específicas? | A fonte de dados, a qualidade do geocoding e os idiomas necessários podem não atender aos locais esperados. |
| Busca | A busca é enviada por formulário, oferece sugestões durante a digitação ou ambos? Quais formatos de entrada devem ser aceitos? | A interação, os requisitos de latência e o comportamento em entradas incompletas ficam indefinidos. |
| Desambiguação | Como o usuário deve distinguir cidades homônimas (país, estado, região ou coordenadas)? | O usuário pode consultar a localidade errada e tomar decisões com base em uma previsão irrelevante. |
| Seleção inicial | O que aparece antes de o usuário buscar uma cidade: tela vazia, cidade padrão, última consulta ou localização atual? | A primeira experiência não pode ser desenhada nem validada de forma consistente. |
| Geolocalização | O produto deve pedir localização do dispositivo ou a cidade será sempre escolhida manualmente? | Se decidido tarde, permissões, privacidade, tratamento de recusa e fluxo inicial exigirão mudanças. |
| Múltiplas cidades | É necessário alternar entre cidades recentes, salvar favoritas ou comparar locais? | Persistência e navegação podem precisar de dados e controles que não estarão previstos na primeira versão. |
| Clima atual | O que “clima atual” inclui: temperatura, sensação térmica, condição, umidade, vento, pressão, precipitação, UV ou outros dados? | O modelo de dados, a fonte e o layout podem ficar incompletos ou ser retrabalhados. |
| Momento da observação | “Atual” significa observação mais recente disponível ou uma condição estimada para o horário local? Deve ser exibido o horário da atualização? | Dados defasados podem parecer atuais e induzir o usuário ao erro. |
| Granularidade da previsão | A previsão será diária, horária ou ambas? | A estrutura de navegação, o volume de dados e a escolha do provedor dependem dessa decisão. |
| Campos da previsão | Quais variáveis são necessárias por período (mínima/máxima, condição, chuva, vento ou outras)? | Não há critério objetivo para considerar a previsão completa ou escolher uma API compatível. |
| Fuso horário | As datas e horas devem seguir o fuso local da cidade consultada? Como tratar mudanças de horário e locais sem fuso identificado? | O dia exibido pode não corresponder ao calendário local e os horários podem ser interpretados incorretamente. |
| Unidades | A alternância C/F afeta apenas temperatura? Quais unidades serão usadas para vento, precipitação e distância? | Valores podem aparecer em unidades inesperadas ou inconsistentes para o usuário. |
| Preferência de unidade | A unidade escolhida pelo usuário deve persistir localmente entre visitas, sem conta? | A preferência pode ser perdida ao fechar a aplicação ou persistida de uma forma não esperada. |
| Idioma e formatos | Além da interface em pt-BR, quais formatos de data, hora e números devem ser usados? | Textos ficam definidos, mas datas e valores ainda podem aparecer em convenções inesperadas. |
| Atualização dos dados | Com que frequência as condições atuais e a previsão devem ser atualizadas? Deve haver atualização automática ou ação manual? | O usuário pode ver dados desatualizados; atualizações frequentes também podem exceder limites ou aumentar custos da API. |
| Fonte de dados | Para o Open-Meteo, quais limites de uso, custos aplicáveis, cobertura, licença, atribuição e disponibilidade devem ser considerados? | A integração escolhida pode não atender à escala ou às regiões necessárias, ou impor obrigações não previstas. |
| Falhas e modo offline | O que deve ocorrer quando não houver rede ou o provedor falhar: erro, última consulta em cache ou somente nova tentativa? | O usuário pode ficar sem orientação sobre se os dados estão indisponíveis ou desatualizados. |
| Alertas | Alertas de tempo severo fazem parte do produto ou estão explicitamente fora da primeira versão? | Incluir alertas altera fontes, urgência, notificações e responsabilidade; excluí-los sem confirmação pode deixar uma necessidade crítica descoberta. |
| Privacidade | Quais dados pessoais ou de localização podem ser coletados, armazenados e por quanto tempo? | Pode haver coleta desnecessária, riscos de privacidade e obrigações legais não consideradas. |
| Acessibilidade | Qual padrão e nível de conformidade são exigidos, e há tecnologias assistivas prioritárias para validação? | Barreiras para pessoas com deficiência podem permanecer sem detecção e correção até fases tardias. |
| Responsividade | Quais larguras, orientações e tamanhos de tela devem ser suportados, e há suporte a tablet/desktop? | “Mobile” permanece subjetivo; telas ou orientações relevantes podem ficar sem cobertura de design e teste. |
| Navegadores e dispositivos | Quais navegadores, versões mínimas e sistemas operacionais são suportados? | A aplicação pode falhar em dispositivos utilizados pelo público, ou o esforço de compatibilidade pode ser estimado incorretamente. |
| Performance | Quais limites de tempo são aceitáveis para abrir a aplicação e retornar uma consulta, em quais condições de rede e percentil? | Metas não podem ser verificadas e atrasos percebidos podem comprometer o uso, especialmente em rede móvel. |
| Disponibilidade | Qual disponibilidade mensal é exigida e como indisponibilidade do provedor externo será contabilizada? | Não há critério de operação ou acordo de nível de serviço; dependências externas podem tornar a meta impraticável. |
| Escala | Quantos usuários e consultas simultâneas são esperados, inclusive em picos? | Limites de API, cache e infraestrutura podem não suportar a demanda ou gerar custos inesperados. |
| Preferências locais | Sem autenticação nem persistência de servidor, alguma preferência deve ser salva localmente no dispositivo? | A unidade e outras preferências podem se comportar de modo inesperado entre sessões ou dispositivos. |
| Critérios de aceite | Quais cenários demonstram que busca, clima atual, previsão e alternância de unidade estão concluídos corretamente? | Desenvolvimento e validação podem usar interpretações diferentes, tornando a entrega difícil de aprovar. |

## Decisões

- **Fonte de dados: Open-Meteo, sem API key.** Escolhida para fornecer geocoding e dados meteorológicos sem exigir uma chave de acesso na aplicação. Resolve a pergunta sobre qual provedor usar e a necessidade de chave; limites de uso, cobertura, licença e atribuição continuam como verificações em aberto.
- **“5 dias” = hoje + os quatro dias seguintes.** Define uma janela de cinco datas consecutivas, incluindo o dia atual, para alinhar implementação e aceite. Resolve a pergunta sobre a composição dos cinco dias; o fuso horário usado para determinar “hoje” ainda precisa ser definido.
- **Unidade padrão: Celsius.** Define o valor inicial quando o usuário ainda não escolheu outra unidade. Resolve a pergunta sobre a unidade padrão; persistência local da preferência continua em aberto.
- **Sem autenticação e sem persistência de servidor.** Mantém a primeira versão sem contas nem armazenamento de dados de usuário no backend. Resolve a necessidade de login e sincronização por conta; ainda se pode decidir se preferências serão guardadas localmente no dispositivo.
- **Idioma da interface: pt-BR.** Define o idioma dos textos e controles da aplicação. Resolve a pergunta sobre o idioma da UI; formatos de data, hora e números permanecem pendentes de confirmação.

## Suposições

- A aplicação será uma interface digital acessível por dispositivos móveis; o briefing não esclarece se também haverá suporte a desktop ou se será um aplicativo nativo.
- A consulta de clima e previsão usará Open-Meteo, conforme decisão registrada acima.
- Celsius e Fahrenheit se aplicam às temperaturas exibidas tanto no clima atual quanto na previsão.
- A busca inicia a consulta de uma cidade escolhida pelo usuário; localização automática não é requisito confirmado.
- As decisões atuais não incluem autenticação nem persistência de servidor; histórico, favoritos e persistência local ainda não foram definidos.
- As metas numéricas de performance e disponibilidade são propostas iniciais e dependem de validação com negócio e equipe técnica.

## Personas

> Personas hipotéticas derivadas do briefing; devem ser validadas com usuários antes de orientar decisões definitivas de produto.

### Ana — Decisora da rotina diária

- **Objetivo principal:** verificar rapidamente as condições atuais da cidade onde está para decidir como se preparar para o dia.
- **Contexto de uso:** principalmente mobile, em momentos curtos antes de sair ou durante deslocamentos; pode estar em uma rede instável.
- **Métrica de sucesso do ponto de vista dela:** encontrar a cidade correta e entender o clima atual em até 30 segundos, sem precisar repetir a busca.

### Bruno — Planejador de viagem

- **Objetivo principal:** consultar a previsão de cinco dias para uma cidade e avaliar as condições durante o período planejado.
- **Contexto de uso:** desktop ao organizar a viagem e mobile para consultas rápidas durante o deslocamento.
- **Métrica de sucesso do ponto de vista dele:** localizar o destino e revisar os cinco dias da previsão em uma única consulta, em até dois minutos.

### Lúcia — Usuária com preferência por Fahrenheit

- **Objetivo principal:** consultar o clima atual e a previsão em uma unidade de temperatura que reconhece e usa no cotidiano.
- **Contexto de uso:** mobile ao comparar informações durante o dia e desktop quando planeja com antecedência; pode alternar entre contextos ou regiões com convenções diferentes.
- **Métrica de sucesso do ponto de vista dela:** alternar para Fahrenheit e confirmar que todas as temperaturas exibidas, no clima atual e nos cinco dias, usam essa unidade sem discrepâncias.