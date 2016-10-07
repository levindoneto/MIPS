# MIPS Modified
__Autores:__ Felipe Tormes, Levindo Neto, Valentina Montserrat

### Introdução

Foram adicionadas as instruções JR, DIV e MFLO nos processadores MIPS monociclo, multiciclo e pipeline. A descrição sobre o que o aluno desenvolveu ao longo do trabalho prático segue a seguir. São mostradas as instruções, com suas respectivas modificações para funcionamento em cada um dos processadores.

### JR - Jump register
#### Monociclo
Foi trocado o sinal de controle “auto” por um sinal de controle “manual”, a fim de que se possa fazer o controle do processador de acordo com as instruções adicionadas, sem prejudicar as que já estavam lá. 
Deixando o manual zerado, o bloco de controle é utilizado.
O sinal de controle é ligado a um not gate para que quando “manual” tivesse em 1, os sinais de controle ativos passassem para o MUX de controle.
Foi criado um comparador de 3 bits e um pino de controle de saída chamado JR, precisando apenas o sinal de igual. 
Esse comparador (apresentado na figura 01) serve para saber o opcode referente ao JR, para ele ser executado.

![Figura 01: Comparador para indicar quando o JR é utilizado.](/resources/figura01.jpg)

Anteriormente o PC a cada instrução somava +4, porém agora, com o JR, teve que se fazer um multiplexador para saber se ele incrementa 4 ou se vai para a posição referente ao JR. O multiplexador e toda parte de circuito utilizada pelo JR,  são mostrados na figura 02, que encontra-se na página seguinte.

![Figura 02: Circuito do MIPS Monociclo com as modificações para executar o JR.](/resources/figura02.jpg)

Teste unitário realizado:

```assembly
lw $r5 0x0 $r8; r5  ← MEM(0) | 0x8D050000
jr $r5; PC ←  r5 | 0x00A00008
```

Foi carregado o valor 0xff da posição de memória zero (0x8c020000) no registrador $r2, e pode-se notar, na figura 03, que após a execução da instrução de jr $r2 (0x600008) que o PC foi para o endereço 0xff.

![Figura 03: PC após a execução do JR apresentado no teste.](/resources/figura03.jpg)

#### Multiciclo
A mesma ideia do circuito monociclo foi utilizada no multiciclo, na qual utiliza um MUX que seleciona se a entrada do PC será pelo que tem sido incrementado em 4 ou se terá como endereço o registrador passado como parâmetro a instrução JR. 
O MUX e sua localização no circuito, são mostrados na figura 04.

![Figura 04: Circuito com foco no MUX utilizado no JR.](/resources/figura04.jpg)

A modificação na máquina de estados para o JR foi apenas depois do estado de busca e decodificação, tendo um estado a partir do ID e que é ligado depois na saída, a qual leva novamente ao IF. 
Esse estado acontece apenas quando a instrução decodificada pelo processador é um Jump Register. 
A máquina de estados finitos modificada é mostrada na figura 05.

![Figura 05: Máquina de Estados Finitos modificada.](/resources/figura05.jpg)

#### Pipeline

Foi mudado o seletor do MUX do PC para controlar se o que vai para o PC é o valor do registrador que JR utiliza ou se incrementa o PC em 4 normalmente.

![Figura 06: Figura que representa o MUX necessário ao JR.](/resources/figura06.jpg)

Foi adicionado, também, um comparador saída da ALU Control para comparar com 3 para saber quando está executando o JR, apresentado na figura 07.

![Figura 07: Comparador para saber quando o JR entrou no estágio de execução.](/resources/figura07.jpg)

### DIV (Divide)

#### Monociclo

Para fazer a divisão, foi implementado um divisor de 32 bits na ALU, com duas entradas de 32 bits (uma para o dividendo e outra para o divisor), essa modificação é apresentada na figura 08.

![Figura 08: Divisor de 32 bits adicionado na ALU.](/resources/figura08.jpg)

Um pino de controle com output de 3 bits foi colocado para comparar 111(2) (opcode da divisão) com os três bits de saída da ALU Control, para saber quando a instrução de divisão for executada. 
O comparador e o pino são apresentados na figura 09.

![Figura 09: Pino para saber quando a divisão é executada.](/resources/figura09.jpg)

A divisão é feita de maneira a guardar o resultado da divisão em um registrador lógico LO, assim como o resto da mesma é colocado em um registrador lógico HI.
```assembly
LO = R[s] div R[t];       O LO guarda o resultado da divisão (Saída).
HI = R[s] mod R[t];                  O HI guarda o resto da divisão (rem).
```

O resultado da divisão é armazenado em um registrador de 32 bits, assim como o resultado da mesma (reaproveitado das outras saídas da ALU, sem prejudicar as outras operações já existentes). 
Com o divisor colocado no opcode correto no MUX da ALU (111b) e com as entradas do divisor referentes aos registradores carregados na instrução (não podendo-se utilizar o $r0, que só possui o zero), a divisão pode ser concluída, como mostrado na figura 10.

![Figura 10: ALU modificada no circuito Monociclo.](/resources/figura10.jpg)

O resultado da divisão e o resto da mesma, quando é executada a divisão, já são passados para os pinos para mostrar a resposta, já que o DIV apenas faz a divisão e não salva em registrador nenhum.

Teste unitário realizado:
```assembly
lw $r1 0x0 $zero; r1 ← MEM(0) | 0x8c010000
lw $r2 0x0 $zero ; r2 ← MEM(0) | 0x8c020000
div $r2 $r1; 0x0041001a
```

#### Multiciclo

A divisão no MIPS multiciclo seguiu a mesma ideia de modificação de hardware no MIPS monociclo. 
Assim como no circuito do monociclo, foi adicionado na ALU um divisor de 32 bits, um comparador de 3 bits, um multiplexador e dois registradores de 32 bits que guardam o resto e o resultado da divisão quando é executada a divisão.
Esses registradores tem como enable de gravação quando é executada a divisão (saída 111(2) da Alu Control) para não perder o resultado da divisão, o qual é usado no MFLO.
Esses registradores são ativos sempre no modo high level do clock, por isso tem uma constante 1 na entrada de clock dos mesmos. 
A ALU modificada no multiciclo é apresentada na figura 11.

![Figura 11: ALU modificada no MIPS multiciclo](/resources/figura11.jpg)

No circuito multiciclo, assim como no monociclo, precisou-se modificar a ALU Control, já que dado o valor 0x12 de quando é executado o MFLO, entra na Alu Control e ao passar pelo circuito combinacional, tem como saída o valor 100(2), devido ao circuito combinacional de conversão mostrado na figura 12. A saída desse circuito é utilizada como seletor no MUX da ALU.

![Figura 12: ALU Control modificada no circuito do MIPS Multiciclo.](/resources/figura12.jpg)

O banco de registradores após a execução da divisão é mostrado na figura 13.

![Figura 13: Banco de registradores após a divisão.](/resources/figura13.jpg)


#### Pipeline

A ideia de implementação segue os moldes do MIPS multiciclo, tendo as seguintes modificações, vistas na figura 14, na ALU.

![Figura 14: Unidade Lógica e Aritmética modificada no circuito MIPS Pipeline.](/resources/figura14.jpg)

O seguinte exemplo foi executado como teste unitário:

```assembly
lw $s1 0x000 $zero; s1<-MEM(0) | 0x8c010000                    
lw $s2 0x000 $zero; s2<-MEM(0) | 0x8c020000
div $s1 $s2; 0x0232001A
```

As figuras 15 e 16 ilustram as duas memórias do MIPS Pipeline com MEM(0) = 3010 e o teste realizado. 

![Figura 15 e 16: Memória de instruções e memória de teste no meio do teste realizado.](/resources/figura1516.jpg)

### MFLO - Move from LO

#### Monociclo

Para pegar apenas o resultado da divisão, utilizou-se a saída output do divisor da ULA.
Essa saída é conectada na entrada 100b do MUX da ULA, e sua passagem por ele só é ativada quando a saída da Alu Control está em 1002 também. Essa saída só fica em 100(2) quando a entrada do ALU Control está em 0x12, que é o valor do campo funct referente à instrução de MFLO.
Como não tinha o MFLO na arquitetura básica do MIPS utilizada, teve que se adaptar a ALU Control, para que, quando entrasse 10010(2)(0x12) ela retornasse tivesse como saída o 100(2). 

Essa adaptação feita foi um comparador com 0x12 juntamente a um multiplexador que seleciona se utiliza o valor 0x12 para decodificar ou se decodifica uma outra entrada qualquer. O circuito combinacional de conversão adicionado na ALU Control é apresentado na figura 17.

![Figura 17: Circuito combinacional adicionado na primeira metade da ALU Control.](/resources/figura17.jpg)

Foi colocado, também, um registrador entre a saída do resto da divisão e a entrada do MUX. Esse registrador foi colocado com o propósito de guardar o resultado da divisão, que se perdia no ciclo de clock seguinte à execução do DIV. 
Para ter o valor da saída do output (saída do resto) sempre indo para o registrador, foi colocada a entrada de clock em constante 1, e alterado o modo do registrador para ter o trigger no nível alto de clock (1).  
O enable desse registrador é setado de acordo com a instrução vinda da aluOP, que no caso foi escolhi alterar o registrador apenas quando a instrução for uma de divisão, para não armazenar lixo nesse registrador em outras operações. 
Esse set é feito por meio de um comparador com 0x7 (divisão), e se o resultado for 1, o output pode ser gravado no registrador.

Teste unitário realizado:

```assembly
lw $r2 0x0 $zero; 0x8c020000
lw $r3 0x0 $zero; 0x8c030000
div $r3 $r0; 0x0041001a
mflo $r4; 0x00000812
```

#### Multiciclo

O MFLO no MIPS multiciclo seguiu a mesma ideia do circuito do MIPS monociclo. Como foi explicado na divisão, o circuito da ALU Control é responsável por gerar a seleção do MUX da ALU. 
Para o caso do MFLO, a saída do registrador de resultado entra no MUX na entrada 4, e o mesmo é passado para a ALU Out, a qual envia o valor que vem da saída do MUX da ALU para o registrador que se quer gravar, que fica no campo RD da instrução.

#### Pipeline
Assim como nos outros circuitos feitos, a ideia do MFLO se manteve, que é pegar a saída da unidade lógica e aritmética do processador e gravar no registrador informado no campo RD.
Nesse caso teve-se que se atentar aos ciclos, já que o DIV, por ser instrução do tipo R, sempre gravava no WB, e o MFLO já precisava desse resultado no segundo estágio. Mas o problema no nosso MIPS Pipeline não precisou ser considerado, daí o DIV em conjunto com o MFLO funcionou de maneira correta.

### Testes

Os testes para cada processador que estão junto aos .circ nas pastas são os mesmos para os três tipos de MIPS. 
O teste feito foi o seguinte:
```assembly
lw $s1 0x000 $zero; 0X8C110000
lw $s2 0x000 $zero; 0X8C120000 (No multiciclo foi 8c120008)
jr $s1; 0X02200008
div $s1 $s2; 0X0232001a
mflo $s4; 0X0000a012
```

O div está na posição que possui o valor 50, já que nos testes, esse foi o valor colocado em MEM(0).
