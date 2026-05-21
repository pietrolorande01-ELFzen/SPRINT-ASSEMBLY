#  EletroCore — Controle Otimizado de Eletropostos em Assembly

---

##  Integrantes

| Nome | RM |
|------|----|
| [ Ana Beatriz Berbel Marini ] | RM574176 |
| [ Gustavo Bonamico Piccoli ] | RM569984 |
| [ Marcelo Francisco Josafá Ribeiro Martins ] | RM573905 |
| [ Maria Eduarda Medeiros Lemos ] | RM574094 |
| [ Pietro Lorande da Silva ] | RM569125 |

---

##  Problema Encontrado

Sistemas de eletropostos modernos frequentemente operam com software de alto nível (Python, Java, Node.js) rodando sobre hardware genérico (PCs industriais, Raspberry Pi sem otimização).

**Software de alto nível** → linguagens como Python ou Java são mais fáceis de programar, mas precisam de um intermediário (interpretador ou compilador + runtime), o que adiciona custo computacional.

**Hardware genérico** → equipamentos feitos para uso geral, não especializados — acabam consumindo mais energia do que o necessário para tarefas simples.</br></br>

**Essa abordagem gera:**
</br></br>
**Consumo desnecessário de energia pelo processador** → realizando operações triviais com dezenas de instruções quando poucas bastariam.
Instruções são comandos básicos que o processador executa (como somar, mover dados, comparar valores).
</br></br>

**Baixa eficiência no processamento de dados críticos,** → como autenticação de usuário, leitura de sensores de tensão/corrente e controle de carga da bateria.
Dados críticos são aqueles que precisam ser processados rapidamente e com precisão, pois afetam diretamente o funcionamento do sistema.
</br></br>

**Desperdício de ciclos de CPU** →
Ciclo de CPU é uma unidade de tempo do processador; quanto mais ciclos uma tarefa usa, mais energia ela consome.
Um simples loop de leitura de sensor em C compilado sem otimização pode executar 40–80 instruções; em Assembly otimizado, o mesmo resultado é obtido com 6–10 instruções.
</br></br>

**Hardware superdimensionado** →
Significa usar um hardware mais potente do que o necessário, apenas para compensar um software ineficiente.
</br></br>

Em um país com mais de 100 mil eletropostos previstos até 2030, esse desperdício computacional acumulado representa um impacto energético real e evitável. </br>
---

##  Justificativa

A eficiência começa no nível mais baixo da pilha de software. Ao trabalhar diretamente com instruções de processador, é possível:

**Eliminar overhead de runtime**
 → Runtime é o ambiente que executa o programa (ex: JVM no Java). Ele consome memória e processamento extra.

**Eliminar garbage collector**
 → mecanismo automático que limpa memória em linguagens como Java — útil, mas consome CPU.

**Eliminar dependência de sistema operacional (SO)**
 → o SO gerencia processos, memória e hardware, mas adiciona latência e consumo.

**Reduzir o número de ciclos de clock necessários por operação**
 → Clock é o “ritmo” do processador — cada batida executa parte de uma instrução.

**Diminuir a frequência de clock (f)**
 → frequência é quantas operações por segundo o processador realiza.

**Relação de consumo energético:**
P ∝ f × C × V²

Onde:</br>
P = potência consumida </br>
f = frequência</br>
C = capacitância</br>
V = voltagem</br>

**Rodar em microcontroladores de ultrabaixo consumo** (dispositivos simples, usados em sistemas embarcados).

→ **mW (miliwatt) VS. W (watt):**</br>
→ 1000 mW = 1 W </br></br>
Ou seja, um sistema em mW consome até 1000x menos energia
---

##  Proposta de Solução

**EletroCore é um módulo de controle embarcado para eletropostos, com rotinas críticas implementadas em Assembly RISC-V (RV32I)**</br>

**Assembly** → linguagem de baixo nível que representa diretamente instruções do processador.</br>
**RISC-V** → arquitetura aberta de processadores, focada em simplicidade e eficiência.</br>
**RV32I** → conjunto base de instruções de 32 bits.</br>

Responsável por:

### Módulos Implementados

| Módulo | Função | Ganho estimado |
|--------|--------|----------------|
| `auth_module` | Autenticação de usuário via hash simples | ~70% menos ciclos vs C não otimizado |
| `sensor_read` | Leitura de tensão/corrente com ADC | ~60% menos instruções |
| `charge_ctrl` | Controle PWM de carga da bateria | Timing determinístico (zero latência de SO) |
| `energy_log` | Log de consumo em memória flash | Acesso direto a registradores de hardware |

**ADC (Analog-to-Digital Converter)** → converte sinais elétricos (analógicos) em números digitais.</br>
**PWM (Pulse Width Modulation)** → técnica que controla potência variando o tempo ligado/desligado de um sinal.</br>
**Memória flash** → memória não volátil (não perde dados ao desligar).</br>

### Fluxo do Sistema

```
[Usuário aproxima cartão RFID]
        ↓
[auth_module.asm] — verifica hash em registradores
        ↓
[charge_ctrl.asm] — inicia ciclo de carga via PWM
        ↓
[sensor_read.asm] — monitora tensão/corrente em loop
        ↓
[energy_log.asm] — registra consumo na flash
        ↓
[charge_ctrl.asm] — encerra carga ao atingir threshold

```
**RFID** → tecnologia de identificação por rádio. </br>
**Hash** → função matemática que transformar dados (ex: ID do cartão) em um código. </br>
**Registradores** → pequenas memórias extremamente rápidas dentro do processador. </br>
**Loop** → repetição contínua de instruções. </br>
**Threshold** → valor limite definido (ex: carga máxima de bateria). </br>

---

##  Arquitetura Utilizada

### Por que RISC-V?

Adotamos a arquitetura **RISC-V (RV32I)** pelos seguintes motivos técnicos:

| Característica | RISC-V | x86 (CISC) |
|----------------|--------|------------|
| Conjunto de instruções | Reduzido (~47 instruções base) | Extenso (~1500+) |
| Ciclos por instrução (CPI) | ~1 (pipeline simples) | 1–10+ |
| Consumo energético | Muito baixo | Alto |
| Complexidade do decodificador | Baixa | Alta |
| Adequação para embarcados |  Excelente |  Inadequado |

**CPI (Cycles Per Instruction)** → quantos ciclos uma instrução leva para executar.</br>
**CISC** → arquitetura com instruções complexas (ex: x86).</br>
**RISC** → arquitetura com instruções simples e rápidas.</br>

---

**Pipeline RISC-V (5 estágios):**
```
IF → ID → EX → MEM → WB

(Fetch) -> (Decode) -> (Execute) ->(Memory) -> (WriteBack)

```
**Pipeline** → técnica que divide execução em etapas, permitindo paralelismo</br>
**Fetch** → busca instrução.</br>
**Decode** → interpreta instrução.</br>
**Execute** → executa operação.</br>
**Memory** → acessa memória.</br>
**WriteBack** → grava resultado.</br>

Sem dependências de dados → CPI efetivo ≈ 1

**Hazard (conflito)** → quando uma instrução depende de outra. </br>
**Forwarding** → técnica para evitar atrasos nesses casos. </br>

### Hardware Alvo

**Microcontrolador**: SiFive HiFive1 Rev B
 → microcontrolador baseado em RISC-V

**Simulador de desenvolvimento**: RARS
→ ambiente para testar Assembly sem hardware físico

**Arduino Mega (AVR)**
→ alternativa baseada em outra arquitetura (AVR)
---

##  Trechos de Código Assembly

### 1. Leitura de Sensor de Tensão (sensor_read.asm)

```asm
# RISC-V RV32I — Leitura ADC para sensor de tensão
# Registradores: a0 = endereço base ADC, a1 = resultado

.text
.globl read_voltage

read_voltage:
    # Carrega endereço do registrador ADC_DATA
    lui     a0, 0x10016         # Base address do ADC (SiFive FE310)
    lw      a1, 0x00(a0)        # Lê valor bruto do ADC (12 bits)
    
    # Máscara para 12 bits (0xFFF)
    li      t0, 0xFFF
    and     a1, a1, t0          # Isola os 12 bits de resolução
    
    # Conversão: valor_ADC * 3300mV / 4096
    li      t1, 3300
    mul     a1, a1, t1          # a1 = raw * 3300
    li      t2, 4096
    div     a1, a1, t2          # a1 = tensão em mV
    
    ret                         # Retorna tensão em a0
```
---

→ Aqui ocorre leitura direta do hardware, sem bibliotecas intermediárias.

→ Conversão:
valor_ADC * 3300mV / 4096

→ Isso transforma o valor digital em tensão real.

---

### 2. Controle de Carga PWM (charge_ctrl.asm)

```asm
# RISC-V RV32I — Controle PWM para regulação de carga
# a0 = duty cycle desejado (0-255), t0-t3 = temporários

.text
.globl set_charge_pwm

set_charge_pwm:
    lui     t0, 0x10015         # Base address PWM0 (FE310)
    
    # Configura período PWM (prescaler = 1, count = 256)
    li      t1, 0x0F            # pwmcfg: enable, zerocmp
    sw      t1, 0x00(t0)        # Escreve configuração
    
    li      t1, 256
    sw      t1, 0x10(t0)        # pwmcount (período)
    
    # Duty cycle no comparador 1
    sw      a0, 0x1C(t0)        # pwmcmp1 = duty cycle
    
    ret
```
→ Controla diretamente o sinal elétrico da carga.

→ Duty cycle:
→ porcentagem do tempo que o sinal fica ligado (ex: 50% = metade do tempo ligado).


---

### 3. Hash Simples para Autenticação (auth_module.asm)

```asm
# RISC-V RV32I — Hash FNV-1a 32-bit para autenticação RFID
# a0 = ponteiro para string UID, a1 = tamanho
# Retorna: a0 = hash de 32 bits

.text
.globl fnv_hash

fnv_hash:
    li      t0, 0x811C9DC5      # FNV offset basis
    li      t1, 0x01000193      # FNV prime
    mv      t2, a0              # ponteiro atual
    add     t3, a0, a1          # ponteiro fim
    
hash_loop:
    bge     t2, t3, hash_done   # se chegou ao fim, encerra
    lbu     t4, 0(t2)           # carrega byte atual (unsigned)
    xor     t0, t0, t4          # XOR com hash atual
    mul     t0, t0, t1          # multiplica pelo primo FNV
    addi    t2, t2, 1           # avança ponteiro
    j       hash_loop
    
hash_done:
    mv      a0, t0              # retorna hash em a0
    ret
```
→ FNV-1a: algoritmo de hash leve e rápido

→ Ele funciona assim:

pega cada byte
aplica XOR
multiplica por um número primo

→ Resultado: um identificador único do usuário

---

##  Comparação: Assembly vs C (Alto Nível)

| Métrica | C (gcc -O0) | C (gcc -O2) | Assembly RISC-V |
|---------|------------|------------|-----------------|
| Instruções (leitura sensor) | ~45 | ~18 | **10** |
| Ciclos de clock (auth) | ~320 | ~140 | **52** |
| Consumo estimado (operação completa) | ~850 µJ | ~370 µJ | **~120 µJ** |
| Latência determinística |  Não |  Não |  Sim |
| Dependência de SO/runtime |  Sim |  Sim |  Não |

> Valores estimados com base em perfis energéticos de microcontroladores RISC-V documentados pela SiFive e RISC-V Foundation.

**gcc -O0** → sem otimização
**gcc -O2** → otimização intermediária

**Latência determinística** →
sempre leva o mesmo tempo para executar

 → Isso é essencial para sistemas críticos

---

##  Impactos Esperados

### Energéticos

→ Redução de **~85% nos ciclos de CPU** nas operações críticas do eletroposto
→ Possibilidade de rodar em hardware com consumo de **50–150 mW** vs **5–15 W** de soluções genéricas
→ Extensão da vida útil da bateria de backup do eletroposto

Operação em faixa de mW (miliwatts)
Comparação prática:
→ 100 mW = consumo de um microcontrolador
→ 10 W = consumo de um mini computador

### Financeiros (em escala)

→ 100.000 eletropostos × (5W economizados) × 8.760 h/ano = **~4,38 GWh/ano economizados**
→ Equivale a ~R$ 2,6 milhões/ano em energia (considerando tarifa média industrial)

→ **GWh (Gigawatt-hora)**:
1 GWh = 1 bilhão de watts por hora

→ Isso mostra impacto em escala nacional

### Técnicos

**Timing determinístico** → sem preempção de SO, ciclos de controle têm latência garantida
**Menor footprint** → código Assembly ocupa ~10x menos memória flash que equivalente em C
**Maior confiabilidade** → menos camadas de abstração = menos pontos de falha

**Footprint** → quantidade de memória usada
Menor código → menos memória → menor custo

**Confiabilidade** → menos camadas → menos chance de erro

---

##  Relação com Sustentabilidade e Energias Renováveis

A sustentabilidade neste projeto opera em dois níveis:

**1. Eficiência do próprio eletroposto:**
Ao reduzir o consumo computacional do sistema de controle, sobra mais energia da fonte renovável (solar/eólica) para carregar os veículos — a energia não é desperdiçada em processamento ineficiente.

Resumo em linha:
->  **menos energia usada no sistema → mais energia disponível para veículos**

**2. Escalabilidade do impacto:**
Código otimizado permite que o mesmo hardware dure mais tempo sem substituição, reduzindo o impacto de fabricação e descarte de componentes eletrônicos (e-waste).

Resumo em Linha:
**menos troca de hardware → menos lixo eletrônico (e-waste)**

**3. Compatibilidade com geração intermitente:**
Microcontroladores de baixo consumo podem operar com painéis solares pequenos e baterias de ciclo de vida estendido, viabilizando **eletropostos off-grid** em locais sem acesso à rede elétrica convencional.

Resumo em Linha:

→ **energia solar/eólica não é constante**

→ **sistemas eficientes funcionam melhor com essa variação**

```
[Painel Solar] → [Bateria LiFePO4] → [EletroCore (RISC-V, ~100mW)] → [Controle de Carga EV]
                                              ↑
                              Processamento em Assembly
                              = menos energia desperdiçada
                              = mais energia para o veículo
```

---

## Tecnologias e Ferramentas a se Utilizar

- **Linguagem**: Assembly RISC-V (RV32I / RV32IMAC)
- **Simulador**: [RARS — RISC-V Assembler and Runtime Simulator](https://github.com/TheThirdOne/rars)
- **Hardware alvo**: SiFive HiFive1 Rev B / Arduino Mega (protótipo AVR)

- **Conceitos aplicados**:
  - Pipeline de 5 estágios (IF, ID, EX, MEM, WB)
  - Hazard detection e data forwarding
  - Cache L1 (instruções e dados)
  - Consumo energético por instrução (Energy per Instruction — EPI)
  - PWM via registradores de hardware

---

##  Links

-  **Vídeo Pitch**: [YouTube — não listado](https://youtube.com)
-  **Repositório GitHub**: https://github.com/pietrolorande01-ELFzen/SPRINT-ASSEMBLY

---

*Projeto desenvolvido para a disciplina de Arquitetura de Computadores* — **FIAP, 2026.**
**Time Lumos Maxima**
