# Revisão Arquitetural do Discovery

## Diagnóstico

O discovery em [discovery.md](discovery.md) já reúne decisões, requisitos, riscos e perguntas relevantes, mas ainda não deve ser tratado como base fechada para arquitetura ou critérios de aceite. É possível começar uma especificação mantendo as lacunas explícitas; antes de congelá-la, convém resolver os pontos abaixo.

## Pontos de Atenção

### Inconsistências e risco de retrabalho

- **Fonte de dados:** a seção Contexto ainda afirma que o provedor não foi definido, embora Open-Meteo esteja registrado como decisão. A mitigação de riscos também sugere comparar provedores. Atualizar essas referências para refletir a escolha e avaliar seus termos, limites, cobertura, custos aplicáveis e atribuição.
- **Plataforma:** o escopo ainda não esclarece se o produto será web responsivo, PWA ou aplicativo nativo, nem se desktop é suportado. Ao mesmo tempo, o documento define larguras de 320–1440 px e inclui personas que usam desktop. A decisão afeta arquitetura, distribuição, compatibilidade e testes.
- **Semântica dos cinco dias:** a janela está definida como hoje mais quatro dias, mas o fuso horário que determina “hoje” permanece aberto. Sem uma regra explícita, a previsão pode começar em uma data diferente da esperada para a cidade consultada.
- **Requisitos sobrepostos:** RF4 e RF6 cobrem a aplicação da unidade escolhida às temperaturas. Consolidá-los evita critérios divergentes. RF5 também precisa esclarecer quando a ação de tentar novamente está disponível.

### Lacunas para especificação

- **Contrato de dados:** definir os campos do clima atual e da previsão, granularidade diária ou horária, momento de referência, fuso horário, atualização e indicação de dados desatualizados.
- **Busca e fluxo inicial:** decidir cobertura geográfica, formatos de entrada, sugestões, desambiguação, seleção inicial e tratamento de resultados vazios.
- **Unidades e localidade:** confirmar se C/F se aplica somente à temperatura, se a preferência persiste localmente e quais convenções de data, hora e números usar além do idioma pt-BR.
- **Falhas e conexão:** especificar o comportamento offline, cache, erros do provedor, nova tentativa e como sinalizar a idade de dados em cache.
- **Critérios de qualidade:** as metas de 2 s para carregamento, 3 s para consulta no p95 e 99,5% de disponibilidade são propostas. Faltam aprovação, condições de rede, definição do que conta como carregamento concluído e separação da disponibilidade da aplicação e do provedor.
- **Personas e sucesso:** as personas são hipóteses; metas como concluir a consulta em 30 segundos ou revisar cinco dias em dois minutos precisam de validação antes de virarem critérios de aceite.
- **Condições do provedor:** confirmar limites de uso, cobertura geográfica, licença, atribuição exigida e custos aplicáveis para o contexto de uso previsto.

## Condições para Avançar com Segurança

1. Definir plataforma, dispositivos e navegadores suportados.
2. Fechar a semântica do clima atual e da previsão, incluindo campos, granularidade, fuso e frescor dos dados.
3. Validar as condições de uso do Open-Meteo para a cobertura e o volume esperados.
4. Especificar os fluxos de busca, estados iniciais, falhas e uso sem conexão.
5. Aprovar metas verificáveis de performance, disponibilidade e acessibilidade, com critérios de medição e teste.
6. Consolidar requisitos sobrepostos, validar as hipóteses das personas e definir critérios de aceite para cada capacidade principal.

## Conclusão

O material está pronto para orientar a elaboração de uma primeira especificação com pendências identificadas. Não é recomendável fechar a arquitetura ou os critérios de aceite até que plataforma, contrato de dados, semântica da previsão e condições do provedor estejam esclarecidos.