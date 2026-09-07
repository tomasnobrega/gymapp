# Treino — tracker de academia

App de treino em um único arquivo (`index.html`), sem dependências, feito pra usar no celular na academia.

## O que faz

- **Treinos A e B** (corpo inteiro) com aquecimento, séries × reps e um **finisher de cardio com peso** no fim (estilo Hyrox / WOD).
- **Ciclo de 6 semanas** com periodização embutida: a cada semana o app muda séries, faixa de reps, descanso e técnica pra manter o estímulo crescente. No fim do ciclo, você manda treinos novos e inicia outro ciclo.
- **Registro de cargas**: em cada exercício aparece o que você fez da última vez (kg × reps por série) e os campos já vêm preenchidos com a carga anterior.
- **Sugestão de carga**: quando a faixa de reps muda de uma semana pra outra, o app calcula a carga inicial pelo 1RM estimado da última sessão e já preenche. Na mesma faixa, se bateu o topo das reps em todas as séries, sugere subir a carga (toque pra aplicar). No deload, sugere −30 %.
- **Timer de descanso**: o ✓ de cada série marca como feita e dispara o timer (padrão 90s, com aviso sonoro e vibração).
- **Histórico** por sessão, com volume total e detalhe das séries.
- **Progresso** por exercício: melhor carga, 1RM estimado (Epley) e gráfico da evolução.
- **Exportar CSV** (uma linha por série) e **backup/restauração em JSON**.
- Tudo fica salvo **localmente no navegador** (localStorage). O rascunho do treino em andamento também persiste se fechar a aba.

## O ciclo de 6 semanas

| Sem | Fase | Compostos | Isolados | Descanso | Técnica |
|----:|------|-----------|----------|---------:|---------|
| 1 | Base | 3 × 10–12 | 3 × 12–15 | 60s | Tempo controlado (3 s na descida) |
| 2 | Base + | 3 × 10–12 | 3 × 12–15 | 75s | Última série AMRAP |
| 3 | Volume | 4 × 8–10 | 3 × 12–15 | 90s | Série extra + pausa de 1 s |
| 4 | Força | 4 × 6–8 | 4 × 10–12 | 120s | Rest-pause na última série |
| 5 | Pico | 1 × 4–6 + 3 × 6–8 | 4 × 8–10 | 180s | Top set + back-off (−10 %) |
| 6 | Deload | 2 × 8–10 | 2 × 12–15 | 90s | −30 % de carga, longe da falha |

Prancha e afins ganham +5 a +15 s de alvo ao longo do ciclo. O finisher também progride por semana (mais rounds/tempo, depois mais carga; leve no deload).

A semana é calculada pela data de início do ciclo (Ajustes → Ciclo). Dá pra trocar a semana manualmente no card do ciclo, por exemplo se pulou uma semana. As fases ficam no bloco `PHASES` do `index.html`.

## Como usar

1. Abra `index.html` no navegador do celular (ou hospede no GitHub Pages e adicione à tela inicial).
2. Escolha A ou B no topo. O app sugere o próximo com base no último treino salvo.
3. Preencha kg e reps; toque no ✓ ao terminar cada série (se deixar em branco, ele assume os valores da última vez).
4. No finisher, anote o tempo (ou rounds) e as cargas usadas.
5. **Concluir treino** salva a sessão no histórico.

## Exportar pro R

Em **Ajustes → Exportar CSV**. Colunas: `data, treino, ciclo, semana, fase, exercicio, serie, kg, reps, segundos, score, cargas, obs_exercicio, obs_treino`.

```r
library(data.table)
dt <- fread("treinos_2026-09-07.csv")
dt[!is.na(kg), .(melhor_kg = max(kg), e1rm = max(kg * (1 + reps / 30))), by = .(exercicio, data)]
```

## Alterar os treinos

Os treinos estão no bloco `WORKOUTS`, no início do `<script>` em `index.html`. Cada exercício aceita:

| campo  | exemplo               | significado                                         |
|--------|-----------------------|-----------------------------------------------------|
| `name` | `'Supino reto'`       | nome (vira a chave do histórico)                    |
| `sets` | `3`                   | número de séries                                    |
| `reps` | `[8, 10]`             | faixa de reps (`[10, 10]` mostra "3 × 10")          |
| `time` | `[30, 45]`            | exercício por tempo, em segundos (prancha)          |
| `alt`  | `['Agachamento', 'Leg press']` | variantes; cada uma tem histórico próprio  |
| `each` | `'cada perna'`        | só rótulo                                           |
| `iso`  | `true`                | isolado: usa a faixa de reps mais alta do ciclo     |
| `cat`  | `'fixed'`             | não muda com a semana (mantém sets/reps)            |

O `finisher` tem `format`, `cap`, `movements`, `score` (`'time'` ou `'rounds'`), `loads` (texto de ajuda pro campo de cargas) e `weeks` (formato em cada uma das 6 semanas).

Mudar a estrutura dos treinos não apaga o histórico: os registros são ligados pelo nome do exercício.
