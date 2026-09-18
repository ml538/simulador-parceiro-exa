# Simuladores · exa energia

Duas calculadoras comerciais em HTML puro, sem dependências, sem build.
Cada arquivo abre com dois cliques e funciona offline (só a tipografia muda,
porque as fontes vêm do Google Fonts).

```
index.html      capa com os dois links
cliente.html    economia do cliente final
parceiro.html   ganho do parceiro
README.md       este arquivo
```

---

## Publicar no GitHub Pages

1. Crie um repositório, por exemplo `exa-simuladores`.
2. Suba os quatro arquivos na raiz do repositório.
3. Em **Settings → Pages**, escolha `Deploy from a branch`, branch `main`, pasta `/ (root)`.
4. O endereço sai como `https://SEU-USUARIO.github.io/exa-simuladores/`.

Para usar domínio próprio, por exemplo `simulador.exaenergia.com.br`:

1. No DNS do domínio, crie um registro `CNAME` apontando
   `simulador` para `SEU-USUARIO.github.io`.
2. Em **Settings → Pages → Custom domain**, informe o domínio e salve.
   O GitHub cria um arquivo `CNAME` no repositório.
3. Marque **Enforce HTTPS** depois que o certificado for emitido.

Os links finais ficam:

```
simulador.exaenergia.com.br/            capa
simulador.exaenergia.com.br/cliente     economia do cliente
simulador.exaenergia.com.br/parceiro    ganho do parceiro
```

---

## Estrutura interna dos arquivos

Os dois simuladores têm o mesmo formato, nesta ordem:

| Onde | O quê |
|---|---|
| `<style>` | Tokens de cor no `:root`, tema claro e escuro, layout |
| Marcação | Cabeçalho, entradas e resultado |
| `<script>` | Bloco `CONFIG` e a função `calc()` |
| Fim do arquivo | O logo da exa em vetor, isolado, não precisa mexer |

Os primeiros 20 KB são tudo o que se edita. O resto é o vetor do logo.

---

## cliente.html · economia do cliente

Mostra quanto o cliente paga na tarifa cheia, quanto passa a pagar com a exa e
quanto sobra por mês e por ano. Aceita várias unidades consumidoras e tem um
interruptor para o caso de o cliente já ter desconto com outro fornecedor. Com
ele ligado, o cálculo final passa a comparar contra o desconto atual, não
contra a tarifa cheia.

O botão **Imprimir ou salvar PDF** esconde a coluna de entrada e gera só o
documento, para mandar ao cliente depois da reunião.

**Desconto máximo e arraste ao vivo.** O campo "Desconto máximo que você pode
oferecer" define o teto. O campo abaixo dele ("Desconto que você oferece
agora") arrasta de 0 até esse teto — dá para definir o máximo antes da
reunião e ir arrastando o desconto real na frente do cliente, sem que a barra
deixe passar do teto. O ícone 👁 ao lado do máximo esconde o número (vira uma
senha, com pontinhos) na hora de virar a tela para o cliente; clique de novo
para revelar. O desconto atual do concorrente usa a mesma escala do teto, então
a posição dele na barra mostra visualmente se está abaixo ou acima do seu máximo.

Todos os campos abrem **zerados** de propósito — nada de números de exemplo
que possam parecer uma oferta real antes do consultor preencher.

### Valores iniciais

```js
var CONFIG = {
  tarifaPadrao:   0,      // R$ por kWh da distribuidora
  descontoMaximo: 0,      // % · teto do desconto, defina antes da reunião
  descontoExa:    0,      // % oferecido com a exa
  descontoAtual:  0,      // % que o cliente já tem hoje, se tiver
  nomeCliente:    "",
  rodapeExtra:    "Sem obra, sem instalação e sem investimento inicial.",
  aviso:          "* A economia apresentada é um valor aproximado e precisa ser revista no estudo completo."
};
```

### Parâmetros pela URL

```
cliente.html?cliente=Rede%20Aguia&tarifa=1.18&kwh=9954&maxexa=30&dexa=30&atual=20
```

| Parâmetro | O quê |
|---|---|
| `cliente` | nome que aparece no documento |
| `tarifa` | R$ por kWh da distribuidora |
| `kwh` | consumo da primeira unidade |
| `maxexa` | teto do desconto que você pode oferecer, em % |
| `dexa` | desconto oferecido com a exa agora, em % (não pode passar de `maxexa`) |
| `atual` | desconto que o cliente já tem; informar este parâmetro já liga o interruptor |

---

## parceiro.html · ganho do parceiro

Mostra quanto a rede do parceiro pode render. Ele informa quantas pessoas
conhece, quantas converte, a conta média e os dois deságios. O resultado é o
spread mensal, o acumulado de doze meses e a régua que compara a oferta dele
com o padrão do mercado.

Diferente do `cliente.html`, aqui não existe um campo de teto separado: o
próprio **deságio de aquisição** (`daq`) é o teto. Defina o `daq` primeiro
(ex. 40%) — ele fica cravado ali, é informação que já é aberta com o parceiro
— e o campo "Deságio ao seu cliente agora", logo abaixo, arrasta de 0 até o
`daq` sem passar dele, mostrando visualmente o quanto sobra de spread para
você. Todos os campos abrem zerados.

### Valores iniciais

```js
var CONFIG = {
  volumeMinimo:      0,      // MWh · 0 desliga o alerta de carteira mínima
  desagioAquisicao:  0,      // % · valor neutro, a condição real vai pela URL · também é o teto do dcli
  desagioCliente:    0,      // % · valor neutro, a condição real vai pela URL
  pessoasPadrao:     0,
  conversaoPadrao:   0,
  contaPadrao:       0,
  tarifaPadrao:      0,      // R$ por kWh, só para converter em MWh
  contato:           "exa energia · exaenergia.com"
};
```

### Parâmetros pela URL

```
parceiro.html?vmin=30&daq=40&dcli=30&pessoas=300&conv=25&conta=1200&tarifa=1.05
```

| Parâmetro | O quê |
|---|---|
| `vmin` | MWh de carteira mínima da condição |
| `daq` | deságio de aquisição, em % · também é o teto do `dcli` |
| `dcli` | deságio oferecido ao cliente agora, em % (não pode passar de `daq`) |
| `pessoas` | tamanho da rede |
| `conv` | conversão, em % |
| `conta` | conta média por cliente, em R$ |
| `tarifa` | R$ por kWh |

---

## Cuidados que valem mais que o código

**A página do parceiro é pública.** Os deságios padrão são neutros de propósito.
A condição comercial real entra pela URL ou é digitada na reunião, nunca fica
no arquivo.

**Link com parâmetro circula.** Se o parceiro reencaminhar um endereço com
`daq=40`, a condição de aquisição vai junto. Para negociação sensível, o
consultor digita na tela.

**Terminologia que não pode ser quebrada.** Nunca escrever "comprar energia",
"vender energia" ou "venda de créditos". O objeto é **saldo de geração**,
adquirido em **EXAs** dentro da plataforma. A operação é rateio de aluguel
dentro da associação civil, e os créditos vêm por consequência no SCEE.

**Padrões de desconto da exa**, por unidade consumidora:

| Consumo por unidade | O que pode ser oferecido |
|---|---|
| Até 500 kWh | Plano Fixo |
| 500 a 2.000 kWh | Plano Fixo ou desconto de até 20% |
| Acima de 2.000 kWh | Desconto de até 30% |

Nenhum dos dois simuladores trava esses limites hoje, porque quem preenche é o
consultor. Se um dia forem usados pelo cliente sozinho, essa trava precisa
existir.

---

## Cores

Os dois arquivos usam os mesmos tokens, definidos em três lugares que precisam
ser alterados juntos: `:root`, o bloco `@media (prefers-color-scheme:dark)` e
o bloco `:root[data-theme="dark"]`.

| Token | Claro | Escuro |
|---|---|---|
| `--bg` | `#F7F3E7` | `#001409` |
| `--accent` | `#00A95B` | `#ADF98E` |
| `--head` | `#11201A` | `#FFFFFF` |

Tipografia: **Inter** no texto, **Oxanium** só nos números e títulos de destaque.
