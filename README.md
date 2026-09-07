# Treino — tracker de academia

App de treino em um único arquivo (`index.html`), sem dependências, feito pra usar no celular na academia.

## O que faz

- **Treinos A e B** (corpo inteiro) com aquecimento, séries × reps e um **finisher de cardio com peso** no fim (estilo Hyrox / WOD).
- **Registro de cargas**: em cada exercício aparece o que você fez da última vez (kg × reps por série) e os campos já vêm preenchidos com a carga anterior.
- **Sugestão de progressão**: se bateu o topo da faixa de reps em todas as séries, o app sugere subir a carga (incremento configurável). Toque na sugestão pra aplicar.
- **Timer de descanso**: o ✓ de cada série marca como feita e dispara o timer (padrão 90s, com aviso sonoro e vibração).
- **Histórico** por sessão, com volume total e detalhe das séries.
- **Progresso** por exercício: melhor carga, 1RM estimado (Epley) e gráfico da evolução.
- **Exportar CSV** (uma linha por série) e **backup/restauração em JSON**.
- Tudo fica salvo **localmente no navegador** (localStorage). O rascunho do treino em andamento também persiste se fechar a aba.

## Como usar

1. Abra `index.html` no navegador do celular (ou hospede no GitHub Pages e adicione à tela inicial).
2. Escolha A ou B no topo. O app sugere o próximo com base no último treino salvo.
3. Preencha kg e reps; toque no ✓ ao terminar cada série (se deixar em branco, ele assume os valores da última vez).
4. No finisher, anote o tempo (ou rounds) e as cargas usadas.
5. **Concluir treino** salva a sessão no histórico.

## Exportar pro R

Em **Ajustes → Exportar CSV**. Colunas: `data, treino, exercicio, serie, kg, reps, segundos, score, cargas, obs_exercicio, obs_treino`.

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

O `finisher` tem `format`, `cap`, `movements`, `score` (`'time'` ou `'rounds'`) e `loads` (texto de ajuda pro campo de cargas).

Mudar a estrutura dos treinos não apaga o histórico: os registros são ligados pelo nome do exercício.
