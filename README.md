# Caderno de Tiro

**→ [caderno-de-tiro.github.io](https://caderno-de-tiro.github.io/)**

Três ferramentas para jogar DDTank Classic com número na mão em vez de chute:
uma calculadora de tiro que resolve ângulo e força pela física do jogo, e duas
cadernetas para guardar o que você já mediu em partida.

Site estático puro — quatro arquivos HTML, sem build, sem servidor, sem
dependência externa, sem uma única requisição de rede. Tudo o que você cadastra
fica no `localStorage` do seu navegador, e nada sai de lá.

## As páginas

### `index.html` — Menu principal

Atalhos para as três ferramentas e o **backup único** — um `.json` só com o
conteúdo das três páginas (ver [Seus dados](#seus-dados)).

### `calculadora.html` — Calculadora de tiro

Você informa a distância e o vento; ela devolve o ângulo e a força.

- **Régua do jogo**: a mesma régua de 0 a 10 que aparece embaixo da tela. Clica
  onde você está, clica onde está o inimigo, e a distância sai da diferença.
- **Distância** de 1 a 20 (com passo de 0,5) e **vento** de 0 a 6 (passo 0,1),
  a favor ou contra.
- **Ângulos**: as abas de 20° / 30° / 50° / 65°, o tiro *full* (força 95) e
  ajuste fino de ±1°.
- **Três pontas**: mostra onde caem as três bolas quando você está com a arma
  de três pontas.
- **Valores medidos**: corrigiu uma célula da tabela? Aquele número passa a ser
  a verdade para a distância. O vento continua vindo do modelo — a correção que
  ele calcula é aplicada por cima do seu valor, em vez de descartá-lo.
- Atalhos: `0-9` digita a distância, `↑ ↓` distância ±0,5, `← →` vento ±0,5,
  `espaço` inverte o vento, `Tab` troca o ângulo, `Esc` zera só o vento.

### `referencias.html` — Referências

Os ângulos e forças que funcionam em cada instância, organizados por **instância
→ stage → tiro**. As instâncias do jogo (Formigueiro, Castelo Sombrio, Covil do
Dragão, Olimpíadas Boom e outras) já vêm criadas, cada uma com seu ícone — você
só pendura o stage nelas, sem precisar cadastrar a instância. Elas não somem:
o botão nelas é **limpar**, que tira o conteúdo e mantém a instância na lista.
Instâncias que você criar, com outro nome, são suas e têm **excluir**. As que
ainda não têm tiro ficam escondidas atrás de um "mostrar", para não poluir.

- Cadastro rápido: dá para digitar `30/70` no campo do ângulo e preencher os
  dois de uma vez.
- Busca por instância, stage, posição ou valor; reordenação com `▲▼`; edição
  clicando direto no nome ou no número.
- Importação de lista colada em texto — o parser entende cabeçalhos como
  `Pintin - Stage 2 (difícil)` e linhas como `alto - 50/50 - 20/45`.

### `lab.html` — Lab

O mesmo caderno, só que solto em blocos livres, para as fases que não encaixam
na estrutura de instância/stage — laboratório, eventos, testes. Também começa
vazio: o primeiro bloco nasce quando você cadastra o primeiro tiro.

## A lógica do tiro

A calculadora não usa as regras aproximadas da comunidade (*"full é 90 − distância"*).
Ela resolve a trajetória que o jogo realmente simula:

- **Bola com gravidade e arrasto linear** (`-kv`), com o vento entrando como
  aceleração horizontal constante. A "força" da barra de carga **é** a
  velocidade de lançamento: `vx = força·cos(ângulo)`, `vy = força·sen(ângulo)`.
- O jogo integra isso em passos de 0,04 s. Refazer o ajuste com o passo discreto
  em vez da forma fechada não muda o resultado, então a página usa a solução
  analítica e resolve força e ângulo por busca binária sobre ela.
- **As constantes** (gravidade, arrasto, escala de distância, altura de saída e
  o fator do vento) saíram de um ajuste por mínimos quadrados sobre 80 células
  de tabela medidas no jogo (ângulos 20/30/50/65, distâncias 1 a 20), mais a
  regra do tiro full. Erro final: RMSE de 1,08 de força, com 75 das 80 células
  dentro de ±2.
- **Conferência cruzada**: o modelo bate com a engenharia reversa do cliente e
  com a leitura do integrador do servidor. Convertidas para as mesmas unidades,
  as três origens independentes diferem em 2,9% no vento, 3,5% na conversão
  força→velocidade e 11,4% no arrasto.

Duas consequências que a página respeita e as regras de comunidade não:

- O ângulo de alcance máximo é **~41°**, não 45° — é o arrasto que puxa esse
  ponto para baixo. Por isso as tabelas de 30° e 50° são quase idênticas: são
  simétricas em torno dele.
- O alcance **satura** perto de 37 com força 100. "45° alcança 45" é
  matematicamente impossível, e a regra do full só vale mesmo entre 60° e 80°.

> As constantes valem para o mapa e a arma em que as tabelas foram medidas. No
> servidor, gravidade e arrasto são definidos **por mapa**, e massa, peso e
> arrasto **por munição** — trocar de mapa ou de arma muda os números.

O detalhamento completo (tabelas de origem, o ajuste e a comparação entre as
fontes) fica em `modelo-de-tiro.txt`, um documento local que não é versionado.

## Seus dados

O site é publicado **em branco** — ninguém herda os tiros de ninguém. Cada
página guarda o que é dela no `localStorage`, por navegador e por dispositivo:

| Página | Chave | O que guarda |
| --- | --- | --- |
| `calculadora.html` | `ddtank_calc` | Distância, vento, ângulo e as forças que você mediu |
| `referencias.html` | `ddtank_af_v1` | Instâncias, stages e tiros |
| `lab.html` | `ddtank_lab_v1` | Blocos e linhas |

### Um arquivo para as três

As páginas moram no mesmo endereço, então dividem o mesmo `localStorage` — é o
que permite um backup só. O menu baixa e carrega as três chaves de uma vez:

```json
{
  "app": "caderno-de-tiro",
  "v": 1,
  "exportado": "<data ISO>",
  "calculadora": { "d": 10, "w": 0, "ang": 65, "ov": { "65|12": 61 } },
  "referencias": { "v": 1, "instances": [ ... ] },
  "lab":         { "v": 1, "blocos":    [ ... ] }
}
```

Esse mesmo arquivo é aceito **dentro das Referências e do Lab**, que pegam de
dentro dele só a fatia delas e deixam o resto intacto. O caminho contrário
também vale: um `.json` antigo, baixado de uma página só, continua sendo aceito
em todo lugar. Qualquer arquivo que chega passa pelo mesmo saneamento — ids
validados, números limitados — então um `.json` de terceiro não consegue
injetar nada na página.

Não existe conta, back-end nem telemetria. As quatro páginas declaram uma
`Content-Security-Policy` com `default-src 'none'` e `connect-src 'none'`: elas
não carregam nem enviam nada para lugar nenhum, e o navegador impede que passem
a fazer isso.

Como nada sincroniza sozinho, o backup é manual e é seu — baixe o `.json` sempre
que cadastrar algo que não quer perder. Ele e os `.txt` de "copiar como texto"
ficam fora do repositório pelo `.gitignore`: são seus dados, não parte do site.
