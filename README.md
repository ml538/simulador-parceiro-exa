# Simulador do parceiro · exa energia

Arquivo único, sem dependências. Abre com dois cliques, sobe em qualquer
hospedagem, funciona offline (só a tipografia muda, porque as fontes vêm
do Google Fonts).

## Estrutura do arquivo

`simulador_parceiro_exa.html` tem quatro partes, nesta ordem:

| Onde | O quê |
|---|---|
| `<style>` | Tokens de cor no `:root`, tema claro e escuro, layout |
| `CABEÇALHO` / `ENTRADAS` / `RESULTADO` | A marcação da página |
| `<script>` | Bloco `CONFIG` e a função `calc()` |
| `LOGO EXA EM VETOR` | O logo traçado, no fim do arquivo, fora do caminho |

Os primeiros 20 KB são o que você edita. O resto é o vetor do logo.

## Onde mexer

**Valores iniciais** — bloco `CONFIG`, no começo do `<script>`:

```js
var CONFIG = {
  volumeMinimo:      0,      // MWh · 0 desliga o alerta de carteira mínima
  desagioAquisicao:  25,     // %
  desagioCliente:    15,     // %
  pessoasPadrao:     200,
  conversaoPadrao:   20,
  contaPadrao:       900,
  tarifaPadrao:      1.00,
  contato:           "exa energia · exaenergia.com"
};
```

**Cores** — `:root` no `<style>`. O tema escuro repete os mesmos tokens
em dois blocos: `@media (prefers-color-scheme:dark)` e `[data-theme="dark"]`.
Mudou um, muda os dois.

**A conta** — função `calc()`. A lógica inteira são seis linhas:

```js
var clientes = Math.floor(pessoas * conv / 100);
var ref      = clientes * conta;          // valor de referência das faturas
var mwh      = clientes * (conta / tarifa) / 1000;
var compra   = ref * (1 - daq  / 100);    // o que o parceiro paga
var venda    = ref * (1 - dcli / 100);    // o que o parceiro recebe
var spread   = venda - compra;            // o ganho
```

## Parâmetros pela URL

Todos opcionais. Servem para mandar um link já preenchido por parceiro:

```
simulador.html?vmin=30&daq=40&dcli=30&pessoas=300&conv=25&conta=1200&tarifa=1.05
```

`vmin` MWh mínimos · `daq` deságio de aquisição · `dcli` deságio ao cliente
`pessoas` tamanho da rede · `conv` % de conversão · `conta` R$/mês por cliente
`tarifa` R$/kWh

Cuidado: link com parâmetro circula. Se o parceiro reencaminhar, a condição
de aquisição vai junto.

## Regras de conteúdo que não devem ser quebradas

- Nunca escrever "comprar energia", "vender energia" ou "venda de créditos".
  O objeto é **saldo de geração**, adquirido em **EXAs** dentro da plataforma.
- Os deságios padrão são neutros de propósito, porque a página é pública.
  A condição real entra pela URL ou é digitada na reunião.
