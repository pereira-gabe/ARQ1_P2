# Resolução Lista de Exercícios ARQ1 - P2

## 1

- *RegWrite:* todas instruções tipo R e I e jumps como jalR pois estas instruções precisam escrever no banco de registradores. Se RegWrite tiver travado em zero, o processador nunca salvará o dado de nenhuma conta ou leitur de memória no registrador de destino `rd`.

- *ALUOp:* A maior parte das operações tipo R. Para instruções Tipo-R, o Main Decoder emite ALUOp = 10 (ou seja, ALUOp1 = 1 e ALUOp0 = 0), indicando ao ALU Decoder que ele deve olhar os campos funct da instrução. Se ALUOp1 travar em 0, o sinal vira 00. O ALU Decoder interpretará isso como uma operação de memória e forçará a ULA a fazer sempre uma Soma. Desse modo, um sub ou um and acabarão executando uma soma por erro.

- *MemWrite:* Instrução SW. O sinal MemWrite é o responsável por dar a permissão de escrita (Write Enable) na Memória de Dados (RAM).

- *ImmSrc:* Instruções do Tipo-S (sw) e do Tipo-J (jal). O sinal de controle ImmSrc[1:0] define como os bits da instrução serão reorganizados para gerar o imediato estendido (ImmExt): 00 para Tipo-I, 01 para Tipo-S, 10 para Tipo-B e 11 para Tipo-J. Se o bit menos significativo (ImmSrc0) travar em 0:

- *ResultSrc1:* Instruções de salvo incondicional (jal, jalr). O multiplexador final de escrita seleciona o que vai voltar para os registradores baseado em ResultSrc[1:0]: 00 (ULA), 01 (Memória) e 10 (PC + 4). Se o bit ResultSrc1 estiver travado em 0, a seleção 10 é convertida em 00. Com isso, os saltos falham em salvar o endereço de retorno (PC + 4) no registrador link (ra/x1), quebrando chamadas de funções.

- *PCSrc:* Instruções de desvio no geral, tanto condícionais (beq) quando incondicionais (jal, jalr). O sinal PCSrc controla o multiplexador da extrema esquerda, que decide se o próximo PC será o sequencial (0 para PC + 4) ou o alvo do desvio (1 para PCTarget).

- *ALUSrc:* Instruções tipo I, lw e sw devido à inacessibilidade de imediatos. O sinal ALUSrc decide se a segunda entrada da ULA vem do registrador rs2 (0) ou do extensor de imediato ImmExt (1). Travado em 0, a ULA sempre utilizará o valor do registrador rs2. Instruções aritméticas com constantes e cálculos de endereço de memória (que somam a base ao imediato) falharão completamente por não conseguirem acessar a constante.

## 2

### (a) `xor`

**Modificações no Diagrama do Datapath**
* [cite_start]Como o `xor` é uma instrução do **Tipo-R** [cite: 75][cite_start], o hardware atual **já possui todo o caminho de dados necessário**[cite: 72, 73]. [cite_start]Os registradores `rs1` e `rs2` são lidos, o multiplexador da ULA seleciona a saída do registrador (`ALUSrc = 0`), a ULA realiza a operação e o multiplexador final salva o resultado de volta no banco de registradores (`ResultSrc = 00`)[cite: 119]. [cite_start]Nenhuma linha ou bloco extra precisa ser adicionado ao datapath[cite: 72, 73].

**Alterações no Decodificador Principal (Main Decoder)**
* [cite_start]O `xor` utiliza o opcode de Tipo-R (`0110011`)[cite: 119]. [cite_start]Portanto, ele reaproveita a linha **R-type** já existente na tabela, mantendo os sinais de controle idênticos[cite: 119]:

| Instruction | Opcode | RegWrite | ImmSrc | ALUSrc | MemWrite | ResultSrc | Branch | ALUOp | Jump |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **R-type** | `0110011` | 1 | XX | 0 | 0 | 00 | 0 | **10** | 0 |

**Alterações no Decodificador da ULA (ALU Decoder)**
* [cite_start]É necessário **adicionar uma nova linha** na tabela do ALU Decoder (Table 7.3) para identificar a operação lógica do `xor` através do seu campo `funct3 = 100` e `funct7₅ = 0`[cite: 74, 121]:

| ALUOp | funct3 | funct7₅ | ALUControl | Instruction |
| :---: | :---: | :---: | :---: | :---: |
| **10** | **100** | **0** | **100** *(xor)* | **xor** |

---

### (b) `sll`

**Modificações no Diagrama do Datapath**
* [cite_start]Assim como o `xor`, o `sll` (Shift Left Logical) é uma instrução do **Tipo-R**[cite: 76]. [cite_start]O datapath existente é suficiente para transportar os operandos de `rs1` e `rs2` até a ULA e salvar o resultado em `rd`[cite: 72, 73]. A única modificação necessária é garantir que o circuito interno da **ULA** possua um deslocador lógico capaz de realizar o *shift* de `SrcA` por `SrcB` (usando os 5 bits menos significativos de `SrcB`).

**Alterações no Decodificador Principal (Main Decoder)**
* [cite_start]Por ser Tipo-R, utiliza o opcode `0110011` e ativa exatamente os mesmos sinais da linha **R-type** [cite: 119][cite_start], delegando a identificação do shift para o ALU Decoder através do sinal `ALUOp = 10`[cite: 119].

**Alterações no Decodificador da ULA (ALU Decoder)**
* [cite_start]Adiciona-se uma nova linha na Table 7.3 utilizando os identificadores do `sll` (`funct3 = 001` e `funct7₅ = 0`) para gerar um novo código de controle para a ULA (por exemplo, `110` ou outra codificação livre de 3 bits do seu laboratório)[cite: 74, 121]:

| ALUOp | funct3 | funct7₅ | ALUControl | Instruction |
| :---: | :---: | :---: | :---: | :---: |
| **10** | **001** | **0** | **110** *(sll)* | **sll** |

---

### (d) `bne`

**Modificações no Diagrama do Datapath**
* [cite_start]A instrução `bne` (Branch if Not Equal) realiza um desvio se os registradores forem **diferentes**[cite: 77]. 
* [cite_start]**Modificação no circuito de controle do PC:** O circuito atual possui uma porta AND que faz `PCSrc = Branch AND Zero`[cite: 15]. Para o `bne`, precisamos que o salto aconteça quando `Zero = 0` (ou seja, quando **não** for igual). 
* [cite_start]É necessário adicionar uma pequena lógica na Unidade de Controle (como uma porta XOR ou um multiplexador controlado por um novo sinal `BranchType`) antes da porta AND final, permitindo que o hardware selecione se o salto ativa com `Zero` (para `beq`) ou com `NOT Zero` (para `bne`)[cite: 73].

**Alterações no Decodificador Principal (Main Decoder)**
* [cite_start]Adiciona-se uma nova linha na Table 7.6 com o opcode de branches (`1100011`)[cite: 119]. [cite_start]Ela é praticamente idêntica à do `beq`, mas se você optar por usar um novo sinal de controle (ex: `BNE`), ele deve ser ativado[cite: 73, 74]:

| Instruction | Opcode | RegWrite | ImmSrc | ALUSrc | MemWrite | ResultSrc | Branch | ALUOp |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **bne** | `1100011` | 0 | 10 *(tipo B)* | 0 | 0 | XX | 1 | 01 |

**Alterações no Decodificador da ULA (ALU Decoder)**
* [cite_start]Nenhuma linha nova é necessária[cite: 74]. [cite_start]O `bne` utiliza o mesmo `ALUOp = 01` do `beq` [cite: 119][cite_start], fazendo com que a ULA execute uma **subtração** para comparar os dois registradores e gerar a flag `Zero`[cite: 121]. [cite_start]A diferença de comportamento será tratada exclusivamente pela nova lógica de portas do PC[cite: 73].