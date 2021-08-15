# Ètudes dataset

Este conjunto de dados foi baseado num subconjunto do MAESTRO (https://magenta.tensorflow.org/datasets/maestro), em sua versão encontrada no Kaggle (https://www.kaggle.com/jackvial/themaestrodatasetv2). O conjunto de dados em questão é uma amostra de alguns estudos de piano, com dados do MAESTRO e estatísticas do IMSLP e Spotify (OBS.: todos os dados utilizados aqui estão sob a licença Creative Commons).

## Processo de criação

1. Amostragem (não-probabilística intencional) do conjunto de dados MAESTRO utilizando o Kaggle (o notebook em questão pode ser visto em https://www.kaggle.com/laiscarraro/maestro-subsetting)
2. Download e separação de arquivos `.mid` (na pasta midi) e `.mp3` (na pasta mp3)
3. Anotação dos metadados, que podem ser encontrados no arquivo `etudes.csv` (https://github.com/hype-usp/Hype-Academy/blob/main/Exerc%C3%ADcios%20de%20sala/etudes/etudes.csv)
4. Extração de *features* dos arquivos `.mid` utilizando a biblioteca `music21` (https://web.mit.edu/music21/)

## Dicionário de dados

| Coluna          | Significado                                                                                           |
|-----------------|-------------------------------------------------------------------------------------------------------|
| id              | id do arquivo                                                                                         |
| composer        | compositor da peça                                                                                    |
| opus            | número da obra                                                                                        |
| number          | número da peça dentro da obra                                                                         |
| historical_era  | era histórica (musical) do compositor                                                                 |
| imslp_downloads | número de downloads da peça ou da obra no IMSLP*                                                      |
| spotify views   | número de visualizações da peça no spotify                                                            |
| key             | tonalidade da peça                                                                                    |
| duration        | duração da peça (medida em número de semínimas)                                                       |
| tempo           | BPM (*beats per minute*)                                                                              |
| time_signature  | fórmula de compasso da peça                                                                           |
| pitch_list      | *string* das notas em notação de cifra, separadas por espaço (**OBS.:** pausas representadas por 'R') |
| midi_list       | *string* do número MIDI das notas, separadas por espaço (**OBS.:** pausas representadas por -1)       |
| durations_list  | *string* da duração em semínimas das notas, separadas por espaço                                      |
| volume_list     | *string* do volume percebido das notas, separados por espaço (**OBS.:** pausas representadas por 0)   |

*IMSLP: https://imslp.org/wiki/Main_Page

## Código para extração de *features*

```python
import os
import pandas as pd

etudes_data = pd.DataFrame(columns=['id', 'key', 'duration', 'tempo', 'time_signature', 'pitch_list', 'midi_list', 'duration_list', 'volume_list'])
files = os.listdir('hype/Exercícios de sala/etudes/midi')

for path in files:
  song = converter.parse('hype/Exercícios de sala/etudes/midi/'+path)
  name = ('hype/Exercícios de sala/etudes/'+path).split('/')[3].split('.mid')[0]
  key = song.analyze('key').name
  duration = song.duration.quarterLength
  tempo = song.flat[1].number
  time_signature = song.flat[2].ratioString

  pitch = ''
  midi = ''
  durations = ''
  volume = ''

  for i in song.flat.notesAndRests:
    if i.isRest:
      pitch += 'R '
      midi += '-1 '
      volume += '0 '
    elif i.isChord:
      pitch += i.root().name + ' '
      midi += str(i.root().midi) + ' '
      volume += str(i.volume.getRealized()) + ' '
    else:
      pitch += i.pitch.name + ' '
      midi += str(i.pitch.midi) + ' '
      volume += str(i.volume.getRealized()) + ' '
    durations += str(round(float(i.quarterLength), 2)) + ' '

  line = pd.DataFrame([name, key, duration, tempo, time_signature, pitch, midi, durations, volume]).transpose()
  line.columns = ['id', 'key', 'duration', 'tempo', 'time_signature', 'pitch_list', 'midi_list', 'duration_list', 'volume_list']
  line.columns = etudes_data.columns
  line.index = [etudes_data.shape[0]]
  etudes_data = etudes_data.append(line)
```

## Sugestões para o uso
Uma sugestão para facilitar o uso do conjunto de dados é transformar em listas as *features* do MIDI que estão como *string* (as 4 últimas), e transformar os elementos das 3 últimas em *float*. Isso pode ser feito através das funções:

```python
def split_str(t):
  return t.split(' ')

def list_float(l):
  return [float(i) for i in l[:-1]]
```

Em conjunto com a função `.apply` do pandas.

## Exemplos de uso
Coming soon
