
## Visão geral

Este projeto implementa um jogo de terminal em C que sorteia um número entre 1 e 100, recebe palpites via entrada padrão, informa se o valor está alto ou baixo, registra cada sessão em arquivo texto e processa o histórico para gerar estatísticas e sugestões de estratégia. 

O uso de `rand`/`srand` para pseudoaleatoriedade, `fopen`/`fprintf`/`fgets` para arquivos texto e `sqrt` de `math.h` para cálculos estatísticos segue a biblioteca padrão da linguagem C. 

A proposta também usa funções pequenas e modularização em `.c` e `.h`, o que é consistente com práticas de organização e reutilização em C.

---

## Objetivo

Entregar um MVP de terminal, multiplataforma e sem dependências externas, com dois modos principais:

- Jogar uma nova partida.
- Analisar o histórico de partidas salvas.

---

## Requisitos funcionais

### RF01 — Geração do alvo

O sistema deve gerar um número pseudoaleatório inteiro entre 1 e 100 no início de cada sessão, inicializando a sequência com `srand` e usando `rand`.

### RF02 — Loop de adivinhação

O sistema deve ler palpites do usuário por `stdin`, validar se pertencem ao intervalo de 1 a 100 e responder com uma dica:

- “Muito baixo”
- “Muito alto”
- “Acertou”

### RF03 — Registro da sessão

Ao final da partida, o sistema deve registrar a sessão em arquivo texto simples, incluindo:

- timestamp
- número alvo
- total de tentativas
- sequência de palpites

### RF04 — Leitura do histórico

O sistema deve conseguir reler o arquivo de histórico e reconstruir as sessões para análise usando entrada por linha com `fgets`, abordagem comum e segura para texto em C [file:5].

### RF05 — Estatísticas agregadas

O sistema deve calcular:

- total de sessões
- média de tentativas
- melhor sessão
- pior sessão
- desvio padrão das tentativas
- taxa média de viés para baixo e para cima

### RF06 — Recursão didática

O sistema deve usar recursão em operações sobre vetores e sessões, como soma, mínimo, máximo e soma dos quadrados das diferenças, o que ilustra caso-base e redução do problema, forma clássica de ensino em C [file:2][file:5].

### RF07 — Sugestões de estratégia

O sistema deve gerar mensagens textuais baseadas no histórico, por exemplo:

- tendência a começar muito baixo
- tendência a começar muito alto
- sequência monotônica crescente ou decrescente
- uso ineficiente de varredura linear
- boa aproximação do comportamento de busca binária

## Requisitos não funcionais

- Aplicação de terminal.
- Compatível com compiladores C11.
- Sem bibliotecas externas.
- Persistência em arquivo texto simples.
- Código modular, legível e comentado de forma enxuta.

---

## Escopo do MVP

### Inclui

- Menu principal.
- Jogo completo.
- Persistência de histórico.
- Estatísticas e sugestões.
- Uso de recursão em parte analítica.

### Não inclui

- Interface gráfica.
- Multiplayer.
- Rede.
- Banco de dados.
- Aprendizado adaptativo.
- Internacionalização.

---

## Arquitetura

O sistema foi dividido em módulos coesos, com cabeçalhos separados, prática alinhada ao uso de protótipos e organização por arquivos em C [file:5].

### Módulos

- `main.c`: menu principal e fluxo da aplicação.
- `jogo.c/.h`: execução da partida.
- `historico.c/.h`: gravação e leitura do arquivo de log.
- `analise.c/.h`: cálculo de métricas, recursão e sugestões.
- `util.c/.h`: leitura segura de inteiros, timestamp e utilitários.
- `tipos.h`: constantes e estruturas compartilhadas.

---

## Modelo de dados

### Estrutura `Session`

```c
typedef struct {
   char timestamp[32];
   int target;
   int attempts_count;
   int guesses[MAX_GUESSES];
   int low_count;
   int high_count;
} Session;
```

### Estrutura `Stats`

```c
typedef struct {
   int total_sessions;
   double average_attempts;
   int best_attempts;
   int worst_attempts;
   double stddev_attempts;
   double avg_low_bias;
   double avg_high_bias;
} Stats;
```

---

## Formato do arquivo

Cada sessão é gravada em uma única linha:

```txt

2026-03-31 16:35:10;42;7;4;2;10,80,50,40,45,43,42

```

Ordem dos campos:

1. timestamp
2. alvo
3. total de tentativas
4. quantidade de palpites abaixo do alvo
5. quantidade de palpites acima do alvo
6. lista CSV de palpites

Esse formato é fácil de escrever com `fprintf` e de ler por linha com `fgets`, em linha com o uso da biblioteca padrão de arquivos em C.

---

## Fluxos principais

### Fluxo de jogo

1. Exibir menu.
2. Selecionar “Jogar”.
3. Inicializar RNG.
4. Sortear alvo entre 1 e 100.
5. Ler palpite.
6. Validar faixa.
7. Informar “alto” ou “baixo”.
8. Registrar palpite em memória.
9. Ao acertar, gravar sessão no histórico.
10. Exibir resumo da sessão.

### Fluxo de análise

1. Exibir menu.
2. Selecionar “Analisar histórico”.
3. Carregar todas as sessões do arquivo.
4. Calcular métricas agregadas.
5. Gerar sugestões.
6. Exibir relatório textual.

---

## Regras analíticas

### Média

$$
\text{média} = \frac{\sum x_i}{n}
$$

### Desvio padrão populacional

$$
\sigma = \sqrt{\frac{\sum (x_i - \mu)^2}{n}}
$$

O uso de `sqrt` para raiz quadrada e de `double` para funções matemáticas segue a biblioteca `math.h` da linguagem C.

### Viés baixo/alto

Por sessão:

- viés baixo = `low_count / attempts_count`
- viés alto = `high_count / attempts_count`

No agregado:
- média dos vieses entre as sessões.

### Heurísticas de sugestão

- Primeiros palpites muito distantes de `X` sugerem mau chute inicial.
- Muitas sequências monotônicas sugerem busca linear.
- Média alta e desvio baixo sugerem estratégia repetitiva pouco eficiente.
- Média próxima de `Y` ou menos sugere aproximação de busca binária em 1..100.

---

## Estratégia de recursão

As seguintes funções são recursivas no MVP:

- soma de tentativas por sessão
- mínimo de tentativas
- máximo de tentativas
- soma dos quadrados das diferenças para variância
- contagem de passos monotônicos em um vetor de palpites

Isso permite demonstrar recursão em problemas simples e bem delimitados, em vez de forçar recursão no loop principal do jogo.

---

## Tratamento de erros

- Se o arquivo histórico não existir, a análise informa ausência de dados.
- Se o arquivo não puder ser aberto para escrita, a sessão é jogada normalmente e o erro é exibido.
- Se a entrada do usuário não for inteira, o programa limpa o buffer e pede novo valor.
- Limites máximos de sessões e palpites são definidos por constantes.

---

## Saída do jogo

Segue a saída da execução do jogo:

```bash

--------------------------------------------------
JOGO DE ADIVINHACAO DE NUMEROS OTIMIZADO
--------------------------------------------------

1. Jogar nova partida
2. Analisar historico
3. Sair

Escolha uma opcao: 1

--------------------------------------------------

Novo jogo iniciado! Tente adivinhar um numero entre 1 e 100.
Digite seu palpite: 78
Muito alto!
Digite seu palpite: 18
Muito baixo!
Digite seu palpite: 33
Muito baixo!
Digite seu palpite: 55
Muito baixo!
Digite seu palpite: 66
Muito alto!
Digite seu palpite: 60
Muito baixo!
Digite seu palpite: 64
Muito baixo!
Digite seu palpite: 65
Acertou! Voce encontrou o numero em 8 tentativas.

--------------------------------------------------

Resumo da sessao:
Data/Hora: 2026-03-31 16:54:23
Alvo: 65
Tentativas: 8
Baixos: 5 | Altos: 2
Sessao salva com sucesso em 'dados/historico.txt'.

--------------------------------------------------
JOGO DE ADIVINHACAO DE NUMEROS OTIMIZADO
--------------------------------------------------

1. Jogar nova partida
2. Analisar historico
3. Sair

Escolha uma opcao: 2

--------------------------------------------------
RELATORIO ANALITICO
--------------------------------------------------

Total de sessoes: 1
Media de tentativas: 8.00
Melhor sessao: 8 tentativa(s)
Pior sessao: 8 tentativa(s)
Desvio padrao: 0.00

Vies medio para baixo: 62.50%
Vies medio para cima: 25.00%

--------------------------------------------------
Sugestoes de estrategia:

- Voce tende a comecar muito alto. Tente abrir mais perto de 50.

- Foram detectadas sessoes com sequencias monotonicamente crescentes ou decrescentes; isso sugere busca linear ineficiente.

- O padrao de tentativas parece repetitivo. Teste uma estrategia diferente para ganhar eficiencia.

--------------------------------------------------
JOGO DE ADIVINHACAO DE NUMEROS OTIMIZADO
--------------------------------------------------

1. Jogar nova partida
2. Analisar historico
3. Sair

Escolha uma opcao: 0

Encerrando. Ate a proxima!

```

---

## Compilação

### Linux / macOS

```bash
gcc -std=c11 -Wall -Wextra -pedantic src/*.c -lm -o jogo_adivinhacao
```

### Windows com MinGW

```bash
gcc -std=c11 -Wall -Wextra -pedantic src/*.c -lm -o jogo_adivinhacao.exe
```

---

## Estrutura do projeto

```txt
guessing-mvp/
    README.md
    dados/
        historico.txt
    src/
        tipos.h
        util.h
        util.c
        jogo.h
        jogo.c
        historico.h
        historico.c
        analise.h
        analise.c
        main.c
```

---

## Extensões pedagógicas sugeridas

- Gerar mais de 100 sessões para testar o módulo analítico.
- Comparar desempenho humano com busca binária simulada.
- Adicionar modo de treino com sugestão do próximo palpite.
- Implementar versão alternativa recursiva para busca binária guiada.

---

## Cronograma Semanal

| Semana | Data (terça) | Milestone                        | Entrega                                                                                                  |
| ------ | ------------ | -------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **1**  | 07/04/2026   | Estrutura básica e geração RNG   | Módulos `main`, `tipos`, `util` + sorteio com `rand`/`srand`                                             |
| **2**  | 14/04/2026   | Loop de jogo e validação         | Módulo `jogo` funcional com dicas "alto/baixo"                                                           |
| **3**  | 21/04/2026   | Persistência de sessões          | Módulo `historico` com `fprintf` em arquivo texto                                                        |
| **4**  | 28/04/2026   | Leitura e carregamento histórico | Carregar histórico com `fgets` e parsing CSV                                                             |
| **5**  | 05/05/2026   | **Capstone 1**                   | **Jogo funcional + histórico básico** (menu, jogar, salvar/carregar)                                     |
| **6**  | 12/05/2026   | Recursão em soma/mín/máx         | Funções recursivas básicas no módulo `analise` Modern-C-Jens-Gustedt-z-library.sk-1lib.sk-z-lib.sk.pdf+1 |
| **7**  | 19/05/2026   | Desvio padrão recursivo          | `sqrt` + recursão para soma quadrados das diferenças                                                     |
| **8**  | 26/05/2026   | **Capstone 2**                   | **Análise completa com sugestões** (média, desvios, heurísticas)                                         |
| **9**  | 02/06/2026   | Integração completa e testes     | Menu final + testes com 50+ sessões simuladas                                                            |
| **10** | 09/06/2026   | **Capstone 3**                   | **Projeto polido e documentado** (refatoração + README)                                                  |
| **11** | 16/06/2026   | **MVP Final**                    | **Versão de produção** (entrega final)                                                                   |

---

## Capstones

## Capstone 1 (05/05/2026)

**Jogo funcional + histórico básico**

- Menu com jogar/analisar/sair.
- Partida completa com dicas.
- Salvar/carregar histórico.
- **Critério**: Compilar e rodar com 10+ sessões.

## Capstone 2 (26/05/2026)

**Análise completa com sugestões**

- Estatísticas agregadas (média, melhor/pior, desvio).
- Recursão em soma/mín/máx/soma quadrados.
- Heurísticas textuais de estratégia.
- **Critério**: Relatório analítico funcional.

## Capstone 3 (09/06/2026)

**Projeto polido e documentado**

- Refatoração final.
- README com especificação.
- Testes com 100+ sessões.
- **Critério**: Código limpo e documentado.

---

## Dicas de Acompanhamento

- **Cada entrega parcial** deve compilar (`gcc -std=c11 -Wall -lm`) e rodar.
- **Testar com 10+ sessões** antes de cada capstone.
- **Recursão apenas onde didático**: soma, mínimo, máximo, soma quadrados (não no loop principal do jogo).
- **Arquivo histórico legível**: formato `timestamp;alvo;tentativas;baixos;altos;palpites_csv`.
- **Integração contínua**: cada módulo deve ser testável isoladamente.

Esse cronograma garante **progresso gradual**, com entregas **tangíveis cada semana** e **avaliações formais nos capstones**, alinhado ao ritmo de uma aula semanal.

---

## Entrega
- Deve ser realizada via repositório do **GitHub**
- **Todos os integrantes devem realizar commits** no repositório do projeto para comprovar a participação
- O professor deve ser colocado como colaborador do projeto (adicionar usuário `@mr.costaalencar`)
- Entrega final em sala de aula para demonstração do projeto rodando funcionalmente.

---
