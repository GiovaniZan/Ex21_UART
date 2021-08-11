# Exercício 21

1 Configure a UART0 do TM4C1294 para taxa de transmissão de 300 bps, 8 bits de dados, paridade par e 2 stop bits

2 Aguarde a recepção do caracter ‘\r’ pela UART0

3 Após a recepção do caracter ‘\r’, responda transmistindo a mensagem “Sistemas Microcontrolados\r\n” pela UART0



Este código foi implementado no processador tm4c129encpdt.

Considerando 16MHz de clock para o UART, usando um divisor de 16 temos e uma taxa de transmissão de 300bps:


$$
BRD = \frac{16000000}{16 \times 300} = \frac{10000}{3}=3333,33333...
$$
 Resultando em uma parte inteira $IBRD = 3333$ e uma parte fracionária
$$
FBRD = 0,3333333... \times 2^6+0.5 = 21,833333 \approx 22
$$
A condição de 8bits, paridade par e 2 stop bits e com FIFO desabilitado produzem o valor 0x6E, a ser colocado no registrador de offset #UART_LCRH  (Uart Line Control - página 1310 do tm4c129encpdt.pdf)

    ; UART_config: configura a UART desejada
    ; R0 = endereço base da UART desejada
    ; Destrói: R1
    UART_config:
            LDR R1, [R0, #UART_CTL]
            BIC R1, #0x01 ; desabilita UART (bit UARTEN = 0)
            STR R1, [R0, #UART_CTL]
            
        // clock = 16MHz, ClkDiv=16
        // BRD = 3333.33333
        // IBRD = 3333
        // FBRD = 22
        MOV R1, #3333
        STR R1, [R0, #UART_IBRD]
        MOV R1, #22
        STR R1, [R0, #UART_FBRD]
         // 8 bits, 2 stop bits, even parity, FIFOs disabled, no interrupts
        MOV R1, #0x6E
        STR R1, [R0, #UART_LCRH]
            ; clock source = system clock
        MOV R1, #0x00
        STR R1, [R0, #UART_CC]
        
        LDR R1, [R0, #UART_CTL]
        ORR R1, #0x01 ; habilita UART (bit UARTEN = 1)
        STR R1, [R0, #UART_CTL]
    
        BX LR`
    

​        Armazenei a string em uma posição de memória ROM e reservei espaço em RAM, o que se mostrou desnecessário para a tarefa.

O loop principal (loop) verifica a chegada de caracteres e atua quando o caractere corresponder a 0x0A, equivalente a '\r'. A partir do reconhecimento deste caractere, passa a executar a escrita dos caracteres no registrador de transmissão da UART, até que se encontre o caractere 0x00 ou '\0', que marca o fim da string.

Por medida de segurança, também se testa um comprimento máximo - para evitar um loop infinito (também se mostrou desnecessário).



## Questão:

Quanto tempo dura a transmissão de um caractere com esta configuração?

Res. Dado que a  taxa de transmissão corresponde a 300 bits por segundo, e que cada caractere possui 8 bits mais 2 bits de parada, mais um bit de paridade, mais um bit de início, em um total de 12 bits, temos  cada caractere ocupando um intervalo de tempo de 0,04 segundos.

A  mensagem inteira (de 25 caracteres mais '\n' e '\r') de 27 caracteres demora um tempo de 1,08 segundos para ser completamente transmitida.

    