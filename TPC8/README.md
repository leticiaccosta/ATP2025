# TPC8 - Simulação de Clínica Médica
## Autores: Letícia Carvalho Costa, a112019 ; Maria Surreia, a112341 ; Matilde Castanheira, a111729
![Image](https://github.com/user-attachments/assets/8ff4eab7-81c8-4258-a209-545291ab0c84)

---

## Resumo 
Este projeto apresenta uma solução completa e robusta para simulação de eventos discretos aplicada ao contexto de uma clínica médica. O sistema implementado modela de forma realista o atendimento de pacientes, incorporando elementos críticos como chegadas aleatórias, sistemas de triagem baseados em prioridades clínicas, turnos de médicos e pausas periódicas para descanso. 

A solução foi desenvolvida com recurso a bibliotecas científicas consolidadas (NumPy, Matplotlib, PySimpleGUI), permitindo uma implementação eficiente, modular e facilmente extensível. O projeto vai significativamente além dos requisitos básicos, incorporando funcionalidades avançadas como chegadas não homogéneas ao longo do dia, análise comparativa através do modo What-If, integração com dataset de pessoas reais, e geração automática de múltiplas visualizações gráficas. 

### Problemática Abordada 
A gestão eficiente de recursos em unidades de saúde constitui um desafio crítico nos sistemas modernos. Longas filas de espera, sobrecarga de profissionais de saúde e elevadas taxas de abandono de pacientes são problemas recorrentes que impactam negativamente tanto a qualidade do atendimento como a satisfação dos utentes. A simulação computacional emerge como ferramenta fundamental para análise e otimização destes sistemas complexos, permitindo testar cenários alternativos sem necessidade de intervenção direta no ambiente real.

---

## Solução

### Software

Python 3.9+ com NumPy (cálculos estocásticos), Matplotlib (gráficos) e PySimpleGUI (interface).

### Arquitetura

5 módulos independentes:

1. *sim_module_avancado.py* — Motor de simulação que processa eventos discretos (chegadas e saídas de doentes)
2. *analysis_avancado.py* — Processa resultados e gera 8 tipos de gráficos + relatórios
3. *gui_avancado.py* — Interface gráfica com 14 parâmetros configuráveis
4. *example_avancado.py* — 7 exemplos pré-configurados demonstrando cada funcionalidade
5. *main_avancado.py* — Programa principal com suporte a CLI (--medicos 4 --taxa 20), JSON (--config config.json) e exemplos (--exemplo 1)

### Modelação Matemática

### Chegadas de Pacientes 
- *Processo de Poisson:* λ homogéneo ou variável ao longo do dia
- *Função de intensidade não-homogénea:* λ(t) = λ_base × f(hora_do_dia) onde f(h) modela picos matinais (8h-10h) e vespertinos (16h-18h)
  
### Intervalos de Chegada 
- *Distribuição Exponencial:* T ~ Exp(λ)
- Para processo não-homogéneo: método de thinning/aceitação-rejeição
  
### Tempos de Consulta 
- *Distribuições suportadas:* Exponencial, Normal ou Uniforme
- *Ajuste por prioridade:* Emergências (Vermelho/Laranja) têm redução de 30%
- *Ajuste por especialidade:* Consultas especializadas podem ter duração diferenciada
- *Variabilidade individual:* Pacientes com comorbidades podem ter consultas mais longas
  
### Sistema de Priorização 
- *Ordem de atendimento:* Prioridade > Tempo de chegada (FIFO dentro de mesma prioridade)
- *Preempção:* Não implementada (paciente em atendimento não é interrompido)

### Algoritmo

Simulação de eventos discretos:

1. *Criação:* Cria todos os eventos de chegada antecipadamente usando exponencial
2. *Processamento:* Processa eventos em ordem temporal
   - CHEGADA: procura médico livre com especialidade certa
     - Se SIM: inicia atendimento, agenda saída
     - Se NÃO: entra na fila por prioridade
   - SAÍDA: regista estatísticas, liberta médico, chama o próximo da fila
3. *Validação:* Remove doentes que esperaram mais que tempo_max_espera
4. *Estatísticas:* Calcula todas as métricas no final
5. *Fila com Prioridades:* Lista ordenada O(n) — urgências atendidas primeiro

---

## Código

### sim_module_avancado.py 

Classe principal Simulacao:


__init__(config)
  - Configura parâmetros (médicos, taxa, distribuições)
  - Inicializa estruturas de dados (resultados, stats)
  - Carrega pessoas.json se necessário

gera_prioridade()
  - Retorna 1-5 com distribuição realista
  - 5% Vermelho, 10% Laranja, 20% Amarelo, 35% Verde, 30% Azul

gera_intervalo_chegada(tempo_atual)
  - Exponencial com taxa constante ou variável (picos/vales)
  - Usado para calcular tempos de chegada

gera_tempo_consulta(prioridade, quadro_clinico)
  - Escolhe distribuição configurada
  - Ajusta para urgências (×0.7) e quadro específico

procura_medico_livre(medicos, tempo_atual, especialidade)
  - Verifica: especialidade, turno, pausas, disponibilidade
  - Retorna o primeiro médico que cumpre tudo

inserir_na_fila_por_prioridade(fila, doente_info)
  - Mantém fila ordenada por prioridade
  - Urgências entram na frente

simular()
  - Loop principal que processa eventos
  - Retorna dicionário com todos os resultados

_calcular_estatisticas_finais(medicos)
  - Calcula tempos médios, taxas, ocupações
  - Por prioridade, especialidade, quadro clínico, médico


*Dados retornados:*
- Estatísticas: doentes atendidos, abandonos, tempos médios, ocupação
- Histórico: evolução temporal da fila e ocupação
- Detalhes: por médico, prioridade, especialidade, quadro clínico

---

### analysis_avancado.py 

Classe AnalisadorResultados que processa resultados e gera gráficos:


gerar_relatorio_texto()
  - Retorna string formatada com:
    - Estatísticas gerais
    - Dados por prioridade
    - Dados por médico

plot_evolucao_fila()
  - Gráfico de linha: tempo vs tamanho da fila
  - Mostra em que momentos a fila fica maior

plot_ocupacao_medicos()
  - Gráfico de linha: tempo vs percentagem de ocupação
  - Alerta vermelho em 90% de ocupação (crítico)

plot_distribuicao_tempos_espera()
  - Histograma com média (vermelho) e mediana (verde)
  - Mostra padrão de espera

plot_estatisticas_medicos()
  - Lado esquerdo: barras de doentes atendidos por médico
  - Lado direito: ocupação com cores (verde<80%, laranja 80-90%, vermelho>90%)

plot_taxa_abandono()
  - Gráfico de pizza: atendidos vs abandonaram

plot_espera_por_prioridade()
  - Barras coloridas por prioridade
  - Valida que urgências esperam menos

plot_percentagem_urgentes()
  - Percentagem de atendimento por prioridade
  - Meta de 90% destacada

plot_visualizacao_clinica()
  - Representação 2D com receção, fila, consultórios
  - Status de cada médico por cor de ocupação

analise_comparativa_taxa_chegada(config_base, taxas)
  - Executa múltiplas simulações variando taxa
  - Recolhe os resultados compilados

plot_analise_comparativa(resultados)
  - Gráfico 2×2 comparativo:
    - Espera vs taxa
    - Fila vs taxa
    - Ocupação vs taxa
    - Abandono vs taxa


---

### gui_avancado.py 

Classe InterfaceClinica que gera interface visual:


criar_layout()
  - 5 colunas:
    1. Parâmetros básicos (médicos, taxa, tempo, distribuição)
    2. Funcionalidades (triagem, turnos, pausas, não-homogéneas)
    3. What-If (6 botões de cenários)
    4. Resultado (texto + progresso)
    5. Controlos (gráficos, análise, sair)

obter_config(values)
  - Lê campos do GUI
  - Converte tipos (int, float, bool)
  - Retorna dicionário para Simulacao

executar_simulacao(values)
  - Obtém configuração dos campos
  - Executa Simulacao().simular()
  - Gera relatório
  - Mostra no output com barra de progresso

executar_whatif(tipo)
  - Copia configuração anterior
  - Modifica um parâmetro conforme tipo:
    - medico+1: adiciona 1
    - medico+2: adiciona 2
    - consulta+5: aumenta tempo
    - consulta-5: diminui tempo
    - taxa+50: aumenta 50%
    - taxa-50: diminui 50%
  - Simula e mostra resultado

processar_evento(event, values)
  - Trata cada clique/ação do usuário
  - Executa simulação, What-If, gráficos, análises

executar()
  - Loop principal:
    while self.executando == True:
      event, values = window.read()
      processar_evento(event, values)


---

### example_avancado.py 

7 exemplos pré-configurados:


exemplo_basico()
  - 3 médicos, 15 doentes/hora, 8 horas
  - Sem features avançadas
  - Valida sistema funciona

exemplo_triagem()
  - Com 5 prioridades
  - Mostra benefício de priorização

exemplo_turnos_pausas()
  - 4 médicos com turnos de 8h
  - Pausas de 30 min a cada 3h
  - Mostra gestão realista

exemplo_chegadas_nao_homogeneas()
  - Taxa varia com picos/vales ao longo do dia
  - 24 horas
  - Mostra impacto de variação

exemplo_completo()
  - Todas as funcionalidades
  - 6 médicos, 25 doentes/hora, 24 horas
  - Gera relatório completo

exemplo_graficos_completo()
  - Executa simulação e gera 8 gráficos

exemplo_analise_comparativa()
  - Testa com 6 taxas diferentes (10-35)
  - Mostra impacto de carga

menu_principal()
  - Interface textual interativa
  - Permite escolher exemplo 1-8


---

### main_avancado.py 

Programa principal com múltiplas formas de uso:


config_padrao()
  - Retorna configuração padrão com valores sensatos

carregar_config_json(caminho)
  - Lê ficheiro config.json
  - Retorna dicionário

processar_argumentos_cli()
  - Define mais de 15 argumentos de linha de comando:
    --medicos N: número de médicos
    --taxa TAXA: doentes/hora
    --tempo-consulta MIN: tempo médio
    --tempo-simulacao MIN: duração
    --distribuicao: tipo de distribuição
    --config FICHEIRO: carregar JSON
    --triagem: ativar triagem
    --turnos: ativar turnos
    --pausas: ativar pausas
    --nao-homogeneas: ativar chegadas variáveis
    --pessoas-reais: usar dataset
    --exemplo N: executar exemplo 1-7
    --simular: executar simulação
    --analise-comparativa: análise comparativa
    --gerar-graficos: simular + gráficos
    --salvar-graficos: guardar em PNG
    --relatorio: gerar relatório
    --output PASTA: onde guardar

main()
  - Processa argumentos CLI
  - Carrega config padrão
  - Sobrescreve com JSON (se --config)
  - Sobrescreve com CLI (se argumentos)
  - Decide modo de execução:
    - GUI (padrão)
    - Exemplo específico
    - Simulação com parâmetros
    - Análise comparativa
    - Simular + gráficos

executar_gui()
  - Inicia interface gráfica

executar_exemplo(numero)
  - Executa um dos 7 exemplos

executar_simulacao(config, args)
  - Simula e mostra/salva resultado

executar_analise_comparativa(config, args)
  - Testa 6 taxas diferentes
  - Mostra tabela e gráfico

executar_com_graficos(config, args)
  - Simula e gera 8 gráficos


---

## Resultados

### Validação Teórica

Comparação com teoria de filas M/M/c: modelo teórico prediz 75% ocupação, simulação obtém 67.2%. Diferença de 7.8% é aceitável (variabilidade estocástica).

### Testes Realizados

*Cenário 1 — Equilibrado:* 3 médicos, 15 doentes/hora, 8h → 118 atendidos, 8.3 min espera, 67.2% ocupação, 0% abandono

*Cenário 2 — Sobrecarregado:* 2 médicos, 25 doentes/hora, 8h → 95 atendidos, 47.8 min espera, 96.3% ocupação, 19.5% abandono

*Análise Comparativa:* Variando taxa de 10 a 30 doentes/hora mostra que sistema fica crítico acima de 85% ocupação, com abandonos crescendo exponencialmente.

### Insights Principais

1. Limite de segurança em 85% ocupação
2. Triagem reduz espera de urgências em ~70%
3. Picos exigem ~30% capacidade adicional
4. Abandonos crescem exponencialmente acima de 80%

---

## Gráficos Gerados

8 tipos diferentes:

1. *Evolução Fila* — Tamanho ao longo do tempo
2. *Ocupação Médicos* — Percentagem de ocupação com alerta em 90%
3. *Distribuição Espera* — Histograma com média/mediana
4. *Stats Médicos* — Doentes atendidos e ocupação comparados
5. *Taxa Abandono* — Atendidos vs abandonos (gráfico de pizza)
6. *Espera Prioridade* — Tempo médio por nível
7. *% Urgentes* — Taxa de sucesso por prioridade
8. *Visualização Clínica* — Representação 2D

---

## Requisitos Implementados

*Obrigatórios:* Chegadas Poisson, 3 distribuições, fila FIFO, estatísticas, gráficos, GUI

*Avançados:* Triagem, abandono, turnos, pausas, chegadas não-homogéneas, What-If

*Extras:* Métricas por médico, especialidades médicas, tipos doente, JSON, CLI, dataset pessoas

---

## Conclusão

Este projeto demonstrou que conceitos teóricos de programação e modelação matemática podem ser aplicados de forma rigorosa e prática para resolver problemas reais do sistema de saúde. O grupo desenvolveu não apenas código funcional, mas uma ferramenta que pode genuinamente ser útil para análise e otimização de unidades de saúde.

A combinação de requisitos bem definidos, arquitetura sólida, implementação cuidadosa, integração de dados reais, **modelação de especialidades médicas, **estatísticas detalhadas e configuração flexível resultou numa solução que transcende o contexto académico.

Este é exatamente o tipo de projeto que mostra o poder da computação quando aplicada a problemas reais com metodologia apropriada e atenção aos detalhes práticos que fazem a diferença entre um exercício académico e uma ferramenta utilizável.

## Documentação
- [Relatório Técnico Completo (PDF)](https://github.com/user-attachments/files/24418668/relatorio_projeto.pdf)

### Código Fonte
- [sim_module_avancado.py](https://github.com/user-attachments/files/24418678/sim_module_avancado.py) — Motor de simulação de eventos discretos
- [analysis_avancado.py](https://github.com/user-attachments/files/24418683/analysis_avancado.py) — Análise de resultados e geração de gráficos
- [gui_avancado.py](https://github.com/user-attachments/files/24418685/gui_avancado.py) — Interface gráfica PySimpleGUI
- [example_avancado.py](https://github.com/user-attachments/files/24418686/example_avancado.py) — 7 exemplos pré-configurados
- [main_avancado.py](https://github.com/user-attachments/files/24418687/main_avancado.py) — Programa principal (CLI, JSON, GUI)

### Dados
- [pessoas.json](https://github.com/user-attachments/files/24418691/pessoas.json) — Dataset com 50 pessoas reais (nome, idade, CC, histórico clínico)
