# FFTLab

Ferramenta livre (licença **MIT**) para análise de vibração e sinais, escrita em Python.
Desenvolvida do zero a partir de métodos de domínio público (Welch, Hilbert, ASTM E1049, fórmulas
clássicas de rolamentos); **não contém código de nenhum outro programa**.

## Recursos
| Recurso | Detalhes |
|---|---|
| Auto-espectro | Welch com médias; escala pico / RMS / PSD; janelas Hann, Hamming, Blackman, Flat-top, Bartlett, Retangular; dB; eixo log |
| Coerência | γ² entre dois canais (+ função de transferência H1 em `dsp.transfer_function`) |
| Envelope 1 e 2 | Passa-banda Butterworth (zero-fase) → Hilbert. **1** = envelope \|x\|; **2** = envelope ao quadrado (realça impactos repetitivos) |
| Waterfall | STFT; vista em linhas ou vista superior (mapa de cores) |
| Rotação | Botão “Estimar rpm” (pico dominante do canal selecionado) |
| Cursores e harmônicos | Clique no gráfico: o cursor “gruda” no pico (interpolação parabólica) e marca N harmônicos; harmônicos de rotação 1x…Nx |
| Rolamentos | FTF, BPFO, BPFI, BSF a partir da geometria, marcados nos gráficos; tabela em arquivo `.BRG` |
| Rainflow | ASTM E1049 (3 pontos): histograma de faixas, matriz faixa×média, coleção de cargas, faixa equivalente e dano de Palmgren-Miner (S-N de Basquin, sem correção de tensão média) |

## Instalação (Windows / Linux / macOS)
1. Python 3.10+ (no Windows, o instalador de python.org já inclui o tkinter).
2. `pip install -r requirements.txt`
3. Interface gráfica: `python -m fftlab`
   Linha de comando: `python -m fftlab.cli arquivo.csv -c canal --rpm 1500 --brg examples/rolamentos.brg --band 3000 5500`

Teste rápido: `python examples/make_demo.py` gera `demo_rolamento.csv` (falha de pista externa em um 6205 a 1500 rpm;
BPFO esperado = 89,6 Hz). No envelope o pico deve aparecer em ~89,6 Hz e nos múltiplos.

## Executável para Windows (sem Python)
Veja `packaging/LEIA-ME_BUILD.md`: um fluxo do GitHub Actions gera o instalador `.exe` e uma versão portátil automaticamente.

## Formatos de entrada
* **CSV/TXT/DAT** (`;`, tab, vírgula ou espaço; cabeçalho opcional; vírgula decimal aceita). Com coluna de tempo, `fs` é calculado; sem ela, preencha `fs [Hz]`.
* **WAV**, **NPY/NPZ** (informe `fs`).
* **Binário bruto**: tipo (`<i2`, `<i4`, `<f4`…), nº de canais, offset do cabeçalho, escala.
* **.LTD (Lynx "time domain")** - leitor completo, validado com um arquivo real (UHE, 18 canais, 167 440 amostras):
  nomes, unidades, sensores, taxa de amostragem, data/hora, local e conversão automática de contagens do ADC para
  unidades de engenharia (`eng = V·(máx−mín)/20 + (máx+mín)/2`, com V = contagem/32768·10). Opções em Python:
  `read_ltd(path, raw=True)` para contagens brutas e `read_ltd(path, channels=[...])` para carregar só alguns canais.
  O leitor confere que o tamanho do arquivo bate exatamente com o declarado no cabeçalho e **recusa** arquivos
  truncados em vez de ler dados deslocados. Se aparecer uma variante diferente, o programa mostra o cabeçalho e
  oferece o importador binário bruto.
  Layout (engenharia reversa, int16 little-endian intercalado por varredura): ver comentário em `fftlab/readers.py`.

## Formato .BRG (texto)
```
# nome ; n_esferas ; diâmetro_esfera ; diâmetro_primitivo ; ângulo_de_contato_graus
6205 ; 9 ; 7.94 ; 39.04 ; 0
```
Fórmulas (pista interna girando com o eixo, externa fixa; n = rotação em Hz, θ = ângulo de contato):
FTF = n/2·(1 − Bd/Pd·cosθ), BPFO = Nb/2·n·(1 − Bd/Pd·cosθ), BPFI = Nb/2·n·(1 + Bd/Pd·cosθ),
BSF = Pd/(2Bd)·n·(1 − (Bd/Pd·cosθ)²).
Observação: este `.BRG` é um formato **novo e em texto**, não o binário de outros programas.

## Testes
`python -m pytest tests -q` (defina `FFTLAB_LTD_SAMPLE=caminho/arquivo.LTD` para incluir um teste com seu arquivo real) (sinais sintéticos com resposta conhecida: escala de amplitude, Parseval,
coerência, BPFO no envelope, varredura no waterfall, exemplo da ASTM E1049, leitores).

## Limitações
* Envelope 1/2 e as escalas seguem definições usuais de literatura; podem diferir em detalhes de outros softwares.
* Cursores/harmônicos aplicam-se às abas Espectro, Coerência e Envelope.
* Sem rastreamento de ordens (order tracking) nem correção de tensão média no Miner.
