# Especificação de Produto — Weather App

## Overview

O Weather App permitirá que usuários pesquisem cidades, consultem o clima atual e a previsão para cinco dias, alternando entre Celsius e Fahrenheit. A experiência deve funcionar em dispositivos móveis e apresentar os textos da interface em pt-BR.

Decisões confirmadas para esta versão:

- Fonte de dados: Open-Meteo, sem API key.
- Período da previsão: hoje e os quatro dias seguintes.
- Unidade inicial: Celsius.
- Sem autenticação e sem persistência de dados no servidor.
- Idioma da interface: pt-BR.

Esta especificação mantém como perguntas em aberto decisões ainda não fechadas no discovery, como plataforma de distribuição, campos meteorológicos, fuso horário, política offline e metas operacionais finais.

Para a linha de base desta especificação, assume-se provisoriamente uma aplicação web responsiva. A confirmação de desktop, PWA e dispositivos suportados permanece em OQ-01; qualquer escolha por aplicativo nativo exige revisão de escopo e arquitetura.

**Status de prontidão:** esta especificação detalha uma linha de base candidata, mas não é aprovação de release. Os pontos em aberto sobre plataforma, contrato de dados, fuso, política de falhas, termos do provedor e metas operacionais precisam de decisão registrada antes de congelar a linha de base de produção.

## Functional Requirements

- **FR-01 — Buscar cidades:** o usuário deve poder pesquisar uma cidade pelo nome e selecionar um resultado.
- **FR-02 — Identificar resultados de busca:** quando houver mais de uma cidade compatível, o sistema deve apresentar informação suficiente para distinguir os resultados, conforme os dados disponíveis do serviço.
- **FR-03 — Consultar clima atual:** após a seleção de uma cidade, o sistema deve apresentar o clima atual correspondente a ela.
- **FR-04 — Consultar previsão de cinco dias:** após a seleção de uma cidade, o sistema deve apresentar a previsão de hoje e dos quatro dias seguintes.
- **FR-05 — Alternar unidade de temperatura:** o usuário deve poder alternar entre Celsius e Fahrenheit; Celsius é a unidade inicial.
- **FR-06 — Manter seleção consistente:** clima atual e previsão devem corresponder à cidade selecionada e exibir temperaturas na unidade selecionada.
- **FR-07 — Comunicar estados da consulta:** o sistema deve informar carregamento, ausência de resultados e falhas. Deve oferecer nova tentativa para falhas transitórias de rede, timeout ou indisponibilidade temporária do serviço; não deve tratar resposta válida sem resultados ou entrada inválida como falha transitória.

## User Stories

- **US-01:** Como Ana, decisora da rotina diária, quero buscar minha cidade e consultar o clima atual para decidir como me preparar para o dia. (**FR-01, FR-03**)
- **US-02:** Como Ana, decisora da rotina diária, quero saber quando a consulta está carregando, falha ou não encontra resultados para entender o que aconteceu e como prosseguir. (**FR-07**)
- **US-03:** Como Bruno, planejador de viagem, quero distinguir cidades com nomes semelhantes para consultar a previsão do destino correto. (**FR-02**)
- **US-04:** Como Bruno, planejador de viagem, quero consultar a previsão de hoje e dos quatro dias seguintes para avaliar as condições durante o período planejado. (**FR-04**)
- **US-05:** Como Lúcia, usuária com preferência por Fahrenheit, quero alternar entre Celsius e Fahrenheit para interpretar as temperaturas na unidade que prefiro. (**FR-05, FR-06**)
- **US-06:** Como Bruno, planejador de viagem, quero que clima atual e previsão correspondam à cidade selecionada para planejar com informações do destino correto. (**FR-06**)

As personas do discovery são hipóteses e devem ser validadas com usuários; estas histórias representam necessidades a validar, não segmentos comprovados.

## Acceptance Criteria

Os critérios abaixo usam Given/When/Then e devem ser validados com fixtures determinísticas de busca e dados meteorológicos. FR-03 e FR-04 dependem da definição dos campos a exibir (OQ-04 e OQ-05); esses contratos precisam ser fechados antes de automatizar o aceite final desses campos.

### FR-01 — Buscar cidades

- **AC-01.1 — Busca com resultado:**
	- **Given:** o serviço de busca retorna uma cidade correspondente à consulta `Curitiba`.
	- **When:** o usuário envia `Curitiba`.
	- **Then:** a cidade retornada aparece como opção selecionável.
- **AC-01.2 — Seleção de resultado:**
	- **Given:** a lista contém a opção `Curitiba`.
	- **When:** o usuário seleciona essa opção.
	- **Then:** `Curitiba` passa a ser a cidade selecionada para as consultas de clima e previsão.
- **AC-01.3 — Busca sem resultados:**
	- **Given:** o serviço retorna uma lista vazia para a consulta `CidadeInexistente`.
	- **When:** a busca termina.
	- **Then:** a interface informa que nenhuma cidade foi encontrada e não seleciona uma cidade automaticamente.
- **AC-01.4 — Input vazio:**
	- **Given:** o campo de busca está vazio ou contém apenas espaços.
	- **When:** o usuário tenta enviar a busca.
	- **Then:** a interface informa em pt-BR que o nome da cidade é obrigatório, não chama o serviço de geocoding e mantém qualquer cidade já selecionada sem associá-la à busca vazia.
- **AC-01.5 — Caracteres válidos em nomes:**
	- **Given:** a fixture contém a cidade `São João d'Aliança`.
	- **When:** o usuário busca `  São João d'Aliança  `.
	- **Then:** espaços externos são removidos para a consulta, acentos e apóstrofo são preservados e a cidade correspondente aparece selecionável.

### FR-02 — Identificar resultados de busca

- **AC-02.1 — Cidades homônimas:**
	- **Given:** os resultados de teste contêm duas cidades chamadas `Springfield`, com países ou subdivisões administrativas diferentes.
	- **When:** o usuário busca `Springfield`.
	- **Then:** cada opção exibe os dados regionais disponíveis no resultado e as duas opções podem ser selecionadas separadamente.
- **AC-02.2 — Resultado único:**
	- **Given:** o serviço retorna uma única cidade para a consulta.
	- **When:** os resultados são exibidos.
	- **Then:** a opção apresenta o nome retornado e pode ser selecionada.
- **AC-02.3 — Dados insuficientes para desambiguação:**
	- **Given:** dois resultados têm o mesmo nome e os mesmos dados regionais disponíveis.
	- **When:** a lista de resultados é apresentada.
	- **Then:** ambos permanecem opções separadas, nenhum é selecionado automaticamente e todos os campos de localização disponíveis são exibidos; o critério para permitir a seleção nessas condições precisa ser definido em OQ-03.

### FR-03 — Consultar clima atual

- **AC-03.1 — Dados da cidade selecionada:**
	- **Given:** a cidade selecionada é `Curitiba` e a fixture de clima atual para ela contém a temperatura `18 °C`.
	- **When:** a consulta atual termina com sucesso.
	- **Then:** a interface identifica `Curitiba` e apresenta a temperatura `18 °C` como clima atual dessa cidade.
- **AC-03.2 — Campos adicionais do clima atual:**
	- **Given:** o catálogo aprovado de campos atuais e a fixture correspondente estão definidos (OQ-04).
	- **When:** a consulta atual termina com sucesso.
	- **Then:** cada campo do catálogo é exibido com o valor correspondente da fixture.
- **AC-03.3 — Dados atuais indisponíveis:**
	- **Given:** não há dados atuais válidos para a cidade selecionada.
	- **When:** a consulta termina.
	- **Then:** a interface informa que os dados atuais não estão disponíveis e não apresenta valores de outra cidade como atuais.
- **AC-03.4 — Resposta atual parcial:**
	- **Given:** a fixture contém a temperatura válida `18 °C` e omite um campo atual opcional do catálogo aprovado (OQ-04).
	- **When:** a consulta atual termina.
	- **Then:** `18 °C` é exibido, o campo ausente é identificado como indisponível e nenhum zero ou valor antigo é usado como substituto.
- **AC-03.5 — Campo atual obrigatório ausente:**
	- **Given:** a fixture omite um campo marcado como obrigatório no catálogo aprovado (OQ-04).
	- **When:** a consulta atual termina.
	- **Then:** o campo é identificado como indisponível; nenhum valor fabricado ou de outra cidade é mostrado e a interface indica que a resposta está incompleta.

### FR-04 — Consultar previsão de cinco dias

- **AC-04.1 — Janela de cinco dias:**
	- **Given:** a data de referência da fixture é 10/03/2026 e a cidade selecionada tem uma previsão válida para cinco datas consecutivas.
	- **When:** a previsão é carregada.
	- **Then:** são exibidas as datas 10/03, 11/03, 12/03, 13/03 e 14/03, uma vez cada e em ordem cronológica. O fuso usado para determinar a data de referência permanece sujeito a OQ-06.
- **AC-04.2 — Valores da previsão:**
	- **Given:** a fixture da cidade selecionada contém temperaturas de 0 °C em 10/03/2026, 5 °C em 11/03/2026, 10 °C em 12/03/2026, 15 °C em 13/03/2026 e 20 °C em 14/03/2026.
	- **When:** a previsão é carregada.
	- **Then:** cada uma das cinco datas exibe sua temperatura correspondente em Celsius, sem associar o valor de outra cidade ou data.
- **AC-04.3 — Campos adicionais da previsão:**
	- **Given:** o catálogo aprovado de campos e granularidade da previsão está definido (OQ-05), com valores correspondentes na fixture.
	- **When:** a previsão é carregada.
	- **Then:** cada data exibe os valores da fixture para todos os campos aprovados.
- **AC-04.4 — Previsão parcial:**
	- **Given:** a fixture não contém previsão para uma ou mais das cinco datas solicitadas.
	- **When:** a previsão é carregada.
	- **Then:** as datas sem dados são identificadas como indisponíveis e nenhum valor de outra data é duplicado para preenchê-las.
- **AC-04.5 — Campo ausente em uma data:**
	- **Given:** a fixture contém temperaturas para as cinco datas, mas omite um campo opcional em uma delas (OQ-05).
	- **When:** a previsão é carregada.
	- **Then:** as temperaturas disponíveis permanecem associadas às datas corretas e o campo ausente é identificado como indisponível, sem substituir dados.
- **AC-04.6 — Campo obrigatório ausente:**
	- **Given:** a fixture omite, em uma data, um campo marcado como obrigatório no catálogo aprovado (OQ-05).
	- **When:** a previsão é carregada.
	- **Then:** a data é identificada como incompleta e o campo ausente como indisponível; valores de outras datas não são usados como substitutos.

### FR-05 — Alternar unidade de temperatura

- **AC-05.1 — Unidade inicial:**
	- **Given:** não há preferência de unidade definida para a experiência.
	- **When:** dados de temperatura são exibidos pela primeira vez.
	- **Then:** todas as temperaturas são apresentadas em Celsius.
- **AC-05.2 — Conversão para Fahrenheit:**
	- **Given:** os dados atuais e a previsão contêm temperaturas de 0 °C e 20 °C.
	- **When:** o usuário seleciona Fahrenheit.
	- **Then:** os valores correspondentes são exibidos como 32 °F e 68 °F, e todas as temperaturas visíveis indicam °F.
- **AC-05.3 — Conversão para Celsius:**
	- **Given:** os dados atuais e a previsão contêm temperaturas de 32 °F e 68 °F.
	- **When:** o usuário seleciona Celsius.
	- **Then:** os valores correspondentes são exibidos como 0 °C e 20 °C, e todas as temperaturas visíveis indicam °C.
- **AC-05.4 — Valor negativo exato:**
	- **Given:** uma temperatura de referência é `-40 °C`.
	- **When:** o usuário seleciona Fahrenheit.
	- **Then:** a temperatura é exibida como `-40 °F`.

### FR-06 — Manter seleção consistente

- **AC-06.1 — Troca de cidade:**
	- **Given:** a interface exibe dados de `Cidade A` e a fixture de `Cidade B` contém valores diferentes.
	- **When:** o usuário seleciona `Cidade B`.
	- **Then:** o nome, o clima atual e a previsão exibidos correspondem a `Cidade B`, sem valores de `Cidade A` apresentados como atuais.
- **AC-06.2 — Unidade consistente:**
	- **Given:** clima atual e previsão estão visíveis e Celsius está selecionado.
	- **When:** o usuário seleciona Fahrenheit.
	- **Then:** todas as temperaturas atuais e previstas visíveis passam a indicar Fahrenheit.

### FR-07 — Comunicar estados da consulta

- **AC-07.1 — Carregamento:**
	- **Given:** uma busca ou consulta foi iniciada e ainda não foi concluída.
	- **When:** a resposta permanece pendente.
	- **Then:** a interface apresenta um estado de carregamento.
- **AC-07.2 — Erro recuperável e nova tentativa:**
	- **Given:** a consulta da cidade selecionada falha por timeout, erro de conexão ou indisponibilidade temporária do serviço.
	- **When:** o erro é apresentado.
	- **Then:** a interface informa que os dados não foram carregados e oferece uma ação de nova tentativa; ao acioná-la, a consulta para a mesma cidade é repetida.
- **AC-07.3 — Erro sem dados válidos:**
	- **Given:** a consulta falha ou retorna dados inválidos.
	- **When:** a interface apresenta o resultado da consulta.
	- **Then:** os dados inválidos não são exibidos como uma previsão válida e a interface informa que os dados não estão disponíveis.
- **AC-07.4 — Falha de geocoding:**
	- **Given:** o serviço de geocoding retorna falha, em vez de uma resposta bem-sucedida sem resultados.
	- **When:** a busca termina.
	- **Then:** a interface comunica uma falha de pesquisa, não a ausência de cidades, e não inicia uma consulta meteorológica.
- **AC-07.5 — Timeout:**
	- **Given:** uma resposta de geocoding, clima atual ou previsão permanece pendente além do limite configurado para o cenário (OQ-16).
	- **When:** o relógio de teste ultrapassa esse limite.
	- **Then:** o carregamento termina, uma mensagem de timeout é apresentada e a nova tentativa, quando disponível, repete a consulta para a mesma cidade.
- **AC-07.6 — Idioma dos estados:**
	- **Given:** a interface apresenta carregamento, ausência de resultados ou falha.
	- **When:** o usuário visualiza o estado.
	- **Then:** o texto e as ações apresentados estão em pt-BR.

### Critérios de aceite não-funcionais

- **AC-NFR-01:** Na linha de base web responsiva, nas larguras de 320 px, 768 px e 1440 px, o usuário consegue buscar e selecionar uma cidade, consultar clima atual e previsão e alternar a unidade sem rolagem horizontal da página. A matriz final de dispositivos e desktop permanece pendente (OQ-01).
- **AC-NFR-02:** Nos fluxos FR-01 a FR-07, uma avaliação automatizada e manual contra WCAG 2.2 nível AA não encontra violações A/AA pendentes; os fluxos podem ser concluídos por teclado, controles têm nomes acessíveis, o foco é visível e o contraste atende aos critérios aplicáveis.
- **AC-NFR-03:** Performance e disponibilidade só serão aceitas contra metas aprovadas e condições de medição definidas; os valores atuais são propostas (OQ-12).
- **AC-NFR-04:** No ambiente de produção, o acesso por HTTP é redirecionado para HTTPS e todas as requisições da aplicação e integrações usam HTTPS.

## Non-Functional Requirements

- **NFR-01 — Responsividade:** na linha de base web responsiva, manter as funções principais utilizáveis em larguras de 320 px a 1440 px, sem rolagem horizontal. A matriz definitiva de dispositivos e navegadores depende de OQ-01.
- **NFR-02 — Acessibilidade:** atender WCAG 2.2 nível AA, incluindo navegação por teclado, foco visível, nomes acessíveis para controles e contraste adequado.
- **NFR-03 — Idioma:** apresentar textos, controles e mensagens da interface em pt-BR. Formatos de data, hora e números ainda precisam ser confirmados.
- **NFR-04 — Resiliência:** tratar falhas de conexão, respostas inválidas e indisponibilidade da fonte sem deixar a interface em estado indefinido nem rotular dados indisponíveis como atuais.
- **NFR-05 — Performance (proposto):** conteúdo inicial em até 2 segundos e conclusão de consulta em até 3 segundos no percentil 95. As condições de rede, o ponto de início/fim da medição e a aprovação dessas metas estão pendentes.
- **NFR-06 — Disponibilidade (proposto):** meta mensal de pelo menos 99,5% para a aplicação. A disponibilidade do serviço externo deve ser acompanhada separadamente; a meta final e a forma de cálculo estão pendentes.
- **NFR-07 — Privacidade e persistência:** não exigir conta nem armazenar dados de usuário no servidor. Qualquer coleta de localização ou persistência local depende de decisão explícita.
- **NFR-08 — Segurança de transporte:** servir a aplicação em HTTPS e usar HTTPS em todas as integrações externas.

NFR-05 e NFR-06 são propostas iniciais do discovery, não compromissos aprovados. Não devem ser considerados SLOs finais antes da validação de negócio e técnica.

## Edge Cases

| Caso | Comportamento esperado |
| --- | --- |
| Input vazio ou composto apenas por espaços | Remover espaços externos e impedir o envio se não restar texto. Mostrar em pt-BR que o nome da cidade é obrigatório, não chamar o geocoding e manter a cidade já selecionada sem associá-la à busca vazia. |
| Cidade inexistente | Se a busca válida não encontrar uma cidade, informar que a cidade não foi encontrada, permitir corrigir a consulta e não iniciar consultas meteorológicas para um resultado inexistente. Dados de uma seleção anterior não devem ser apresentados como resultado da nova busca. |
| Caracteres especiais | Preservar caracteres válidos em nomes de cidade, incluindo acentos, hífens e apóstrofos; normalizar apenas espaços externos. Se não houver correspondência, apresentar o estado sem resultados, sem alterar silenciosamente o texto digitado. |
| Geocoding sem resultados | Tratar uma resposta vazia bem-sucedida como “nenhuma cidade encontrada”, não como falha do serviço. Não selecionar uma cidade automaticamente nem consultar a previsão. |
| Falha de API | Informar que a consulta falhou em pt-BR, encerrar o estado de carregamento e não exibir dados inválidos como atuais. Oferecer nova tentativa quando o erro for recuperável; distinguir erro de geocoding de erro de clima/previsão quando possível. |
| Timeout | Ao atingir o limite de espera aprovado, encerrar o carregamento e informar que a consulta demorou demais. Manter a cidade selecionada e permitir nova tentativa sem duplicar resultados. O limite e os erros que permitem nova tentativa estão pendentes em OQ-16. |
| Resposta parcial | Exibir os campos e datas válidos recebidos; marcar dados ausentes como indisponíveis e nunca substituir valores ausentes por zero, dados antigos ou valores de outra cidade/data. A lista de campos obrigatórios depende de OQ-04 e OQ-05. |
| Cidades homônimas ou nomes transliterados | Exibir os dados regionais disponíveis para distinguir os resultados. A previsão só é carregada para a opção explicitamente selecionada. |
| Previsão com menos de cinco datas | Manter a posição/data de cada item recebido e identificar cada data sem dados como indisponível; não duplicar a previsão de outro dia. |
| Troca de cidade durante uma consulta pendente | Após a seleção de uma nova cidade, somente os dados associados à seleção atual podem ser apresentados como atuais. Uma resposta atrasada da cidade anterior não deve substituir os dados da nova seleção. |
| Alternância de unidade durante carregamento | Aplicar a unidade selecionada no momento em que os dados são exibidos, inclusive quando a resposta chega após a alternância. Todas as temperaturas visíveis devem usar a mesma unidade. |
| Perda de conexão após uma consulta bem-sucedida | Se dados em cache forem exibidos, identificá-los como dados anteriores e mostrar o horário da última atualização. Sem cache válido, informar indisponibilidade e permitir nova tentativa. A política de cache permanece sujeita a OQ-09. |
| Mudança de data no fuso da cidade | Apresentar as cinco datas conforme a regra de fuso aprovada para a cidade; testes devem usar relógio e fuso controlados. A regra final permanece pendente em OQ-06. |
| Teclado virtual, viewport estreito, ampliação de texto ou orientação móvel | Manter busca, resultados, unidade e dados meteorológicos utilizáveis sem sobreposição ou perda de controles; validar nos tamanhos/dispositivos definidos em OQ-01. |
| Recusa de permissão de localização | Como geolocalização automática não está confirmada, a aplicação não deve exigir essa permissão para a busca manual. Se a localização for aprovada futuramente, a recusa deve manter o fluxo manual disponível. |

## Assumptions

- O produto é uma aplicação digital para consulta individual; o briefing não estabelece personas validadas nem volume esperado de usuários.
- Open-Meteo será a fonte de geocoding e de dados meteorológicos, sem API key, conforme decisão registrada no discovery.
- “Hoje” no período de cinco dias significa a data local da cidade selecionada; a regra precisa ser confirmada quanto ao fuso horário.
- Celsius é a unidade inicial e C/F se aplica a temperaturas atuais e previstas.
- A interface será em pt-BR; formatos regionais e dados meteorológicos localizados ainda precisam de definição.
- A primeira versão não requer autenticação nem persistência de servidor.
- Busca manual é o fluxo assumido; geolocalização automática não será solicitada sem decisão e consentimento explícitos.
- As metas de performance, disponibilidade e sucesso das personas são propostas, não resultados medidos nem compromissos aprovados.

## Risks

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Indisponibilidade, limite de requisições ou alteração do Open-Meteo | Média | Alto: a consulta principal pode falhar. | Verificar limites e termos antes da entrega; definir timeout, tratamento de erro, cache e nova tentativa compatíveis com as condições do serviço. |
| Dados defasados, parciais ou interpretados no fuso errado | Média | Alto: usuários podem tomar decisões com informação enganosa. | Definir semântica e frescor dos dados; validar respostas e apresentar horário/fuso quando aprovado. |
| Cidade homônima selecionada incorretamente | Média | Alto: clima e previsão podem ser de outro local. | Exibir dados de desambiguação disponíveis e permitir confirmar a localidade. |
| Plataforma e dispositivos suportados não definidos | Alta | Médio: arquitetura, UX e testes podem precisar ser refeitos. | Confirmar web responsiva/PWA/nativo, desktop, navegadores e dispositivos antes do plano técnico. |
| Metas propostas de performance/disponibilidade serem inadequadas | Média | Alto: custo ou capacidade operacional podem ser subestimados. | Aprovar metas, escopo de medição, rede de referência e separação da dependência externa. |
| Campos e granularidade da previsão permanecerem ambíguos | Alta | Médio: retrabalho de interface e divergência na aceitação. | Fechar o contrato de dados e critérios de aceite antes da implementação. |
| Conversão ou rótulo de unidade inconsistente | Baixa | Alto: reduz confiança nos dados. | Validar conversões, arredondamento e unidade em todas as temperaturas exibidas. |
| Barreiras de acessibilidade impedirem o uso dos fluxos principais | Média | Alto: pessoas com deficiência podem não conseguir consultar ou interpretar os dados. | Avaliar os fluxos FR-01 a FR-07 contra WCAG 2.2 nível AA, com testes automatizados e manuais, e corrigir violações A/AA antes do aceite. |
| Tráfego sem criptografia expor consultas ou permitir alteração dos dados em trânsito | Baixa | Alto: dados podem ser interceptados ou modificados. | Exigir HTTPS no ambiente de produção para o site e todas as integrações; validar redirecionamento de HTTP e ausência de endpoints externos em HTTP. |
| Localização ou preferências serem persistidas sem decisão de privacidade | Baixa | Alto: coleta desnecessária e perda de confiança. | Não coletar localização nem persistir preferências fora do dispositivo sem aprovação explícita. |

## Out of Scope

- Autenticação, contas de usuário e persistência de dados no servidor nesta versão.
- Alertas de tempo severo, notificações push e garantias de prevenção de risco meteorológico.
- Geolocalização automática, favoritos, histórico e comparação de várias cidades como funcionalidades confirmadas.
- Garantia de funcionamento offline. Cache de última consulta só será incluído após definição de política de frescor e privacidade.
- Aplicativos nativos não fazem parte da linha de base provisória. A instalação como PWA e o suporte a desktop também não estão confirmados; OQ-01 deve fechar o formato de entrega antes da arquitetura. Se uma dessas opções for aprovada, esta seção e os critérios de compatibilidade devem ser revisados.

Os itens não confirmados acima ficam fora da linha de base desta especificação até serem aprovados como mudança de escopo.

## Open Questions

1. **OQ-01 — Plataforma e compatibilidade:** confirma-se a linha de base como site responsivo? PWA ou aplicativo nativo são necessários? Desktop, tablets, navegadores, versões mínimas e larguras de teste são suportados?
2. **OQ-02 — Cobertura geográfica:** a busca precisa atender cidades de quais países e regiões?
3. **OQ-03 — Busca e seleção inicial:** haverá sugestões durante a digitação? Quais formatos de entrada, ordenação, estado inicial e campos mínimos para desambiguar resultados serão usados?
4. **OQ-04 — Campos do clima atual:** quais variáveis são obrigatórias ou opcionais e qual fonte/horário define “atual”?
5. **OQ-05 — Campos e granularidade da previsão:** previsão diária, horária ou ambas? Quais variáveis são obrigatórias ou opcionais para cada data?
6. **OQ-06 — Fuso e datas:** “hoje” e as datas da previsão seguem o fuso local da cidade? Como lidar com transições de horário e virada do dia durante uma consulta?
7. **OQ-07 — Unidades e apresentação:** C/F se aplica apenas à temperatura? Qual regra de arredondamento e precisão será usada e quais unidades representam vento e precipitação?
8. **OQ-08 — Atualização:** com que frequência os dados serão atualizados e como a interface indicará a idade dos dados?
9. **OQ-09 — Falhas e offline:** há cache de última consulta? Por quanto tempo os dados podem ser mostrados como recentes e quais erros permitem nova tentativa?
10. **OQ-10 — Preferências locais:** a escolha de unidade persiste localmente entre visitas, apesar da ausência de contas e armazenamento de servidor?
11. **OQ-11 — Uso do Open-Meteo:** quais limites, cobertura, licença, atribuição e custos aplicáveis valem para o uso pretendido?
12. **OQ-12 — Metas de qualidade:** as metas propostas de performance (2 s e 3 s no p95) e disponibilidade (99,5% ao mês) são aprovadas? Quais rede/dispositivo, pontos inicial e final da medição, janela de cálculo, manutenção planejada e falhas externas entram na medição?
13. **OQ-13 — Negócio e validação:** qual objetivo de negócio e métrica de sucesso serão usados? As personas e os tempos propostos serão validados com usuários?
14. **OQ-14 — Escala e operação:** qual volume de usuários/consultas é esperado e quem monitora e responde a falhas do provedor?
15. **OQ-15 — Alertas:** alertas meteorológicos estão explicitamente fora da primeira versão ou precisam ser considerados em uma fase posterior?
16. **OQ-16 — Timeout e nova tentativa:** qual o tempo máximo de espera por tipo de consulta (geocoding, clima atual e previsão), e quais classes de falha permitem nova tentativa?