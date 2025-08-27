# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: __9__ syscalls
- ex1b_write: __7__ syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**
```
Por que enquanto o write() é uma função que chama a syscall diretamente e conversa diretamente com o Kernel, o printf() funciona mais como uma mascara que facilita e abstrai o funcionamento da syscall. 

Desta forma, o printf() possui comportamentos "por baixo dos panos" diferentes do esperado, chamando mais syscalls.
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
Para um desenvolvedor que esta trabalhando com um programa de performance crítica e que precisa de controle a respeito do envio de dados, o método mais previsivel é o write(), que utiliza uma syscall por chamada. 
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: __3__
- Bytes lidos: __127__

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
O file descriptor usado foi o 3, isto ocorreu pois os outros fd's já estavam ocupados para variaveis do inicio do programa, como entrada, saida e erro.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Eu sei que o arquivo foi lido completamente pois utilizamos um BUFFER_SIZE maior que os bytes do arquivo, assim, ele leu até o final do arquivo. E como não ocorreu erro, então sei que foi lido completamente
```

**3. Por que verificar retorno de cada syscall?**

```
Para que não ocorra erros de execuções que terminem a atividade do processo, verificamos se todas syscalls foram corretamente executadas para podermos prosseguir.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: _____ (esperado: 25)
- Caracteres: _____
- Chamadas read(): _____
- Tempo: _____ segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |       83        |  0.001360 |
| 64          |       22        |  0.000486 |
| 256         |        7        |  0.000254 |
| 1024        |        3        |  0.000179 |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
O quão maior for o tamanho do buffer, menos syscalls serão necessárias para poder ler o arquivo, visto que a quantidade que uma syscall lê se torna maior.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**
```
Não, todas as chamadas que foram as ultimas a ser chamadas não retornaram buffer_size bytes, isto por que elas leram apenas o que sobrou do arquivo, desta forma esse numero de bytes variava. 
```

**3. Qual é a relação entre syscalls e performance?**

```
Um numero maior de syscalls diminui a velocidade de um processo, isto por que, ao ocorrer uma syscall, o sistema deve interromper o processo e o Kernel toma controle da CPU, este é uma tarefa demorada e acaba atrasando o tempo do processo.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: __1364__
- Operações: __6__
- Tempo: ___0.000378__ segundos
- Throughput: __3523.89___ KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para termos certeza que todo conteudo do origem foi para o destino 
```

**2. Que flags são essenciais no open() do destino?**

```
O_WRONLY | O_CREAT | O_TRUNC
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim, visto que a quantidade que sera escrito vai depender da quantidade de bytes lidos, assim, se ler da origem 20 bytes, entao vai escrever 20 bytes e as syscalls read e write vao ter o mesmo tamanho
```

**4. Como você saberia se o disco ficou cheio?**

```
A syscall write() retornaria -1 ao tentar escrever no arquivo
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
File descriptors ficam abertos, consumindo recursos e dados podem não ser gravados no disco corretamente.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls demonstrams esta transisção pois ela que retira o processo de execução e coloca o kernel em controle para executar operações que o usuário não possui permissão. Assim, as syscalls transicionam pelas diferentes hierarquias de processos
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os file descriptors identificam arquivos abertos, assim, o kernel pode controlar estes sem contextos de hardware nem espaçamento de memória. Ou seja, FD's permitem o acesso e manipulações de arquivos de forma consistente pelo programa
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
O tamanho do buffer e a perfomance são diretamente ligadas, de forma que o quão maior for o tamanho do bufffer maior será a perfomance. Isto pode ser explicado ao se colocar em perspectiva que com maior memoória para trabalhar, programas precisam utilizar menos syscalls, assim, diminuindo o tempo de execução e aumentando a performance.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** _____

**Por que você acha que foi mais rápido?**

```
O cp do sistema foi mais rápido, acredito que o cp, por ser interno do sistema, tem uma conexão mais direta com o kernel, diminuindo o tempo de syscalls
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
