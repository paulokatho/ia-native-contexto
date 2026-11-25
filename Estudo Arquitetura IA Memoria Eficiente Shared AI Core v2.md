# Arquitetura de IA modular, memória eficiente e conhecimento partilhado
## Estudo técnico, decisões estratégicas e proposta de repositório comum
 
**Data:** 24 de novembro de 2026  
**Estado:** Documento de referência e proposta arquitetural — revisão ampliada com AI Control Plane  
**Âmbito:** Conhecimento transversal a todos os projetos, produtos e negócios que venham a ser desenvolvidos.
 
---
 
## 1. Objetivo deste documento
 
Este documento reúne o estudo sobre **Memory Caching**, arquiteturas recorrentes e híbridas, alternativas de investigação e modelos disponibilizados por empresas como Google, IBM e NVIDIA. Converte também essas aprendizagens numa orientação estratégica para os projetos que serão desenvolvidos separadamente com Claude Code, GitLab e outras ferramentas.
 
A ideia central é construir produtos que possam evoluir sem ficarem presos:
 
- a um único fornecedor de IA;
- a um único modelo;
- a uma única arquitetura de rede neuronal;
- a uma única modalidade de contratação (gratuita, paga, local, cloud ou infraestrutura do cliente);
- a uma única estratégia de memória, recuperação ou execução.
A aplicação deve conseguir escolher a solução adequada ao problema e ao orçamento do cliente, preservando qualidade, segurança, manutenção e valor entregue.
 
**Princípio estratégico interno:** modularidade, substituibilidade e adaptação ao contexto do cliente são parte da nossa competência central. A implementação concreta, os mecanismos de seleção, os critérios internos e a forma como combinamos componentes constituem conhecimento proprietário. Não é necessário expor aos clientes toda a estratégia interna para lhes entregar os benefícios.
 
---
 
## 2. Resumo executivo
 
O artigo **Memory Caching: RNNs with Growing Memory**, publicado nos anais da ICML 2026, propõe melhorar modelos recorrentes através do armazenamento de checkpoints dos seus estados internos. O objetivo é permitir que a capacidade efetiva de memória cresça com o comprimento da sequência, sem simplesmente reproduzir o custo de uma arquitetura Transformer convencional.
 
A contribuição é relevante, mas não significa que o Google tenha resolvido definitivamente o problema da memória das IAs ou que os Transformers tenham sido ultrapassados. Os próprios resultados descritos pelos autores indicam que os Transformers continuam a obter a melhor precisão nas tarefas de recuperação dentro do contexto; as variantes de Memory Caching aproximam-se e superam referências recorrentes anteriores.
 
A investigação deve ser entendida como parte de uma evolução mais ampla:
- **Memory Caching:** checkpoints de estados recorrentes.
- **MARCH:** âncoras de estado recorrente com chaves de conteúdo para encaminhar consultas para estados históricos.
- **Mamba-3:** evolução de modelos State Space, com melhorias na recorrência, representação do estado e utilização de múltiplas entradas/saídas.
- **Modelos híbridos comerciais/abertos:** combinam componentes Mamba/SSM e Transformer, procurando equilibrar eficiência e recuperação precisa.
A decisão empresarial mais importante não é escolher hoje um vencedor absoluto. É construir uma arquitetura que permita testar, trocar e combinar modelos e serviços, com avaliação objetiva e sem reescrever o produto.
 
### Recomendação principal
 
Criar um **repositório central privado de conhecimento e capacidades transversais**, separado dos repositórios de cada produto. Cada projeto consome uma versão identificada desse conhecimento comum, em vez de manter cópias manuais divergentes.
 
Nome recomendado, a validar:
- Repositório: `ai-platform-core` ou `shared-ai-knowledge`
- Nome funcional: **Shared AI Core — Knowledge, Skills & Architecture**
- Diretório conceptual: `knowledge/`, `skills/`, `architecture/`, `standards/`, `templates/`
O nome `skills_plug_and_play` pode ser utilizado como nome de uma pasta ou coleção de capacidades, mas não deve misturar indiscriminadamente documentação, código executável, segredos, configurações específicas de clientes e instruções para agentes.
 
---
 
## 3. O problema técnico: memória e contexto em modelos de IA
 
### 3.1 Transformer
 
Os Transformers utilizam mecanismos de atenção para relacionar tokens de uma sequência. No caso típico de inferência autoregressiva, o modelo mantém um Key-Value Cache (KV cache) para reutilizar chaves e valores dos tokens anteriores.
 
Vantagens:
- forte capacidade de relacionar partes diferentes do contexto;
- recuperação detalhada de informação presente na janela de contexto;
- ecossistema muito desenvolvido e ampla disponibilidade de modelos.
Custos e limitações:
- o KV cache cresce com o contexto e pode consumir muita memória;
- a atenção pode tornar o processamento de sequências longas dispendioso;
- contexto longo não garante, por si só, recuperação perfeita ou raciocínio correto.
Nota: a expressão “complexidade quadrática” deve ser usada com precisão. A atenção densa convencional tem custo quadrático em comprimento de sequência durante o cálculo de atenção. Na inferência autoregressiva, o custo por novo token e a utilização do KV cache têm comportamentos próprios; não se deve resumir todos os custos do Transformer a uma única fórmula.
 
### 3.2 RNN e modelos recorrentes
 
Uma rede recorrente atualiza um estado interno à medida que recebe novos elementos. Em vez de manter necessariamente todos os tokens anteriores, transporta uma representação resumida do histórico.
 
Vantagens:
- potencial para processamento sequencial eficiente;
- estado de tamanho fixo em muitas arquiteturas recorrentes;
- menor necessidade de guardar o KV cache completo.
Limitação histórica:
- comprimir todo o passado num estado fixo pode fazer perder detalhes;
- informação antiga pode ser sobrescrita ou tornar-se difícil de recuperar;
- tarefas de recuperação exata e de contexto longo podem ser difíceis.
### 3.3 Memory Caching
 
O artigo estudado introduz o armazenamento de checkpoints dos estados internos da rede recorrente. Em vez de depender apenas do estado mais recente, o sistema conserva estados históricos que podem ser agregados ou selecionados.
 
A proposta procura criar um compromisso entre:
- memória recorrente fixa;
- memória crescente associada a Transformers;
- precisão de recuperação;
- custo computacional e de memória.
O artigo apresenta quatro variantes, incluindo agregação com gates e mecanismos seletivos/esparsos.
 
**O que isto não é:**
- não é simplesmente uma cache HTTP ou Redis;
- não é uma base de dados vetorial;
- não é RAG;
- não é memória persistente de um agente;
- não é uma funcionalidade que possa ser ativada em qualquer modelo comercial sem suporte arquitetural.
Memory Caching é uma técnica ao nível da arquitetura/modelo recorrente.
 
---
 
## 4. Artigo científico principal
 
### Memory Caching: RNNs with Growing Memory
 
- **Autores:** Ali Behrouz, Zeman Li, Yuan Deng, Peilin Zhong, Meisam Razaviyayn e Vahab Mirrokni.
- **Publicação:** Proceedings of the 43rd International Conference on Machine Learning (ICML 2026).
- **Série:** Proceedings of Machine Learning Research, volume 306, páginas 7342–7360.
- **Preprint:** arXiv:2602.24281.
- **Tema:** aumentar a capacidade efetiva de memória de modelos recorrentes através de checkpoints dos estados internos.
Links originais:
- Página oficial PMLR: https://proceedings.mlr.press/v306/behrouz26a.html
- PDF oficial: https://proceedings.mlr.press/v306/behrouz26a/behrouz26a.pdf
- arXiv: https://arxiv.org/abs/2602.24281
### Pergunta de investigação
 
É possível melhorar a capacidade de recuperação de modelos recorrentes, mantendo parte da eficiência que torna as arquiteturas recorrentes interessantes para sequências longas?
 
### Contribuição
 
Os autores propõem guardar checkpoints de estados recorrentes e utilizá-los para aumentar a memória efetiva. O mecanismo permite um compromisso configurável entre o estado fixo das RNNs e a memória crescente associada à atenção dos Transformers.
 
### Resultados e interpretação
 
Os resultados apresentados em modelação de linguagem e compreensão de contexto longo indicam melhorias face a referências recorrentes. Nas tarefas de recuperação dentro do contexto, os Transformers continuam a apresentar a melhor precisão; as variantes de Memory Caching aproximam-se e superam modelos recorrentes de referência.
 
### Limitações e cautelas
 
1. Os resultados dependem das tarefas, modelos, dados e configurações avaliadas.
2. Não demonstram superioridade universal sobre Transformers.
3. Não demonstram que qualquer aplicação empresarial deva trocar Transformers por RNNs.
4. Não provam que um sistema possa memorizar indefinidamente sem custos.
5. A técnica exige modelos construídos ou adaptados para a suportar.
6. Um resultado em benchmark não substitui avaliação com dados, cargas e requisitos reais do produto.
**Conclusão:** artigo científico relevante e adequado para a nossa biblioteca, mas deve ser tratado como uma linha arquitetural em evolução, não como solução universal ou produto pronto a integrar diretamente.
 
---
 
## 5. Alternativas e linhas de investigação relacionadas
 
### 5.1 MARCH — Memory-Anchor Routing across Context History
 
**Título apresentado no arXiv:** *MARCH: Scaling Recurrent Memory with Content-Routed State Anchors*  
**Autores:** Ming Zhang, Kaisen Yang, Shu Yu, Ermo Hua, Ning Ding, Xia Hu, Bowen Zhou, Chaochao Lu e Youbang Sun.  
**Data do preprint:** 12 de agosto de 2026.  
**Estado:** preprint; não assumir revisão por pares sem confirmação editorial posterior.
 
Link: https://arxiv.org/abs/2608.12435
 
MARCH também conserva checkpoints de estados recorrentes, mas associa a cada âncora uma chave compacta condicionada pelo conteúdo. Uma consulta gera uma chave de pesquisa e encaminha a recuperação para estados históricos relevantes.
 
Contributo:
- banco de âncoras de estado que pode crescer com o contexto;
- recuperação condicionada pelo conteúdo;
- compromisso configurável entre resolução histórica e custo de memória;
- agregação de âncoras históricas com o estado atual.
Os autores relatam resultados favoráveis face a várias variantes de atenção linear em tarefas de raciocínio, LongBench e recuperação em contexto.
 
**Cautela:** os resultados são os reportados pelos autores e não constituem uma comparação universal com todos os Transformers ou modelos comerciais. Por ser preprint, deve ser acompanhado e validado independentemente.
 
### 5.2 Mamba-3
 
**Título:** *Mamba-3: Improved Sequence Modeling using State Space Principles*  
**Conferência:** ICLR 2026.  
**Autores:** Aakash Sunil Lahoti, Kevin Li, Berlin Chen, Caitlin Wang, Aviv Bick, J. Zico Kolter, Tri Dao e Albert Gu.
 
Links:
- ICLR: https://proceedings.iclr.cc/paper_files/paper/2026/hash/8abd2043b71a074278d5f687947bff9c-Abstract-Conference.html
- PDF OpenReview: https://openreview.net/pdf/09c67b76b7d1ad254ea30eeba839b8592de9bd78.pdf
Mamba-3 é uma evolução dos State Space Models (SSMs), com:
- recorrência mais expressiva;
- atualização de estado com valores complexos, para melhorar o acompanhamento de estado;
- formulação MIMO (multi-input, multi-output), procurando melhorar a qualidade sem aumentar proporcionalmente a latência de descodificação.
Na escala de 1,5 mil milhões de parâmetros, o artigo reporta ganhos médios em tarefas downstream face a Gated DeltaNet e resultados adicionais com a variante MIMO. Também apresenta experiências em que consegue perplexidade comparável à do Mamba-2 com metade do tamanho do estado.
 
**Interpretação:** Mamba-3 é uma linha importante para eficiência e modelação sequencial. Não é uma solução idêntica ao Memory Caching: são propostas arquiteturais relacionadas, mas distintas.
 
### 5.3 ATLAS
 
**Título:** *ATLAS: ...* — consultar a página oficial PMLR para o título, autores e detalhes bibliográficos completos antes de registar uma referência final.
 
Link de referência: https://proceedings.mlr.press/v306/behrouz26b.html
 
ATLAS pertence à mesma linha de investigação sobre memória neural e aprendizagem de contexto. Deve ser estudado em conjunto com Memory Caching, mas não se devem inferir detalhes técnicos, resultados ou equivalência entre as propostas sem leitura direta do artigo.
 
**Ação de investigação:** confirmar título completo, autores, método, benchmarks e diferenças concretas face a Memory Caching antes de utilizar esta referência em decisões de produto.
 
---
 
## 6. Implementações e modelos disponibilizados
 
A investigação científica e a disponibilidade de modelos são coisas diferentes. Um artigo pode propor uma arquitetura sem disponibilizar pesos utilizáveis; um modelo pode implementar uma arquitetura híbrida sem implementar a técnica específica de Memory Caching.
 
### 6.1 IBM Granite 4.0
 
Documentação IBM:
https://www.ibm.com/granite/docs/models/granite4-0
 
A documentação descreve a família Granite 4.0 como uma arquitetura híbrida Mamba-2/Transformer, com Mixture of Experts (MoE) em determinados modelos. A IBM reporta, em comparações específicas, redução superior a 70% na utilização de memória e inferência até duas vezes mais rápida face a modelos comparáveis.
 
**Atenção:** a própria página indica que a documentação já não está a ser atualizada. Antes de qualquer adoção, confirmar os modelos, licenças, pesos, requisitos e documentação atuais nos canais oficiais/Hugging Face/GitHub.
 
Interesse:
- possibilidade de experimentar modelos com pesos disponibilizados;
- execução local ou em infraestrutura controlada, dependendo do modelo e dos recursos;
- alternativa a modelos exclusivamente disponibilizados por API paga;
- potencial para cenários com requisitos de privacidade ou controlo de infraestrutura.
Não assumir que os números promocionais se repetem no nosso hardware, idioma, carga ou tarefa.
 
### 6.2 NVIDIA Nemotron 3
 
Fontes oficiais:
- Família Nemotron 3: https://research.nvidia.com/labs/nemotron/Nemotron-3/
- Nemotron 3 Ultra: https://research.nvidia.com/labs/nemotron/Nemotron-3-Ultra/
- Relatório técnico Ultra: https://research.nvidia.com/labs/nemotron/files/NVIDIA-Nemotron-3-Ultra-Technical-Report.pdf
- Modelos e documentação: https://developer.nvidia.com/topics/ai/nemotron
A família Nemotron 3 utiliza uma arquitetura híbrida Mamba-Transformer com Mixture of Experts. A NVIDIA descreve:
- componentes Mamba para eficiência em sequências longas;
- componentes de atenção para recuperação mais precisa;
- LatentMoE em determinadas variantes;
- Multi-Token Prediction;
- contextos até 1 milhão de tokens em modelos da família.
O relatório técnico do Nemotron 3 Ultra descreve um modelo com 550 mil milhões de parâmetros totais e 55 mil milhões ativos, com contexto até 1 milhão de tokens, e apresenta resultados de throughput em comparações específicas.
 
**Cautelas de adoção:**
- verificar a licença exata de cada checkpoint;
- confirmar requisitos de GPU e memória;
- distinguir pesos base, modelos pós-treinados e versões quantizadas;
- medir desempenho no hardware que realmente será utilizado;
- confirmar compatibilidade com o runtime e o harness;
- não confundir contexto máximo anunciado com capacidade de utilizar todo o contexto com precisão.
### 6.3 Comparação resumida
 
| Solução | Tipo | Principal interesse | Maturidade considerada |
|---|---|---|---|
| Google Memory Caching | Técnica/modelo recorrente | Checkpoints de estado e memória crescente | Artigo ICML 2026 |
| MARCH | Arquitetura recorrente | Âncoras com encaminhamento por conteúdo | Preprint |
| Mamba-3 | Arquitetura SSM | Eficiência e capacidade de estado | Artigo ICLR 2026 |
| IBM Granite 4.0 | Família de modelos híbridos | Modelos para experimentar e integrar | Modelos/documentação disponíveis; confirmar versões |
| NVIDIA Nemotron 3 | Família de modelos híbridos MoE | Contexto longo e workloads agentivos | Modelos e relatórios disponibilizados |
 
### Conclusão comparativa
 
Não existe base suficiente para declarar um vencedor absoluto. As propostas têm objetivos, escalas, ambientes de teste e níveis de maturidade diferentes.
 
Para investigação:
1. Memory Caching;
2. MARCH;
3. Mamba-3;
4. ATLAS, após validação bibliográfica e leitura direta.
Para prototipagem:
1. IBM Granite 4.0, após confirmar versões e licença;
2. NVIDIA Nemotron, selecionando a variante adequada ao hardware;
3. outros modelos comerciais e abertos que passem os mesmos testes internos.
A seleção final deve ser feita por benchmark próprio e custo total de operação, não por popularidade ou por um único número publicado.
 
---
 
## 7. Conceitos que não devemos confundir
 
A palavra “memória” é usada para mecanismos muito diferentes.
 
| Mecanismo | Onde atua | Para que serve |
|---|---|---|
| KV Cache | Inferência do Transformer | Reutilizar chaves/valores dos tokens anteriores |
| Memory Caching | Arquitetura recorrente | Conservar checkpoints de estados internos |
| Mamba/SSM state | Arquitetura do modelo | Representar o histórico através de estado recorrente |
| RAG | Aplicação/sistema | Recuperar informação externa para fornecer contexto ao modelo |
| Vector Database | Camada de dados | Indexar e pesquisar representações vetoriais |
| Memória de agente | Aplicação/harness | Guardar factos, objetivos, decisões e estado de tarefas |
| Cache tradicional | Infraestrutura | Reutilizar resultados para reduzir latência/custo |
| Base de dados de negócio | Domínio da aplicação | Guardar registos transacionais e informação oficial |
 
Estas camadas podem coexistir. Nenhuma substitui automaticamente todas as outras.
 
### Princípio fundamental
 
**A memória de negócio não deve depender exclusivamente da memória interna de um modelo.**
 
Informação crítica — clientes, contratos, encomendas, permissões, decisões, transações e histórico oficial — deve permanecer em sistemas de dados controlados pela aplicação. O modelo pode consultar, resumir ou utilizar essa informação, mas não deve ser a única fonte de verdade.
 
---
 
## 8. Princípio estratégico: arquitetura “Lego” / Plug and Play
 
### 8.1 Visão
 
Os produtos serão desenvolvidos em projetos separados, cada um com os seus requisitos, utilizadores, dados, interface e lógica de negócio. No entanto, devem reutilizar uma base transversal de conhecimento, padrões, ferramentas, componentes e capacidades.
 
A nossa cultura técnica deve ser:
 
> Construir sistemas compostos por componentes substituíveis, com contratos claros, avaliação contínua e liberdade para escolher a melhor combinação de tecnologia para cada cliente.
 
Isto significa:
- utilizar serviços pagos quando justificam o valor;
- utilizar modelos gratuitos/open-weight quando são suficientes;
- utilizar modelos locais quando privacidade, controlo ou custo o justificam;
- combinar modelos diferentes para tarefas diferentes;
- trocar de fornecedor quando preço, qualidade, disponibilidade ou condições mudarem;
- preservar dados, memória de negócio, regras e lógica de domínio fora do fornecedor de modelo;
- manter capacidade de fallback e migração.
### 8.2 O que “não ficar preso” significa na prática
 
Não significa que a troca de fornecedor seja sempre instantânea ou sem custo. APIs, formatos de tool calling, capacidades multimodais, janelas de contexto, políticas de segurança, latência e qualidade variam.
 
A portabilidade deve ser construída com:
1. contratos internos estáveis;
2. adaptadores por fornecedor/modelo;
3. perfis de capacidades;
4. testes de compatibilidade;
5. evals comparáveis;
6. gestão de versões;
7. observabilidade;
8. estratégia de fallback;
9. separação entre dados do negócio e estado específico do fornecedor.
**Evitar o falso “menor denominador comum”:** uma abstração demasiado genérica pode esconder funcionalidades importantes e impedir tirar partido de capacidades específicas. A camada comum deve oferecer um núcleo portátil e permitir extensões específicas, explicitamente identificadas.
 
### 8.3 Escolha por cliente e por tarefa
 
A decisão pode variar consoante:
- orçamento e modelo de contratação;
- volume e frequência de utilização;
- complexidade da tarefa;
- precisão exigida;
- latência e disponibilidade;
- idioma;
- contexto necessário;
- privacidade, localização e retenção de dados;
- infraestrutura disponível;
- licenças e direitos de utilização;
- suporte e SLA.
Não devemos escolher sempre o modelo mais caro ou o mais barato. Devemos escolher a solução com melhor relação entre valor, qualidade, custo, risco e manutenção para aquele cenário.
 
### 8.4 Roteamento de modelos
 
A arquitetura pode ter uma política de seleção que encaminha tarefas para modelos diferentes.
 
Exemplo conceptual:
 
| Tipo de tarefa | Candidato de perfil | Critério dominante |
|---|---|---|
| Classificação simples | Modelo pequeno/económico | Custo e latência |
| Extração estruturada | Modelo fiável em formato estruturado | Precisão e validação |
| Raciocínio complexo | Modelo de maior capacidade | Qualidade |
| Pesquisa documental | RAG + modelo adequado | Fidelidade às fontes |
| Dados muito sensíveis | Modelo local/privado autorizado | Privacidade e controlo |
| Alto volume | Modelo eficiente/híbrido | Custo por tarefa e throughput |
| Falha de fornecedor | Alternativa compatível | Continuidade |
 
O encaminhamento deve ser baseado em requisitos e resultados medidos. Não deve ser uma regra fixa do tipo “modelo X é sempre para tarefa Y”.
 
### 8.5 AI Control Plane: seleção, tokens, previsão de custos e planeamento de atividades
 
A arquitetura comum deve prever um **AI Control Plane** (nome funcional; a designação final pode mudar) que coordena a utilização de modelos e ferramentas. Não é apenas um Model Router. É uma camada de decisão e controlo que recebe uma tarefa, avalia opções, estima recursos, propõe um plano e acompanha a execução.
 
A visão inclui seis capacidades relacionadas:
 
1. **AI Model Router:** seleciona fornecedor, modelo, variante e nível de esforço adequados à tarefa.
2. **Provider Token Intelligence:** compreende como cada fornecedor/modelo tokeniza e contabiliza entrada, saída, cache e outras categorias faturáveis.
3. **Cost Intelligence & Forecasting:** calcula custos previstos antes da execução e reconcilia previsões com utilização e custos reais.
4. **Task Intelligence:** classifica a tarefa, estima complexidade, contexto, ferramentas, iterações e critérios de sucesso.
5. **Budget & Activity Planner:** recomenda quais atividades executar, adiar, dividir ou encaminhar para outra opção, considerando orçamento, prioridade, dependências e valor.
6. **Context & Handoff:** preserva o estado necessário para transferir uma tarefa entre modelos, fornecedores ou agentes, sem presumir que o contexto interno de um fornecedor é portável.
Estas capacidades podem começar como módulos lógicos dentro de uma aplicação. Não é necessário criar seis microserviços no início.
 
#### 8.5.1 Seleção de modelo, esforço e fornecedor
 
A seleção deve considerar tarefa e resultado esperado, complexidade, criticidade, qualidade mínima, nível de raciocínio/esforço, contexto, tool calling, saída estruturada, latência, disponibilidade, limites, preço, orçamento, privacidade, licenças e fallback.
 
O sistema deve manter um **catálogo de capacidades** e uma **política de roteamento** versionada. Pode propor um modelo económico para classificação simples e um modelo de maior capacidade para análise arquitetural, mas a decisão deve basear-se em avaliações reais, não em pressupostos fixos.
 
A “chave” de alternância deve ser entendida como configuração/política de seleção (por exemplo, perfil de tarefa, modelo preferido, esforço, limite de custo ou fornecedor permitido). Não deve ser confundida com API keys. As credenciais são segredos de autenticação, guardados num secret manager ou mecanismo seguro equivalente, com permissões e rotação próprias. O router seleciona uma identidade de fornecedor/modelo autorizada; não expõe nem troca credenciais em texto simples.
 
Quando a execução muda de Claude para um modelo OpenAI, ou para um modelo local, o sistema deve preparar um handoff explícito: objetivo, resumo, decisões, artefactos, estado, ferramentas permitidas e próximos passos. Não assumir que o histórico interno de uma ferramenta pode ser transferido integralmente ou que o modelo seguinte interpretará o contexto da mesma forma.
 
#### 8.5.2 Provider Token Intelligence: tokenização por fabricante
 
A contagem de tokens não pode basear-se numa regra universal de palavras por token. Tokenizadores diferentes podem dividir o mesmo texto de formas diferentes; o mesmo fornecedor pode ter diferenças entre famílias de modelos, versões ou modalidades.
 
O módulo deve manter um **Provider/Model Tokenization Registry**, com fornecedor, identificador e versão exata do modelo, tokenizador associado, biblioteca/versão oficial utilizada, API oficial de contagem quando existir, modalidades suportadas, categorias de tokens faturáveis, comportamento de cache, limites de contexto/saída e fonte/data/confiança da informação.
 
Ordem de preferência para estimar tokens:
1. API oficial de contagem do fornecedor/modelo, quando disponível e aplicável;
2. tokenizador oficial ou biblioteca publicada pelo fornecedor, com versão compatível;
3. estimador calibrado com dados reais daquele modelo;
4. estimativa conservadora baseada em amostras, explicitamente marcada como aproximada.
A contagem local é uma **previsão**, não prova de faturação. A utilização reportada pela resposta/API ou pelo mecanismo oficial de usage do fornecedor é a referência de reconciliação, quando disponível. Em ferramentas de programação e subscrições (por exemplo, utilização através de aplicações/CLI), os dados de consumo e faturação podem não ser equivalentes à tarifação direta por token de uma API. O sistema deve distinguir os regimes de cobrança e não inventar uma conversão para euros quando o fornecedor não expõe dados suficientes.
 
Categorias a contabilizar, conforme o fornecedor:
- tokens de entrada;
- tokens de saída;
- tokens de entrada em cache/reutilizados;
- tokens de escrita/criação de cache;
- tokens de raciocínio ou categorias especiais, se faturadas separadamente e reportadas;
- tokens/unidades de imagem, áudio, vídeo ou outras modalidades;
- chamadas a ferramentas, pesquisa, execução ou outros serviços cobrados à parte.
Não assumir que todos os fornecedores expõem todas estas categorias ou utilizam a mesma nomenclatura. Conservar o campo original reportado pelo fornecedor e mapear apenas o que for semanticamente equivalente para o esquema interno.
 
#### 8.5.3 Pricing Registry e previsão de custo antes da execução
 
Criar um **Pricing Registry** versionado e temporal, associado ao identificador exato do modelo e regime de utilização. Deve guardar preço por unidade e categoria, moeda, entrada/saída/cache/modalidades, escalões e descontos, custos de ferramentas, região/plano, validade, URL oficial, data da verificação e regime de cobrança (API, subscrição, licença, infraestrutura própria ou outro).
 
**Não codificar preços como constantes permanentes no código.** Devem ser dados versionados e atualizáveis, com origem e data. Quando não for possível confirmar um preço, marcar a previsão como incompleta e solicitar validação; nunca inventar valores.
 
Antes de executar uma tarefa, o **Cost Forecasting Engine** deve:
1. classificar a tarefa e complexidade;
2. estimar o tamanho do contexto e tokenização por cada modelo candidato;
3. estimar tokens de entrada e saída;
4. estimar cache hit/miss ou criação de cache, se aplicável;
5. estimar número de chamadas, iterações, retries e ferramentas;
6. contabilizar modalidades e custos não-token;
7. aplicar preços válidos na data da previsão;
8. calcular cenários otimista, provável e conservador;
9. apresentar intervalo, pressupostos e confiança;
10. comparar alternativas de modelo/esforço e custo estimado por resultado concluído.
A previsão deve ser feita para a **tarefa completa**, não apenas para a primeira chamada. Uma atividade com várias rondas, pesquisa, ferramentas, revisão e correção pode custar várias vezes mais do que a chamada inicial.
 
Exemplo meramente ilustrativo (valores fictícios, não são preços atuais):
 
| Alternativa | Custo estimado | Qualidade esperada | Decisão possível |
|---|---:|---|---|
| Modelo económico | €0,08 | Adequada para tarefa simples | Escolher se cumpre o limiar |
| Modelo intermédio | €0,22 | Elevada | Escolha equilibrada |
| Modelo avançado | €0,65 | Muito elevada | Reservar para maior criticidade |
 
A previsão deve mostrar o que está incluído e o que ficou de fora. Para modelos locais, o custo não é zero: considerar amortização/uso de GPU, eletricidade, infraestrutura, operação, manutenção e tempo de engenharia, quando relevante.
 
#### 8.5.4 Budget Intelligence: orçamento e limites de execução
 
Permitir limites por cliente, produto/projeto, ambiente, utilizador/equipa, fornecedor/modelo, período (por execução, diário, semanal, mensal) e tipo de tarefa.
 
##### Orçamento hierárquico e distribuição multiempresa
 
O orçamento não pode ser tratado como um único saldo global nem dividido apenas por tarefa. O Core deve gerir uma **hierarquia de orçamento e consumo**, com isolamento entre empresas e distribuição controlada pelos níveis organizacionais. A unidade de topo é o plano/contrato da empresa; abaixo dele podem existir carteiras por período, projetos, equipas (numa fase posterior ou quando aplicável), módulos/áreas de trabalho e atividades/tickets. A profundidade deve ser configurável: não obrigar todos os clientes a utilizar todos os níveis.
 
Modelo conceptual inicial:
 
```text
Empresa / Tenant
└── Plano contratado + limites e direitos de utilização
    └── Carteira de orçamento do período (dia / semana / mês)
        ├── Projeto A
        │   ├── Envelope do projeto / reserva
        │   ├── Equipa(s), se configuradas
        │   ├── Módulo ou categoria de trabalho (ex.: Git, Logs)
        │   └── Atividades / tickets / execuções
        └── Projeto B
            ├── Envelope do projeto / reserva
            ├── Módulo ou categoria de trabalho
            └── Atividades / tickets / execuções
```
 
**Exemplo ilustrativo — duas empresas com o mesmo limite, mas estruturas diferentes:**
 
- Empresa A tem um limite mensal de €1.000 e dois projetos. O Projeto A trabalha nos módulos Git e Logs, com um conjunto de demandas cuja previsão total é, por exemplo, €350. O Projeto B tem dez tickets de desenvolvimento analisados, com previsão total de, por exemplo, €420. O Core deve conseguir mostrar a distribuição por projeto, módulo e ticket, o valor reservado, o consumo real e o saldo ainda não atribuído.
- Empresa B também tem €1.000 mensais, mas quatro projetos. O sistema não deve assumir automaticamente que cada projeto recebe €250. A distribuição pode ser igualitária, ponderada por prioridade, volume de trabalho, contrato, criticidade, histórico ou decisão explícita do administrador.
Os valores acima são apenas exemplos matemáticos, não preços nem uma regra de distribuição. O princípio é que **o número de projetos não determina sozinho a quota**. O cliente pode escolher uma distribuição inicial manual ou uma política assistida, e o Core pode sugerir uma alocação baseada em procura prevista, prioridade e histórico, sempre apresentando os pressupostos e permitindo aprovação.
 
**Regras essenciais de distribuição:**
 
1. **Plano contratado como limite superior:** o sistema deve conhecer os direitos efetivamente contratados pela empresa — por exemplo, créditos monetários, unidades de utilização, limites de chamadas, capacidade incluída, ou uma combinação — e o período de validade. Não presumir que todos os planos são convertíveis num saldo monetário único.
2. **Carteira por período:** gerir limites diário, semanal e mensal, com timezone, início/fim do ciclo, regras de renovação e eventual rollover explicitamente configurados. Os limites de períodos diferentes devem coexistir sem serem somados como se fossem dinheiro adicional.
3. **Alocação para projetos:** distribuir uma parte do orçamento global para cada projeto. A soma das alocações e reservas não pode ultrapassar o limite disponível, salvo se existir uma política explícita de sobrealocação aprovada.
4. **Subalocação opcional:** um projeto pode repartir a sua quota entre equipas, módulos, categorias ou centros de custo. O suporte a múltiplas equipas por projeto deve fazer parte do modelo, mesmo que a primeira versão operacional comece apenas com projeto e atividade.
5. **Atividade/ticket:** cada execução deve estar associada à empresa, projeto e, quando existam, equipa, módulo, ticket, utilizador iniciador e centro de custo. A atribuição não pode depender apenas de texto livre ou ser inferida de forma silenciosa.
6. **Pool não atribuído:** manter uma parcela disponível e não distribuída para novas demandas, imprevistos ou projetos prioritários. Não forçar a atribuição de 100% do orçamento no início do ciclo.
7. **Reafetação controlada:** permitir mover orçamento entre projetos durante o período, com permissões, limites, motivo, registo de antes/depois e auditoria. A reafetação não pode apagar o histórico do consumo já ocorrido.
8. **Reserva antes da execução:** quando uma tarefa for aprovada, reservar o valor máximo ou uma reserva calculada com margem de segurança. Ao terminar, substituir a reserva pelo custo apurado e libertar a diferença. Evitar que várias execuções concorrentes gastem simultaneamente o mesmo saldo.
9. **Controlo em cascata:** uma tarefa só pode avançar se respeitar simultaneamente o limite da empresa, o limite do período, o envelope do projeto e os limites inferiores configurados. Uma quota de projeto não deve permitir ultrapassar o limite global da empresa.
10. **Exceções e prioridades:** permitir políticas de emergência, tarefas críticas e aprovação de excedentes, mas com autorização explícita, limite máximo e trilho de auditoria.
**Como decidir a quota de cada projeto:** disponibilizar políticas configuráveis, sem fixar uma única fórmula: quota manual; divisão igual; ponderação por prioridade/criticidade; previsão de procura; histórico de consumo ajustado; orçamento por centro de custo; ou combinação destas. O Planner deve apresentar a proposta, a justificação, o nível de confiança e o impacto nos restantes projetos. A decisão final e os limites de autonomia pertencem ao cliente/administrador.
 
**Consumo do plano contratado:** é necessário distinguir o orçamento comercial que o cliente paga à nossa plataforma dos custos de utilização que a nossa plataforma suporta junto dos fornecedores de IA. São duas dimensões relacionadas, mas não equivalentes. O contrato do nosso produto pode incluir créditos, quotas ou limites próprios; por baixo, cada execução pode consumir tokens, chamadas ou custos de vários fornecedores. O Core deve manter um ledger interno normalizado e fazer a reconciliação com os dados reais disponíveis, sem prometer equivalência exata quando o fornecedor não disponibiliza telemetria ou quando o plano é uma subscrição fixa.
 
Quando o cliente utiliza uma subscrição de consumidor, CLI ou IDE, e não uma API com faturação por utilização, o sistema não deve inventar um custo unitário nem assumir que consegue observar todo o consumo. Deve identificar a modalidade de acesso, as métricas efetivamente disponíveis e a limitação de medição. Nesses casos, pode apresentar consumo estimado/unidades internas e confiança reduzida, separado de custo faturado confirmado.
 
**Exemplo de visualização que o Core deve permitir:**
 
| Nível | Limite/alocação | Previsto/comprometido | Real | Disponível |
|---|---:|---:|---:|---:|
| Empresa A — mês | €1.000 | €770 | A apurar | €230 nominais antes da reconciliação |
| Projeto A — mês | €400 | €350 | A apurar | €50 nominais |
| Projeto B — mês | €450 | €420 | A apurar | €30 nominais |
| Pool não atribuído | €150 | €0 | €0 | €150 |
 
Valores fictícios. A tabela ilustra uma alocação possível, não uma recomendação universal. O dashboard deve permitir descer da empresa até ao projeto, equipa, módulo e ticket, mantendo a mesma definição de período e distinguindo valor alocado, reservado, estimado, reportado e faturado.
 
**Períodos e prevenção de dupla contagem:** o mesmo consumo real pertence a um único evento de utilização e deve ser agregado em vistas diária, semanal e mensal. Não se deve somar o limite diário ao semanal e ao mensal. Cada política define se o limite é independente, cumulativo ou uma janela de controlo; por exemplo, um limite diário de €40 e mensal de €1.000 são dois tetos simultâneos, não €1.040 de orçamento. Definir claramente semanas (por exemplo, segunda-feira a domingo), timezone, fecho de período, renovação e tratamento de execuções que atravessam a fronteira entre períodos.
 
**Precisão e confiança:** a previsão deve ser calibrada ao nível da tarefa e agregada por projeto e empresa. Para evitar desvios grandes, o sistema deve usar intervalos, margens de segurança, reservas, limites de concorrência e alertas de desvio. Deve comparar previsões com usage reportado e, quando existir, faturação do fornecedor. Apresentar separadamente: custo previsto, custo reservado/comprometido, consumo medido, custo calculado, custo faturado e custo ainda não reconciliado. Não apresentar uma estimativa como valor exato.
 
Acompanhar orçamento aprovado, consumo previsto/comprometido, consumo real reportado, saldo disponível, despesas pendentes, limites rígidos e alertas, e reservas para tarefas prioritárias.
 
Distinguir **orçamento**, **previsão**, **custo comprometido** e **custo real**. Um pedido pode ter estimativa de €2, mas o custo real só será conhecido após execução e reconciliação.
 
Políticas possíveis:
- permitir automaticamente abaixo de limite baixo;
- pedir aprovação humana acima de determinado valor;
- bloquear ou adiar quando o limite rígido for atingido;
- reservar orçamento para tarefas críticas;
- escolher alternativa mais económica se cumprir requisitos mínimos;
- interromper ou pedir autorização quando o consumo real divergir significativamente da previsão.
Uma recomendação de custo nunca deve reduzir silenciosamente requisitos de segurança, privacidade ou qualidade mínima.
 
#### 8.5.5 Activity Planner: recomendar o que executar com o orçamento disponível
 
O **Budget & Activity Planner** transforma o orçamento e o backlog numa proposta de execução. Não é apenas um mecanismo que escolhe o modelo mais barato.
 
Para cada atividade/ticket, estimar objetivo e critérios de aceitação, prioridade/urgência, valor de negócio, dependências, complexidade/incerteza, tempo humano e de máquina, custo de IA (intervalo), risco de falha/retrabalho, revisão humana, possibilidade de dividir a tarefa e modelo/esforço candidato.
 
Pode recomendar:
- **Executar agora:** alto valor/prioridade e custo compatível;
- **Executar em etapas:** dividir tarefa grande em entregas menores e verificáveis;
- **Usar alternativa económica:** quando cumpre a qualidade mínima;
- **Escalar o modelo:** quando a primeira abordagem não atinge critérios;
- **Pedir aprovação:** quando ultrapassa limite ou a incerteza é elevada;
- **Adiar:** quando há tarefas mais valiosas ou dependências por resolver;
- **Não executar automaticamente:** quando risco, ambiguidade ou impacto exigem decisão humana.
A recomendação deve considerar valor, prioridade, risco, dependências, tempo e orçamento, e explicar o motivo. Não deve ordenar tarefas apenas por custo baixo. Pode integrar Jira ou outros gestores de trabalho, mas o primeiro protótipo pode funcionar com um backlog simples.
 
Exemplo conceptual: orçamento mensal aprovado €100; custo real acumulado €62; saldo nominal €38. Tarefa A (correções simples) €4,50; B (testes unitários) €8; C (refatoração arquitetural) €24; D (análise extensa) €35. O planner pode propor A e B, reservar parte do saldo para C ou pedir autorização para D. Valores fictícios: a decisão real inclui prioridade, dependências, intervalo de custo e margem de segurança.
 
#### 8.5.6 Reconciliação e aprendizagem com consumo real
 
Após cada execução, recolher, quando disponível: modelo/versão utilizados, tokens/unidades por categoria, chamadas e ferramentas, retries/falhas, duração/latência, custo reportado ou calculado, resultado/avaliação de qualidade, intervenção humana e tarefa/projeto associados.
 
Comparar custo previsto vs. real, tokens estimados vs. reportados, duração estimada vs. real, qualidade esperada vs. observada e número previsto vs. real de iterações.
 
Calcular desvios e atualizar estimadores por modelo, tipo de tarefa, idioma, tamanho de contexto e perfil de execução. Preservar histórico suficiente para auditoria, aplicando minimização, retenção e controlos de privacidade.
 
O sistema aprende com execuções anteriores, mas o passado não garante custo futuro. Mudanças de modelo, prompt, contexto, preço, cache, runtime ou comportamento do utilizador podem invalidar estimativas anteriores.
 
#### 8.5.7 Transparência da recomendação
 
Cada proposta deve apresentar tarefa/objetivo, modelo/fornecedor/esforço recomendado, alternativas, custo previsto em intervalo, pressupostos de tokenização e preços, confiança, orçamento disponível e saldo projetado, qualidade mínima esperada, riscos/limitações e necessidade de aprovação humana.
 
A decisão automática deve ter limites configuráveis e auditáveis. O cliente/utilizador pode definir políticas, mas mecanismos internos de otimização e benchmarks próprios podem permanecer no core proprietário.
 
#### 8.5.8 Separar a chave de seleção das credenciais
 
A “chave” ou seletor para alternar modelos é uma configuração de política: por exemplo `economy`, `balanced`, `quality`, `private`, `provider=...`, `model=...` ou `max_cost=...`. São exemplos conceptuais, não uma API aprovada.
 
API keys e credenciais são segredos separados. Devem ser guardados num secret manager, com acesso mínimo, rotação e auditoria. Nunca devem ser armazenados em ficheiros de conhecimento, prompts, logs ou no repositório partilhado.
 
#### 8.5.9 Limites e perguntas que o futuro agente deve investigar
 
A implementação concreta ainda não está decidida. O agente responsável deve pesquisar e comparar:
- tokenizadores e bibliotecas oficiais por fornecedor/modelo;
- APIs oficiais de contagem de tokens;
- formatos de usage/telemetria e exportação de custos;
- diferenças entre APIs pagas, subscrições e ferramentas/CLI;
- APIs de preços, páginas oficiais e atualização do Pricing Registry;
- mecanismos de cache e respetiva faturação;
- limites de contexto, esforço/raciocínio e tool calling;
- frameworks de model routing e gateways multi-provider;
- integração com Claude Code, APIs OpenAI/Codex e modelos open-weight;
- conectores para Jira/GitLab e ingestão de backlog;
- soluções de orçamento, quotas, approval gates e observabilidade;
- ferramentas de evals e benchmarks de custo/qualidade;
- mecanismos de handoff/context portability entre agentes.
A pesquisa deve distinguir documentação oficial, código open-source, produto comercial, artigo científico e hipótese. Testar as ferramentas em pequenos protótipos antes de recomendar dependências.
 
**Não assumir que uma API de um fornecedor está disponível para uma subscrição de consumidor, nem que dados de utilização de uma CLI podem ser obtidos ou convertidos em custo por token.** Confirmar contratos e capacidades de cada integração.
 
---
 
## 9. Arquitetura de referência para os produtos
 
Uma arquitetura de referência possível:
 
```text
┌─────────────────────────────────────────────────────────────┐
│                    PRODUTO / APLICAÇÃO                       │
│             UI, API, regras e domínio do negócio             │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                AI APPLICATION / AI HARNESS                   │
│ Contexto • políticas • planeamento • ferramentas • estado    │
│ permissões • validação • retries • tracing • observabilidade │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    AI CONTROL PLANE                          │
│ Task • Token • Cost Forecast • Budget & Allocation Ledger    │
│ Tenant • Plan • Period • Project • Team • Module • Ticket    │
│ Model Router • Effort • Activity Planner • Handoff           │
│ Reservations • Reconciliation • Approval Policies            │
└──────────────┬──────────────────────┬────────────────────────┘
               │                      │
       ┌───────▼────────┐     ┌───────▼──────────┐
       │ Provider APIs  │     │ Local / Private  │
       │ Paid / Free    │     │ Open-weight      │
       └───────┬────────┘     └───────┬──────────┘
               └────────────┬─────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│             ADAPTERS / NORMALIZED INTERFACES                 │
│ Chat • embeddings • tools • structured output • multimodal   │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                  KNOWLEDGE & MEMORY LAYER                    │
│ RAG • vector search • DB • cache • history • files           │
│ Source of truth • access control • provenance                │
└─────────────────────────────────────────────────────────────┘
 
Transversal:
Evals • segurança • custos • métricas • logs • auditoria • CI/CD
```
 
Esta arquitetura é conceptual. Não implica que todos os produtos necessitem de todos os componentes desde o primeiro dia. A implementação deve começar simples e evoluir quando os requisitos justificarem.
 
### 9.1 Componentes que devem ser separados
 
**Domínio do negócio**
- regras de negócio;
- entidades;
- permissões;
- fluxos;
- validações;
- dados oficiais.
**Camada de IA**
- seleção de modelo;
- construção de contexto;
- prompts versionados;
- tool calling;
- memória de agente;
- RAG;
- validação de saídas.
**Integração**
- adaptadores de fornecedores;
- contratos internos;
- retries e timeouts;
- rate limits;
- fallback;
- tratamento de erros.
**Operação**
- logs e tracing;
- métricas;
- custo por tarefa;
- latência;
- qualidade;
- avaliação;
- alertas.
### 9.2 Contratos e adaptadores
 
Criar interfaces internas para as capacidades que realmente são comuns, por exemplo:
- `ChatModel`
- `EmbeddingModel`
- `Reranker`
- `StructuredOutputModel`
- `ToolExecution`
- `DocumentRetriever`
- `MemoryStore`
Os nomes são ilustrativos; os contratos concretos devem ser definidos conforme as necessidades dos projetos.
 
Um adaptador converte o contrato interno para a API ou runtime do fornecedor. Deve preservar metadados relevantes e expor diferenças de capacidade, em vez de fingir que todos os modelos fazem exatamente o mesmo.
 
### 9.3 Perfil de capacidades
 
Cada modelo/serviço deve ter um registo versionado com:
- fornecedor e identificador exato do modelo;
- versão/checkpoint;
- modalidade (API, local, privado);
- contexto máximo anunciado e contexto testado;
- suporte a ferramentas;
- suporte a JSON/saída estruturada;
- multimodalidade;
- embeddings/reranking, se aplicável;
- idiomas avaliados;
- limites e rate limits;
- licença e restrições comerciais;
- requisitos de hardware;
- preço e data da última verificação;
- métricas internas de qualidade, latência e custo;
- limitações conhecidas.
Preços e limites mudam. Não codificar preços como verdades permanentes; guardar fonte e data de verificação.
 
---
 
## 10. Repositório GitLab comum a todos os projetos
 
### 10.1 Problema que queremos resolver
 
Ao criar projetos independentes com Claude Code, existe o risco de:
- copiar os mesmos documentos para várias pastas;
- esquecer de atualizar uma cópia;
- cada projeto passar a utilizar uma versão diferente dos padrões;
- perder decisões arquiteturais comuns;
- misturar conhecimento geral com regras específicas de um produto;
- expor informação interna num repositório que deveria ser partilhado com um cliente.
Um repositório central reduz estes riscos, desde que tenha governação, versões e regras de consumo.
 
### 10.2 Nome e missão
 
Nome técnico recomendado: `shared-ai-core` ou `ai-platform-core`.
 
Missão:
 
> Manter conhecimento, padrões, instruções, skills e componentes reutilizáveis que possam ser consumidos por múltiplos projetos, com versões, proveniência, revisão e controlo de acesso.
 
O repositório não deve tornar-se um depósito indiscriminado de ficheiros. Cada conteúdo deve ter finalidade, proprietário, estado e versão.
 
### 10.3 Estrutura sugerida
 
```text
shared-ai-core/
│
├── README.md
├── VERSION
├── CHANGELOG.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE-AND-USAGE.md
│
├── principles/
│   ├── architecture-principles.md
│   ├── plug-and-play-strategy.md
│   ├── vendor-neutrality.md
│   ├── privacy-and-data-ownership.md
│   └── decision-records.md
│
├── knowledge/
│   ├── ai-fundamentals/
│   ├── model-architectures/
│   │   ├── transformers.md
│   │   ├── recurrent-models.md
│   │   ├── state-space-models.md
│   │   ├── hybrid-architectures.md
│   │   └── memory-caching-march.md
│   ├── context-and-memory/
│   ├── rag-and-retrieval/
│   ├── agents-and-harness/
│   ├── evaluation-and-observability/
│   └── security-and-governance/
│
├── skills/
│   ├── research-paper-analysis/
│   ├── architecture-review/
│   ├── model-selection/
│   ├── tokenization-and-usage-analysis/
│   ├── cost-forecasting/
│   ├── budget-and-activity-planning/
│   ├── provider-integration/
│   ├── rag-evaluation/
│   ├── security-review/
│   └── cost-optimization/
│
├── standards/
│   ├── coding-standards/
│   ├── api-contracts/
│   ├── logging-and-tracing/
│   ├── testing-and-evals/
│   ├── security/
│   └── documentation/
│
├── reference-architectures/
│   ├── model-router/
│   ├── rag-service/
│   ├── agent-harness/
│   └── provider-adapters/
│
├── templates/
│   ├── architecture-decision-record.md
│   ├── model-evaluation.md
│   ├── threat-model.md
│   └── project-ai-profile.yaml
│
├── registries/
│   ├── model-capabilities/
│   ├── provider-capabilities/
│   ├── tokenizers/
│   ├── pricing/
│   ├── budget-policies/
│   ├── allocation-policies/
│   └── plan-entitlements/
│
├── schemas/
│   ├── tenant-and-project/
│   ├── budget-ledger/
│   ├── usage-events/
│   └── cost-reconciliation/
│
└── evaluated-components/
│
├── examples/
│   ├── java-spring/
│   ├── python/
│   └── integration-patterns/
│
└── projects/
    └── project-integration-guide.md
```
 
### 10.4 Separar conhecimento, skills e código
 
Não tratar tudo como “skill”.
 
- **Knowledge:** explicações, estudos, decisões científicas e documentação de referência.
- **Skills:** instruções operacionais reutilizáveis para um agente executar uma tarefa de forma consistente.
- **Standards:** regras e convenções que os projetos devem cumprir.
- **Reference architectures:** exemplos e desenhos de implementação.
- **Libraries/packages:** código reutilizável, quando existir maturidade para o manter como software.
- **Templates:** modelos para decisões, avaliações, requisitos e documentação.
- **Registries:** inventários estruturados de modelos, fornecedores, capacidades e avaliações.
Uma skill deve indicar quando usar, entradas esperadas, procedimento, critérios de qualidade, limites, saídas e exemplos. Não deve ser apenas um texto longo com conhecimento genérico.
 
### 10.5 Como os projetos consomem o repositório
 
Há várias estratégias. Não é necessário escolher apenas uma para tudo.
 
| Estratégia | Vantagens | Riscos/limitações | Uso recomendado |
|---|---|---|---|
| Git submodule | Mantém referência a um commit exato; evita cópia manual | Operação pode ser menos intuitiva; exige inicialização/atualização | Consumir uma versão exata do repositório comum |
| Git subtree | Conteúdo fica dentro do projeto e pode ser atualizado a partir da origem | Histórico e sincronização podem ficar mais complexos | Quando o projeto precisa de conteúdo incorporado |
| Package/artefacto versionado | Ideal para código e contratos reutilizáveis | Exige pipeline, publicação e compatibilidade | Bibliotecas Java/Python e componentes executáveis |
| Clone/checkout em workspace | Simples para leitura por Claude Code durante desenvolvimento | Não fixa automaticamente a versão usada por cada projeto | Exploração e desenvolvimento local |
| Link para documentação remota | Evita duplicação | Disponibilidade e contexto podem variar; não garante que o agente leu o conteúdo | Referência complementar, não dependência crítica |
 
**Recomendação inicial:**
1. Criar o repositório central privado.
2. Começar com Markdown, skills e templates.
3. Fixar em cada projeto a versão/commit que utiliza.
4. Usar submodule ou subtree depois de testar o fluxo com dois projetos reais.
5. Publicar código como package apenas quando houver interfaces estáveis e testes.
### 10.6 Integração com Claude Code
 
Cada repositório de produto deve ter um ficheiro de entrada local, por exemplo `CLAUDE.md`, que:
- explique o objetivo e arquitetura daquele produto;
- indique a versão do `shared-ai-core` utilizada;
- aponte para os diretórios de conhecimento e skills aplicáveis;
- indique regras específicas do projeto;
- esclareça o que é obrigatório, recomendado ou apenas referência;
- proíba assumir que uma referência foi lida quando não foi carregada.
Exemplo conceptual:
 
```markdown
# Instruções do projeto
 
## Conhecimento transversal
Este projeto utiliza o repositório privado `shared-ai-core`,
versão/tag `v1.0.0` (ou commit fixado).
 
Consultar:
- principles/
- standards/
- skills/model-selection/
- skills/architecture-review/
 
## Regras locais
As regras específicas deste produto prevalecem quando forem
explicitamente aprovadas e não violarem os requisitos de segurança.
 
## Atualizações
Não atualizar automaticamente a versão do conhecimento comum.
Propor atualização, avaliar impacto e registar a decisão.
```
 
O mecanismo exato de carregamento de skills e instruções deve ser validado com a versão instalada do Claude Code. Não assumir que um link remoto é automaticamente lido ou que todos os caminhos são carregados por omissão.
 
### 10.7 Versionamento e atualização
 
Adotar versionamento semântico quando fizer sentido:
- `MAJOR`: alteração incompatível de contrato, padrão obrigatório ou estrutura.
- `MINOR`: nova capacidade ou orientação compatível.
- `PATCH`: correção, clarificação ou atualização editorial sem alteração de contrato.
Cada projeto deve registar:
- versão/tag ou commit do `shared-ai-core`;
- data de adoção;
- alterações relevantes;
- exceções locais;
- responsável pela aprovação.
Não atualizar todos os projetos automaticamente. Uma atualização central deve gerar uma proposta de adoção, testes e decisão por projeto.
 
### 10.8 Governação e segurança
 
O repositório central contém conhecimento estratégico interno. Recomenda-se:
- visibilidade privada no GitLab;
- acesso por grupos e função;
- branch protegida;
- Merge Requests obrigatórios para alterações;
- revisão de alterações a padrões e arquitetura;
- histórico de decisões;
- secret scanning;
- dependabot/renovação de dependências quando aplicável;
- classificação de informação;
- política clara sobre o que pode ser copiado para repositórios de clientes.
**Nunca colocar no repositório:**
- passwords, tokens, API keys ou certificados privados;
- dados reais de clientes sem autorização;
- dumps de bases de dados;
- prompts com informação confidencial;
- credenciais de ambientes;
- contratos ou documentação de cliente sem autorização;
- detalhes que permitam comprometer infraestrutura.
### 10.9 Separação do conhecimento proprietário
 
A estratégia de modularidade e seleção de modelos deve ser tratada como conhecimento interno. Separar:
1. **Core interno:** critérios de seleção, benchmarks próprios, estratégia de negociação, custos internos, arquitetura de roteamento e know-how.
2. **Contratos de integração:** interfaces e requisitos necessários para operar o produto.
3. **Documentação do cliente:** funcionalidades, garantias, limites, dados, segurança e operação que o cliente precisa de conhecer.
4. **Código do produto:** implementação e configuração que o contrato/licença permite entregar.
A transparência com o cliente sobre dados, segurança, custos contratados e limitações não deve ser confundida com revelar todos os mecanismos internos de vantagem competitiva.
 
---
 
## 11. Como medir se um componente é realmente melhor
 
Não selecionar modelos por anúncios, rankings isolados ou apenas pelo número máximo de tokens.
 
Criar um processo de avaliação repetível.
 
### 11.1 Métricas técnicas
 
- qualidade da resposta;
- precisão factual;
- recuperação correta de fontes;
- taxa de alucinação em cenários definidos;
- taxa de sucesso de tool calling;
- qualidade de saída estruturada;
- latência p50, p95 e p99;
- throughput;
- tokens de entrada/saída;
- memória de GPU/RAM;
- taxa de erro e indisponibilidade;
- recuperação após falhas;
- custo por tarefa concluída.
### 11.2 Métricas de negócio
 
- custo por utilizador ativo;
- custo por fluxo concluído;
- tempo poupado;
- taxa de resolução sem intervenção humana;
- taxa de escalonamento para operador;
- satisfação e qualidade percebida;
- custo de suporte e manutenção;
- margem por cliente;
- valor entregue face ao preço.
### 11.3 Testes de contexto longo e memória
 
Construir um conjunto de testes que inclua:
- factos colocados no início, meio e fim do contexto;
- perguntas que exigem combinar informações distantes;
- informação contraditória e atualizações;
- dados irrelevantes que devem ser ignorados;
- fontes com versões diferentes;
- instruções maliciosas dentro de documentos;
- consultas sem resposta disponível;
- necessidade de citar a origem;
- permissões distintas por utilizador.
 
Avaliar:
- se recupera o facto correto;
- se identifica a fonte;
- se distingue informação antiga de atual;
- se admite quando não sabe;
- se não revela informação sem permissão;
- se o custo e latência são aceitáveis.
### 11.4 Avaliação comparativa justa
 
Comparar modelos com:
- o mesmo conjunto de tarefas;
- as mesmas condições de contexto;
- prompts e ferramentas equivalentes;
- parâmetros documentados;
- hardware e runtime identificados;
- várias execuções quando existir variabilidade;
- métricas definidas previamente.
Registar versão exata do modelo, data, fornecedor, região, hardware, runtime, quantização, configuração e custo observado.
 
**Regra:** não declarar que um modelo é melhor se apenas foi testado numa tarefa ou benchmark.
 
---
 
## 12. Estratégia comercial e relação com o cliente
 
A escolha tecnológica deve ser interna e orientada a valor. A oferta ao cliente pode apresentar níveis de serviço adequados ao orçamento e aos requisitos.
 
Exemplo conceptual:
 
| Perfil de solução | Composição possível | Adequado quando |
|---|---|---|
| Essencial | Modelo económico/API ou open-weight validado | Orçamento limitado e tarefas simples |
| Equilibrado | Combinação de modelos por tipo de tarefa | Relação qualidade/custo é prioritária |
| Avançado | Modelos de maior capacidade e ferramentas especializadas | Tarefas complexas e exigência elevada |
| Privado/Controlado | Modelo local ou infraestrutura dedicada | Requisitos fortes de privacidade, controlo ou residência de dados |
 
Estes perfis não devem ser apresentados como garantia automática de qualidade. Cada um exige avaliação, limites e condições explícitas.
 
A nossa vantagem está em conseguir escolher a combinação apropriada e alterá-la ao longo do tempo, sem comprometer a continuidade do produto.
 
### Regras de decisão
 
- Não vender “modelo mais caro” como sinónimo de melhor solução.
- Não escolher local apenas por ser open-weight: hardware, operação e manutenção também custam.
- Não escolher API apenas por facilitar o arranque: dependência, preço, limites e tratamento de dados devem ser avaliados.
- Não prometer independência total de fornecedor: explicar as dependências inevitáveis e como são mitigadas.
- Rever periodicamente preço, qualidade, licenças e disponibilidade.
- Definir um caminho de migração e fallback para componentes críticos.
---
 
## 13. Riscos e medidas de mitigação
 
| Risco | Consequência | Mitigação |
|---|---|---|
| Abstração demasiado genérica | Perda de funcionalidades específicas | Núcleo comum + extensões por fornecedor |
| Dependência de um fornecedor | Aumento de preço ou indisponibilidade | Adaptadores, fallback e testes de migração |
| Modelo barato com qualidade insuficiente | Erros e retrabalho | Evals por tarefa e thresholds mínimos |
| Modelo local subdimensionado | Latência e falhas | Teste de carga e cálculo de TCO |
| Contexto longo sem recuperação eficaz | Respostas erradas apesar de muito contexto | RAG, reranking, citações e testes de recuperação |
| Memória do modelo confundida com memória do negócio | Perda ou corrupção de dados importantes | Source of truth em bases de dados controladas |
| Skills/documentos desatualizados | Agentes seguem regras antigas | Versionamento, changelog e revisão |
| Repositório central demasiado amplo | Dificuldade de navegação e risco de exposição | Separação por domínio e permissões |
| Segredos em Git | Comprometimento de sistemas | Secret scanning, vault e rotação |
| Benchmark pouco representativo | Decisão tecnológica errada | Dataset e testes próprios do produto |
| Mudança de licença | Risco jurídico/comercial | Registo e revisão da licença antes de cada adoção |
 
---
 
## 14. Plano de implementação incremental
 
### Fase 0 — Registar a decisão
 
- Aprovar o princípio de arquitetura modular.
- Definir o que é conhecimento comum e o que é específico de cada produto.
- Definir classificação de informação e regras de acesso.
- Não criar microserviços ou abstrações prematuramente.
### Fase 1 — Repositório central mínimo
 
Criar `shared-ai-core` privado com:
- README;
- princípios;
- arquitetura Plug and Play;
- conhecimento sobre modelos e memória;
- skills iniciais;
- templates de decisão e avaliação;
- changelog e versionamento.
### Fase 2 — Projeto piloto
 
Escolher dois projetos distintos e testar:
- consumo da mesma versão do repositório;
- leitura das instruções por Claude Code;
- reutilização de uma skill;
- atualização controlada;
- registo de exceções locais.
### Fase 3 — Contratos e adaptadores
 
Só depois de casos reais:
- definir interfaces comuns;
- implementar um primeiro adaptador;
- integrar dois fornecedores/modelos;
- criar testes de contrato;
- validar fallback e logging.
### Fase 4 — Model evaluation e routing
 
- criar dataset de tarefas representativas;
- testar modelos pagos e open-weight;
- medir qualidade, latência, custo e segurança;
- criar política de seleção;
- documentar limites e decisões.
### Fase 5 — Maturidade operacional
 
- monitorização e tracing;
- alertas de custo e falha;
- avaliação contínua;
- revisão de modelos e licenças;
- processo de atualização do repositório central;
- plano de migração por produto.
---
 
## 15. Integração no currículo “IA — Formação, Conhecimento e Evolução”
 
Este estudo deve ser incorporado nos três eixos já definidos, sem criar um eixo separado.
 
### Eixo A — Fundamentos e compreensão
 
Adicionar/expandir:
- arquiteturas de sequência: RNN, Transformer e State Space Models;
- atenção e KV cache;
- memória recorrente e estados internos;
- arquiteturas híbridas;
- eficiência de inferência e trade-offs de memória;
- interpretação de benchmarks de contexto longo.
### Eixo B — Engenharia e arquitetura
 
Adicionar/expandir:
- Memory Engineering;
- memória de agentes vs memória interna de modelos;
- gestão de contexto e recuperação histórica;
- model routing e abstração de fornecedores;
- adaptadores e contratos de capacidades;
- avaliação comparativa de modelos;
- custo total de operação (TCO);
- observabilidade de modelos e fornecedores;
- arquitetura modular Plug and Play.
### Eixo C — Aplicação e pensamento crítico
 
Aplicar:
- selecionar modelo para um caso de negócio;
- justificar escolha por custo, qualidade, segurança e manutenção;
- desenhar uma arquitetura substituível;
- testar fallback;
- comparar modelo API, open-weight e local;
- comunicar trade-offs sem prometer independência absoluta.
### Projeto integrador
 
O projeto integrador de pesquisa documental pode evoluir para testar, quando os pré-requisitos estiverem compreendidos:
1. um Transformer/API;
2. um modelo open-weight;
3. uma alternativa híbrida, quando o hardware e a licença permitirem;
4. RAG independente do fornecedor;
5. avaliação comum;
6. troca de modelo sem alterar a lógica de negócio.
Não é necessário implementar Memory Caching diretamente. O objetivo pedagógico inicial é compreender a diferença entre arquiteturas e avaliar a relevância empresarial.
 
---
 
## 16. Decisões propostas e estado
 
| ID | Decisão | Estado recomendado |
|---|---|---|
| ADR-001 | Arquitetura modular e substituível como princípio transversal | Proposta para aprovação |
| ADR-002 | Repositório privado central de conhecimento e skills | Proposta para implementação |
| ADR-003 | Separar conhecimento comum de regras específicas de cada produto | Princípio recomendado |
| ADR-004 | Versionar o conhecimento comum e fixar versão por projeto | Princípio recomendado |
| ADR-005 | Avaliar modelos por tarefa e custo total, não por ranking isolado | Princípio recomendado |
| ADR-006 | Manter dados de negócio independentes do fornecedor de IA | Princípio fundamental |
| ADR-007 | Preservar estratégia de seleção e otimização como know-how interno | Orientação estratégica interna |
| ADR-008 | Incluir arquiteturas de memória e modelos eficientes no currículo | Proposta de atualização curricular |
 
Estas decisões só devem ser marcadas como aprovadas depois de confirmação explícita e, quando aplicável, registo no repositório.
 
---
 
## 17. Glossário
 
- **Transformer:** arquitetura de rede neuronal baseada em mecanismos de atenção.
- **RNN:** rede neuronal recorrente que atualiza um estado à medida que processa uma sequência.
- **SSM:** State Space Model; família de modelos que representa sequências através de estados dinâmicos.
- **Mamba:** arquitetura baseada em SSM com mecanismos seletivos.
- **KV Cache:** armazenamento de chaves e valores de atenção usados na inferência de Transformers.
- **Memory Caching:** técnica do artigo estudado que guarda checkpoints de estados recorrentes.
- **MARCH:** arquitetura que encaminha consultas para âncoras de estados históricos com base no conteúdo.
- **MoE:** Mixture of Experts; arquitetura que ativa subconjuntos de especialistas por entrada.
- **RAG:** Retrieval-Augmented Generation; recuperação de informação externa para apoiar a geração.
- **Embedding:** representação vetorial de conteúdo.
- **Reranker:** componente que reordena resultados recuperados segundo a sua relevância.
- **AI Harness:** sistema envolvente que controla contexto, ferramentas, permissões, execução, validação e observabilidade de um modelo/agente.
- **Model Router:** componente que seleciona o modelo/fornecedor adequado a uma tarefa.
- **Adapter:** camada que traduz um contrato interno para a API/runtime específico.
- **Open-weight:** modelo cujos pesos são disponibilizados; não significa automaticamente licença irrestrita ou software totalmente open-source.
- **Evals:** avaliações repetíveis de qualidade, segurança e comportamento.
- **TCO:** Total Cost of Ownership; custo total de aquisição, operação, manutenção e evolução.
- **Vendor lock-in:** dependência de um fornecedor que torna a mudança difícil ou dispendiosa.
- **Plug and Play:** capacidade de substituir/adicionar componentes através de contratos e testes definidos, sem reconstruir todo o sistema.
---
 
 
## 18. Brief obrigatório para o próximo agente de arquitetura/implementação
 
Este documento é uma base estratégica e de requisitos, não uma decisão de implementação fechada. O próximo agente (Claude Code, ChatGPT Work ou outro) deve **validar, aprofundar e desafiar** estas propostas antes de implementar.
 
### Objetivo da análise seguinte
 
Produzir uma arquitetura técnica detalhada para o AI Control Plane e o repositório `shared-ai-core`, incluindo alternativas, decisões justificadas, riscos, ferramentas e plano incremental.
 
### Investigações obrigatórias
 
1. **Tokenização e usage por fornecedor:** verificar documentação oficial e ferramentas de contagem para Anthropic/Claude, OpenAI, Google e fornecedores relevantes; confirmar se a contagem local corresponde ao modelo/versão exatos; mapear categorias de usage e faturação; distinguir API, subscrição, CLI e utilização em IDEs/aplicações; documentar o que pode ser medido e o que só pode ser estimado.
2. **Preços e custos:** identificar fontes oficiais e mecanismos de atualização; modelar preços por input/output/cache/modalidade/ferramenta; analisar regiões, tiers, descontos e limites; desenhar Pricing Registry com validade temporal; calcular custo total de modelos locais e APIs.
3. **Previsão de tarefas:** definir esquema de tarefa e complexidade; estimar contexto, tokens, chamadas, ferramentas, retries e saída; apresentar cenários otimista/provável/conservador; definir intervalo e confiança; medir erro e calibração.
4. **Model Router e esforço:** comparar gateways/frameworks e integração direta; verificar suporte a Claude, OpenAI/Codex, Google e modelos open-weight; definir perfis de qualidade/custo/latência/privacidade; distinguir seleção de modelo, esforço e credenciais; desenhar fallback, retries e handoff.
5. **Budget & Activity Planner:** definir orçamento por cliente/projeto/equipa/período; propor quotas, reservas, alertas, hard limits e approval gates; integrar backlog GitLab/Jira apenas após validar o modelo de dados; desenhar recomendação considerando prioridade, valor, dependências, risco, tempo e custo; garantir que a otimização financeira não viola segurança nem qualidade mínima.
6. **Orçamentação hierárquica e consumo de planos:** modelar isolamento multiempresa (tenant), plano/contrato e direitos de utilização, carteiras diária/semanal/mensal, alocação e reafetação por projeto, subalocação opcional por equipa/módulo/centro de custo e imputação por ticket/execução; definir políticas de distribuição manual, igualitária, ponderada e baseada em previsão; impedir dupla contagem entre períodos e sobreconsumo concorrente; definir reservas, pool não atribuído, rollover, fecho de período, timezone, limites em cascata e auditoria; separar custo comercial do plano do custo de fornecedores; validar o que é mensurável em API versus subscrição/CLI/IDE; desenhar ledger e reconciliação com incerteza explícita.
7. **Memória e continuidade:** rever a decisão de aproveitar `ai-memory`, confirmando estado, licença, manutenção e adequação; comparar memória de agente, handoff e armazenamento independente do fornecedor.
8. **Segurança e governação:** separar segredos, dados de cliente, usage e conhecimento proprietário; definir retenção, minimização, auditoria, permissões e isolamento multi-tenant; não colocar credenciais ou dados sensíveis no repositório comum.
### Entregáveis esperados
 
- diagrama de componentes e fluxos;
- comparação de pelo menos duas opções de implementação por capacidade central;
- matriz de fornecedores, tokenizadores, usage, preços e limitações, com fontes oficiais e data;
- modelo de dados para modelos, tokenizadores, preços, previsões, budgets e execuções, incluindo tenant, plano/entitlements, período, alocações hierárquicas, reservas, eventos de consumo, ledger e reconciliação;
- contratos/interfaces sugeridos;
- protótipo mínimo e testes;
- análise de custo/benefício e riscos;
- ADRs com decisões aprovadas, rejeitadas e pendentes;
- plano de evolução por fases.
### Regras para o próximo agente
 
- Não tratar ferramentas citadas neste documento como escolha definitiva.
- Não inventar preços, capacidades, APIs, métricas ou compatibilidade.
- Confirmar documentação atual e licenças antes de recomendar dependências.
- Separar factos verificados, hipóteses, inferências e propostas.
- Não implementar decisões comerciais ou financeiras sem aprovação.
- Não alterar a lógica de negócio apenas para acomodar uma abstração genérica.
- Preferir o menor protótipo que permita medir valor e risco.
- Registar fontes e data de cada informação volátil.
---
 
## 19. Fontes e ligações
 
### Artigos científicos
 
1. Behrouz et al. (2026). *Memory Caching: RNNs with Growing Memory*. ICML 2026, PMLR 306.
   - https://proceedings.mlr.press/v306/behrouz26a.html
   - https://proceedings.mlr.press/v306/behrouz26a/behrouz26a.pdf
   - https://arxiv.org/abs/2602.24281
2. Zhang et al. (2026). *MARCH: Scaling Recurrent Memory with Content-Routed State Anchors*. Preprint.
   - https://arxiv.org/abs/2608.12435
3. Lahoti et al. (2026). *Mamba-3: Improved Sequence Modeling using State Space Principles*. ICLR 2026.
   - https://proceedings.iclr.cc/paper_files/paper/2026/hash/8abd2043b71a074278d5f687947bff9c-Abstract-Conference.html
   - https://openreview.net/pdf/09c67b76b7d1ad254ea30eeba839b8592de9bd78.pdf
4. ATLAS — página oficial PMLR, referência a validar antes de citação bibliográfica final.
   - https://proceedings.mlr.press/v306/behrouz26b.html
### Documentação de empresas
 
5. IBM Granite 4.0.
   - https://www.ibm.com/granite/docs/models/granite4-0
6. NVIDIA Nemotron 3.
   - https://research.nvidia.com/labs/nemotron/Nemotron-3/
7. NVIDIA Nemotron 3 Ultra.
   - https://research.nvidia.com/labs/nemotron/Nemotron-3-Ultra/
8. NVIDIA Nemotron — modelos e recursos.
   - https://developer.nvidia.com/topics/ai/nemotron
### Nota de atualização
 
Modelos, preços, licenças, limites de contexto, disponibilidade e requisitos de hardware mudam. As referências deste documento representam o estado da pesquisa consultada em 24/11/2026. Antes de cada decisão de adoção, confirmar a documentação oficial, os pesos/checkpoints, a licença e os resultados no ambiente real.
 
---
 
## 20. Recomendação final
 
O verdadeiro ganho para os nossos negócios não será simplesmente utilizar o modelo mais inteligente, mais caro ou mais recente.
 
Será construir uma plataforma que consiga **escolher, combinar e substituir modelos conforme o problema**, mantendo o conhecimento, os dados de negócio, as regras e a lógica da aplicação sob o nosso controlo.
 
Isto cria uma vantagem competitiva maior do que ficar dependente de uma tecnologia específica da Google, NVIDIA, IBM ou de qualquer outro fornecedor.
 
A cultura que devemos adotar é:
 
1. **Modularidade:** cada componente tem uma responsabilidade e um contrato.
2. **Substituibilidade:** modelos, fornecedores e serviços podem ser trocados com impacto controlado.
3. **Avaliação:** nenhuma escolha é considerada melhor sem evidência relevante para o caso.
4. **Economia:** custo por tarefa concluída e valor entregue importam mais do que o preço unitário do token.
5. **Privacidade e controlo:** os dados e a memória de negócio permanecem sob governação da aplicação e do cliente.
6. **Evolução:** a arquitetura permite incorporar novas técnicas sem reescrever o produto.
7. **Conhecimento partilhado:** princípios, skills e padrões comuns vivem num repositório central versionado.
8. **Know-how protegido:** os critérios internos de otimização e a forma como combinamos componentes são ativos estratégicos; expomos ao cliente o que é necessário para confiança, segurança e operação, sem divulgar desnecessariamente a nossa vantagem competitiva.
**Recomendação de execução:** criar primeiro o repositório privado `shared-ai-core`, colocar nele este estudo e os princípios de arquitetura, e testá-lo em dois projetos independentes. Em paralelo, especificar o AI Control Plane como capacidade transversal, começando por um protótipo pequeno de Token Intelligence, Cost Forecasting e Budget/Activity Planning. Só depois de validar o fluxo de partilha com Claude Code/GitLab e a precisão das previsões devemos transformar padrões em bibliotecas ou serviços comuns.
 
O objetivo não é construir uma plataforma complexa antes de termos produtos. É garantir, desde o início, que cada produto nasce com uma arquitetura suficientemente modular para evoluir, controlar custos e entregar valor a diferentes tipos de clientes.
 