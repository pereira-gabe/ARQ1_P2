# Resolução Lista de Exercícios ARQ1 - P2

## 1

- *RegWrite:* todas instruções tipo R e I e jumps como jalR pois estas instruções precisam escrever no banco de registradores. Se RegWrite tiver travado em zero, o processador nunca salvará o dado de nenhuma conta ou leitur de memória no registrador de destino `rd`.

- *ALUOp:* A maior parte das operações tipo R. Para instruções Tipo-R, o Main Decoder emite ALUOp = 10 (ou seja, ALUOp1 = 1 e ALUOp0 = 0), indicando ao ALU Decoder que ele deve olhar os campos funct da instrução. Se ALUOp1 travar em 0, o sinal vira 00. O ALU Decoder interpretará isso como uma operação de memória e forçará a ULA a fazer sempre uma Soma. Desse modo, um sub ou um and acabarão executando uma soma por erro.

- *MemWrite:* Instrução SW. O sinal MemWrite é o responsável por dar a permissão de escrita (Write Enable) na Memória de Dados (RAM).

- *ImmSrc:* Instruções do Tipo-S (sw) e do Tipo-J (jal). O sinal de controle ImmSrc[1:0] define como os bits da instrução serão reorganizados para gerar o imediato estendido (ImmExt): 00 para Tipo-I, 01 para Tipo-S, 10 para Tipo-B e 11 para Tipo-J. Se o bit menos significativo (ImmSrc0) travar em 0:

- *ResultSrc1:* Instruções de salvo incondicional (jal, jalr). O multiplexador final de escrita seleciona o que vai voltar para os registradores baseado em ResultSrc[1:0]: 00 (ULA), 01 (Memória) e 10 (PC + 4). Se o bit ResultSrc1 estiver travado em 0, a seleção 10 é convertida em 00. Com isso, os saltos falham em salvar o endereço de retorno (PC + 4) no registrador link (ra/x1), quebrando chamadas de funções.

- *PCSrc:* Instruções de desvio no geral, tanto condícionais (beq) quando incondicionais (jal, jalr). O sinal PCSrc controla o multiplexador da extrema esquerda, que decide se o próximo PC será o sequencial (0 para PC + 4) ou o alvo do desvio (1 para PCTarget).

- *ALUSrc:* Instruções tipo I, lw e sw devido à inacessibilidade de imediatos. O sinal ALUSrc decide se a segunda entrada da ULA vem do registrador rs2 (0) ou do extensor de imediato ImmExt (1). Travado em 0, a ULA sempre utilizará o valor do registrador rs2. Instruções aritméticas com constantes e cálculos de endereço de memória (que somam a base ao imediato) falharão completamente por não conseguirem acessar a constante.

---

## 2

### (a) `xor`

**Modificações no Diagrama do Datapath**
* Como o `xor` é uma instrução do **Tipo-R** rdware atual **já possui todo o caminho de dados necessário**Os registradores `rs1` e `rs2` são lidos, o multiplexador da ULA seleciona a saída do registrador (`ALUSrc = 0`), a ULA realiza a operação e o multiplexador final salva o resultado de volta no banco de registradores (`ResultSrc = 00`)huma linha ou bloco extra precisa ser adicionado ao datapath
**Alterações no Decodificador Principal (Main Decoder)**
* O `xor` utiliza o opcode de Tipo-R (`0110011`)tanto, ele reaproveita a linha **R-type** já existente na tabela, mantendo os sinais de controle idênticosInstruction | Opcode | RegWrite | ImmSrc | ALUSrc | MemWrite | ResultSrc | Branch | ALUOp | Jump |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **R-type** | `0110011` | 1 | XX | 0 | 0 | 00 | 0 | **10** | 0 |

**Alterações no Decodificador da ULA (ALU Decoder)**
* É necessário **adicionar uma nova linha** na tabela do ALU Decoder (Table 7.3) para identificar a operação lógica do `xor` através do seu campo `funct3 = 100` e `funct7₅ = 0`

| ALUOp | funct3 | funct7₅ | ALUControl | Instruction |
| :---: | :---: | :---: | :---: | :---: |
| **10** | **100** | **0** | **100** *(xor)* | **xor** |

### (b) `sll`

**Modificações no Diagrama do Datapath**
* Assim como o `xor`, o `sll` (Shift Left Logical) é uma instrução do **Tipo-R**tapath existente é suficiente para transportar os operandos de `rs1` e `rs2` até a ULA e salvar o resultado em `rd`A única modificação necessária é garantir que o circuito interno da **ULA** possua um deslocador lógico capaz de realizar o *shift* de `SrcA` por `SrcB` (usando os 5 bits menos significativos de `SrcB`).

**Alterações no Decodificador Principal (Main Decoder)**
* Por ser Tipo-R, utiliza o opcode `0110011` e ativa exatamente os mesmos sinais da linha **R-type** egando a identificação do shift para o ALU Decoder através do sinal `ALUOp = 10`Alterações no Decodificador da ULA (ALU Decoder)**
* Adiciona-se uma nova linha na Table 7.3 utilizando os identificadores do `sll` (`funct3 = 001` e `funct7₅ = 0`) para gerar um novo código de controle para a ULA (por exemplo, `110` ou outra codificação livre de 3 bits do seu laboratório)

| ALUOp | funct3 | funct7₅ | ALUControl | Instruction |
| :---: | :---: | :---: | :---: | :---: |
| **10** | **001** | **0** | **110** *(sll)* | **sll** |

### (d) `bne`

**Modificações no Diagrama do Datapath**
* A instrução `bne` (Branch if Not Equal) realiza um desvio se os registradores forem **diferentes***Modificação no circuito de controle do PC:** O circuito atual possui uma porta AND que faz `PCSrc = Branch AND Zero` o `bne`, precisamos que o salto aconteça quando `Zero = 0` (ou seja, quando **não** for igual). 
* É necessário adicionar uma pequena lógica na Unidade de Controle (como uma porta XOR ou um multiplexador controlado por um novo sinal `BranchType`) antes da porta AND final, permitindo que o hardware selecione se o salto ativa com `Zero` (para `beq`) ou com `NOT Zero` (para `bne`)lterações no Decodificador Principal (Main Decoder)**
* Adiciona-se uma nova linha na Table 7.6 com o opcode de branches (`1100011`) é praticamente idêntica à do `beq`, mas se você optar por usar um novo sinal de controle (ex: `BNE`), ele deve ser ativado
| Instruction | Opcode | RegWrite | ImmSrc | ALUSrc | MemWrite | ResultSrc | Branch | ALUOp |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **bne** | `1100011` | 0 | 10 *(tipo B)* | 0 | 0 | XX | 1 | 01 |

**Alterações no Decodificador da ULA (ALU Decoder)**
* Nenhuma linha nova é necessáriane` utiliza o mesmo `ALUOp = 01` do `beq` endo com que a ULA execute uma **subtração** para comparar os dois registradores e gerar a flag `Zero`iferença de comportamento será tratada exclusivamente pela nova lógica de portas do PC 

## 3

A principal vantagem é o ganho na quantidade de instruções concluídas por unidade de tempo. Ao paralelizar o fluxo, o processador não precisa esperar uma instrução terminar completamente para iniciar a próxima.<br>
No monociclo, o ciclo de clock é determinado pelo caminho crítico (a instrução mais longa, lw, que passa por todo o circuito). No pipeline, como o caminho é quebrado em 5 estágios isolados por registradores de estágio, o ciclo de clock passa a ser determinado apenas pelo estágio mais lento. Isso permite que o processador opere em frequências de clock muito mais altas.

---

## 4

Conforme o ftiamento dos circuitos é aumentado, o tempo gasto pelo atraso físico e com a sincronização dos registradores de estágio começa a engolir o ganho de processamento obtido. Além disso, criar um pipeline excessivamente longo aumenta drasticamente a penalidade de hazards (conflitos): a distância entre o estágio que gera um dado e o estágio que precisa dele exige circuitos de encaminhamento gigantescos, e errar a previsão de um desvio condicional força o processador a descartar dezenas de instruções que já estavam na esteira.

---

## 5

Hazards em processadores com pipeline podem ser de dois tipos:
- Hazard de dados: Ocorre quando uma instrução no pipeline usa o valor de um registrador que ainda não foi gravado no RegFile por uma instrução anterior.
- Hazard de controle: Ocorre quando a próxima instrução a ser executada ainda não foi decidida devido a um desvio condicional.

Para estes dois tipos, existem diferentes maneiras de tratamento com diferentes vantagens/desvantagens, sendo eles:
- Hazard de Dados 
    - Inserir nops (no operation, `addi x0, x0, 0`) em tempo de compilação, o que gera perda de desempenho.
        - *Vantagem:* Não exige nenhuma lógica ou circuitos adicionais de hardware no chip.
        - *Desvantagem:* Desperdiça ciclos de clock preciosos, reduzindo a vazão de instruções.
    - Reorganizar o código em tempo de compilação para evitar instruções adjacentes que usem o mesmo registrador, o que pode ser inconsistente.
        - *Vantagem:* Otimiza o tempo de execução sem nenhum custo de hardware.
        - *Desvantagem:* Depende da existência de instruções independentes suficientes no programa para preencher os espaços.
    - Fazer Stalls para congelar estágios do pipeline, o que também gera perda de desempenho.
        - *Vantagem:* Garante o funcionamento correto diretamente no hardware, independente do compilador.
        - *Desvantagem:* Reduz o desempenho global ao deixar partes do processador ociosas (insere bolhas).
    - Fazer Encaminhamento (Forwarding) conectando fios e multiplexadores para acessar dados diretamente do barramento dos estágios avançados.
        - *Vantagem:* Resolve a maioria dos conflitos de dados de forma imediata, sem perda de ciclos de clock.
        - *Desvantagem:* Aumenta drasticamente a complexidade do circuito de controle e a área física de silício. *Nota: Um Load (`lw`) seguido de uso imediato ainda exige obrigatoriamente 1 Stall.*
- Hazard de Controle 
    - Fazer Stalls para congelar o pipeline até que a condição de desvio seja resolvida.
        - *Vantagem:* Circuito de controle muito simples e direto de ser projetado no hardware.
        - *Desvantagem:* Gera uma penalidade pesada de desempenho, introduzindo múltiplos ciclos ociosos a cada branch.
    - Fazer a Limpeza do Pipeline (Flush) assumindo que o desvio não será tomado e limpando a esteira caso ele seja.
        - *Vantagem:* Se o desvio não acontecer, o ganho é de 100% (zero ciclos perdidos).
        - *Desvantagem:* Se o desvio for tomado, paga-se a penalidade total limpando e reiniciando a esteira com instruções vazias.
    - Implementar Predição de Desvio (Branch Prediction) estática ou dinâmica no estágio de busca.
        - *Vantagem:* Reduz drasticamente as paradas em estruturas de loops e repetições de código.
        - *Desvantagem:* Exige um hardware de controle extremamente complexo e eleva consideravelmente o consumo de energia.

---

## 6 e 7

De modo geral, os registradores são lidos no estágio ID (*Instruction Decode*) e escritos no estágio WB (*Writeback*)
- 6:
    lidos: s4 e s1 (`sw s4, 20(s1)` no estágio ID)
    escritos: s1 (`xor s1, s2, s3` no estágio WB)

- 7:
    lidos: s0 e s3 (`sw s3, 20(s0)` no estágio ID)
    escritos: s1 (`addi s1, zero` no estágio WB)

---

## 8 

**Análise de Conflitos, Encaminhamentos e Stalls:**

- **`addi` (1) $\rightarrow$ `lw` (2) baseado em `s1`:** O `lw` precisa de `s1` no estágio EX para calcular o endereço. O `addi` calcula o valor no estágio EX. 
    - **Solução:** **Forwarding** direto da saída da ALU (estágio EX/MEM) para a entrada da ALU (estágio ID/EX) no Ciclo 3. **0 stalls.**
- **`lw` (2) $\rightarrow$ `lw` (3) baseado em `s2`:** O segundo `lw` precisa de `s2` para calcular seu endereço no estágio EX. Como `s2` vem de um carregamento de memória, o dado só fica pronto no final do estágio MEM do primeiro `lw` (conflito de *Load-Use*).
    - **Solução:** **1 Stall** no Ciclo 4. O dado é então **encaminhado** da saída da memória (estágio MEM/WB) para a entrada da ALU no Ciclo 5.
- **`lw` (3) $\rightarrow$ `add` (4) baseado em `s5`:** O `add` precisa de `s5` vindo do `lw` para realizar a soma no estágio EX. Cai novamente no caso de conflito *Load-Use*.
    - **Solução:** **1 Stall** no Ciclo 6. O dado é **encaminhado** da saída da memória (estágio MEM/WB) para a ALU no Ciclo 7.
- **`add` (4) $\rightarrow$ `or` (5) baseado em `s3`:** O `or` depende do resultado da soma.
    - **Solução:** **Forwarding** direto da saída da ALU (EX/MEM) para a entrada da ALU (ID/EX). **0 stalls.**
- **`or` (5) $\rightarrow$ `and` (6) baseado em `s4`:** O `and` depende do resultado da operação lógica `or`.
    - **Solução:** **Forwarding** direto da saída da ALU (EX/MEM) para a entrada da ALU (ID/EX). **0 stalls.**

---

**Diagrama de Execução do Pipeline (Mapeamento de Ciclos)**

Abaixo está o diagrama temporal de execução demonstrando o impacto das bolhas (*stalls*) injetadas pela **Hazard Unit**:

```text
Instrução / Ciclos      | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10 | 11 | 12 |
-------------------------------------------------------------------------------------
1. addi s1, zero, 11    | IF | ID | EX | MEM| WB |    |    |    |    |    |    |    |
2. lw s2, 25(s1)        |    | IF | ID | EX | MEM| WB |    |    |    |    |    |    |
3. lw s5, 16(s2)        |    |    | IF | ID | * | EX | MEM| WB |    |    |    |    |
4. add s3, s2, s5       |    |    |    | IF | * | ID | * | EX | MEM| WB |    |    |
5. or s4, s3, t4        |    |    |    |    | * | IF | * | ID | EX | MEM| WB |    |
6. and s2, s3, s4       |    |    |    |    |    |    | * | IF | ID | EX | MEM| WB |

Legenda: * = Ciclo de Parada (Stall / Bolha introduzida no estágio correspondente)
```
---

## 9 

Vamos analisar o comportamento das instruções na esteira do pipeline, identificando os adiantamentos (*forwarding*) e se há necessidade de congelar o processador:

- **`addi` (1) $\rightarrow$ `addi` (2) baseado em `s1`:** O dado fica pronto na saída da ULA (estágio EX). A Hazard Unit resolve o conflito enviando o valor diretamente para a entrada da ULA da instrução seguinte via **Forwarding**. **0 stalls.**
- **`addi` (2) $\rightarrow$ `lw` (3) baseado em `s0`:** O endereço base do `lw` depende de `s0`. Conflito resolvido integralmente por **Forwarding** de EX para EX. **0 stalls.**
- **`lw` (3) $\rightarrow$ `sw` (4) baseado em `s3`:** A instrução `sw` precisa de `s3` para salvá-lo na memória de dados. Na microarquitetura clássica do RISC-V, há um caminho de encaminhamento dedicado que injeta o dado vindo do estágio MEM direto para o barramento de escrita de dados (`WriteData`). Portanto, **não ocorre stall** entre um `lw` e um `sw` imediatamente vizinhos que compartilham o mesmo dado. **0 stalls.**
- **`lw` (3) $\rightarrow$ `xor` (5) baseado em `s3`:** A instrução `xor` utiliza `s3` em sua operação lógica no estágio EX. Como o dado trazido do `lw` só sai da memória no final do estágio MEM, ocorre um conflito de uso de carga (*Load-Use Hazard*).
    - **Solução:** A Hazard Unit introduz obrigatoriamente **1 Stall** para adiar a execução do `xor`.
- **`xor` (5) $\rightarrow$ `or` (6) baseado em `s2`:** O `or` depende do resultado do `xor`. Conflito resolvido via **Forwarding** de EX para EX. **0 stalls.**

Abaixo está a grade temporal de execução passo a passo do trecho, exibindo graficamente o impacto do ciclo de parada (*stall*) inserido no hardware`text
Instrução / Ciclos      | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10 |
---------------------------------------------------------------------------
1. addi s1, zero, 52    | IF | ID | EX | MEM| WB |    |    |    |    |    |
2. addi s0, s1, -4      |    | IF | ID | EX | MEM| WB |    |    |    |    |
3. lw s3, 16(s0)        |    |    | IF | ID | EX | MEM| WB |    |    |    |
4. sw s3, 20(s0)        |    |    |    | IF | ID | EX | MEM| WB |    |    |
5. xor s2, s0, s3       |    |    |    |    | IF | ID | * | EX | MEM| WB |
6. or s2, s2, s3        |    |    |    |    |    | IF | * | ID | EX | MEM|

Legenda: * = Ciclo de Parada (Stall / Bolha injetada no estágio)

--- 

## 10

O conceito de **hierarquia de memórias** consiste na organização de diferentes tecnologias de armazenamento em uma estrutura de pirâmide, visando balancear o conflito entre **velocidade, capacidade e custo**. Conforme subimos na hierarquia, as memórias tornam-se mais rápidas, mais caras por bit e com menor capacidade de armazenamento; conforme descemos, a capacidade aumenta, o custo por bit cai, mas o tempo de acesso torna-se consideravelmente mais lento.

A ordem de prioridade pela qual o processador busca um dado, do topo à base da hierarquia, é composta pelos seguintes níveis físicos:

- **Nível 0 - Registradores:** Localizados dentro do próprio núcleo do processador. É o nível mais rápido de todos, operando na velocidade nativa do clock (0 ciclos de atraso). Possui uma capacidade minúscula (apenas algumas centenas de bytes no total) e armazena os dados que estão sendo processados no exato instante atual.
- **Nível 1 - Memória Cache (SRAM):** É uma memória estática muito rápida integrada diretamente no chip do processador para evitar gargalos. Possui baixo volume de armazenamento e é dividida em subníveis de proximidade e velocidade: o cache L1 (mais rápido e menor), L2 (intermediário) e L3 (maior, compartilhado entre os núcleos).
- **Nível 2 - Memória Principal (RAM/DRAM):** É uma memória dinâmica volátil que armazena os programas e dados em execução pelo sistema operacional. Devido à sua microarquitetura baseada em capacitores, possui uma densidade muito maior e custo menor que o cache, permitindo volumes na casa dos Gigabytes. É consideravelmente mais lenta que a memória cache e sua conexão com a CPU é feita externamente através de barramentos.
- **Nível 3 - Memória Secundária (HDD / SSD):** É o nível de armazenamento não-volátil em massa (armazenamento magnético ou flash). Possui o maior volume de dados de todo o sistema (na casa dos Terabytes), porém é extremamente lenta em comparação com as memórias cache e principal. O processador não acessa este nível diretamente para executar instruções; os dados precisam ser copiados do disco para a memória RAM antes de serem processados.

---

## 11

O Memory Gap (Lacuna de Memória ou Gargalo de Memória) é o fenômeno que descreve a gigantesca disparidade entre o ritmo de evolução dos processadores e o ritmo de evolução das memórias principais (RAM) ao longo das últimas décadas. O impacto direto do memory gap é o surgimento de um gargalo conhecido como Von Neumann Bottleneck (Gargalo de Von Neumann) ou Memory Wall.<br>
Desperdício de Ciclos da CPU (Stalls): Como o processador opera em frequências de Gigahertz extremamente rápidas, toda vez que ele precisa buscar um dado ou instrução diretamente na memória RAM, ele é obrigado a "congelar" e esperar por dezenas ou até centenas de ciclos de clock até que a memória envie a informação.<br>
Processador Ocioso: O desempenho real do computador passa a ser ditado não pela velocidade da CPU, mas sim pela lentidão da memória. O processador passa a maior parte do tempo ocioso, esperando por dados, anulando o potencial do seu poder de processamento.<br>
É justamente para mitigar o impacto destruidor do memory gap que os engenheiros criaram a Memória Cache. Colocando uma memória estática (SRAM) ultra-rápida dentro do chip da CPU, o processador consegue achar a maioria dos dados de forma imediata, recorrendo à lenta memória RAM externa apenas em caso de falha (cache miss).

---

## 12

O Princípio da Localidade dita que os programas acessam dados de forma previsível e não aleatória, dividindo-se em localidade temporal, onde um dado acessado tende a ser reutilizado em breve (como variáveis em loops), e localidade espacial, onde dados em endereços vizinhos tendem a ser acessados em sequência (como vetores e códigos sequenciais); a memória cache se aproveita disso para trazer blocos inteiros de dados adjacentes da RAM e mantê-los perto do processador, alcançando altas taxas de acerto. Quando a cache enche, entram em cena as políticas de substituição, que usam essa lógica para decidir qual dado antigo descartar para abrir espaço para o novo, destacando-se a política LRU (Least Recently Used), que descarta o bloco há mais tempo sem uso por presumir que ele não será necessário logo, a FIFO (First-In, First-Out), que remove a informação mais antiga na cache independente de seu uso recente, e a Random, que escolhe um bloco aleatoriamente para economizar complexidade de hardware.

---

## 13

A hierarquia de memória é eficiente porque se baseia no Princípio da Localidade, aproveitando o fato de que os programas tendem a reutilizar os mesmos dados recentemente acessados (localidade temporal) e a acessar endereços vizinhos em sequência (localidade espacial); assim, ao manter uma memória Cache (SRAM) pequena e ultra-rápida integrada diretamente ao chip do processador, o sistema consegue fornecer de forma quase instantânea (geralmente em 1 ciclo de clock) a imensa maioria das informações que a CPU precisa no dia a dia, alcançando taxas de acerto superiores a 90% e camuflando com sucesso a lentidão crônica e a distância da memória principal (RAM), o que evita que o processador fique ocioso esperando por dados e maximiza o desempenho global do computador com o melhor custo-benefício possível.

---

## 14

lol. acho que não cai na prova. preguiça<br>

resposta geminística abaixo:<br>

O Write-Through atualiza a memória cache e a RAM de forma simultânea a cada operação de escrita, garantindo consistência total e imediata dos dados, mas sacrificando o desempenho ao prender a CPU à lentidão do barramento externo da RAM; por outro lado, o Write-Back realiza as modificações temporariamente apenas dentro da própria cache em alta velocidade, sinalizando a linha modificada através de um Dirty Bit, de modo que a memória RAM só é atualizada tardiamente quando esse bloco modificado é descartado para abrir espaço, oferecendo desempenho muito superior ao reduzir drasticamente o tráfego de dados nos barramentos do sistema.