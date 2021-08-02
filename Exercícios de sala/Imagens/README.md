# Dataset "Experimento monitor"

Eu sempre tive a impressão de que o monitor do meu computador era azul demais. Por isso, eu coletei uma amostra de 40 fotos (tiradas por mim) composta por:

- fotos de desenhos (meus)
- fotos minhas
- fotos de coisas coloridas no geral
- e algumas fotos aleatórias (por exemplo, doguinhos)

Depois de coletar essa amostra, separei numa pasta as fotos originais. Então, usei a câmera de um mesmo celular para tirar foto de todas as fotos no monitor do meu computador (na pasta PC) e, já que eu já estava fazendo isso mesmo, tirei também do monitor do meu celular (na pasta Cel).

Quando já tinha as fotos em mãos, eu precisava extrair as features das imagens. O que eu pensei foi o seguinte: cada imagem é uma matriz de pixels, e cada pixel colorido é composto por 3 números inteiros: um corresponde à quantidade de vermelho, outro à quantidade de verde e o último, de azul (ou seja, RGB). Como eu queria saber se uma foto era mais azul do que a outra, eu só precisava extrair todos os números de "azul" para cada pixel e tirar a média de azuis na foto. No fim, qual tiver a maior média vai ser a foto mais azul. Só que como eu já estava fazendo isso mesmo, resolvi extrair azul, verde e vermelho para cada foto. Aí eu decidi extrair também a média dos tons de cinza (quando a imagem é em preto e branco, em vez de 3 números cada pixel tem só 1) para ver a luminosidade, porque sim.

Aí eu só criei uma coluna pra cada uma dessas features que eu comentei, e o dataframe no final ficou assim (cada linha é uma imagem, considerando suas 3 versões):

| Coluna  | Significado                                                |
|---------|------------------------------------------------------------|
| **orig_b**  | Média dos azuis da imagem original                         |
| **pc_b**    | Média dos azuis da imagem no monitor do computador         |
| **cel_b**   | Média dos azuis da imagem no monitor do celular            |
| **orig_g**  | Média dos verdes da imagem original                        |
| **pc_g**    | Média dos verdes da imagem no monitor do computador        |
| **cel_g**   | Média dos verdes da imagem no monitor do celular           |
| **orig_r**  | Média dos vermelhos da imagem original                     |
| **pc_r**    | Média dos vermelhos da imagem no monitor do computador     |
| **cel_r**   | Média dos vermelhos da imagem no monitor do celular        |
| **orig_bw** | Média dos tons de cinza da imagem original                 |
| **pc_bw**   | Média dos tons de cinza da imagem no monitor do computador |
| **cel_bw**  | Média dos tons de cinza da imagem no monitor do celular    |

Para fazer a extração das features, eu usei exatamente este código:

```python
!git clone https://github.com/hype-usp/Hype-Academy.git hype

data = pd.DataFrame(columns=['orig_b', 'pc_b', 'cel_b', 'orig_g', 'pc_g', 'cel_g', 'orig_r', 'pc_r', 'cel_r', 'orig_bw', 'pc_bw', 'cel_bw'])
data

import cv2
import numpy as np
import matplotlib.pyplot as plt
import os

# usei isso pra ter os paths prontos na hora de abrir as imagens
nomes_orig = os.listdir('hype/Exercícios de sala/Imagens/Originais')
orig = 'hype/Exercícios de sala/Imagens/Originais/'
pc = 'hype/Exercícios de sala/Imagens/PC/'
cel = 'hype/Exercícios de sala/Imagens/Cel/'

for n in nomes_orig:
  imgs = [orig+n, pc+n.split('_')[0]+'_pc.jpg', cel+n.split('_')[0]+'_cel.jpg'] # abro as 3 versões da mesma imagem
  features = [0]*12

  for i in range(len(imgs)):
    color = cv2.imread(imgs[i]) # leio a imagem normal
    bw = cv2.imread(imgs[i], cv2.IMREAD_GRAYSCALE) # e em preto e branco

    avg_channels = [0,0,0]
    for ch in range(3):
      avg_channels[ch] = round(np.mean(color[:,:,ch]),2) # aqui eu pego a média dos azuis, dos verdes e dos vermelhos (sim, é BGR ao invés de RGB)
    
    features[i] = avg_channels[0] # a primeira posição vai ser o azul
    features[i+3] = avg_channels[1] # depois o verde
    features[i+6] = avg_channels[2] # aí o vermelho
    features[i+9] = round(np.mean(bw),2) # e no fim tem a média do preto e branco porque sim
  
  # e nessa parte eu só adiciono as features que eu extraí dessa imagem no dataframe
  ft = pd.DataFrame(features).transpose()
  ft.columns = data.columns
  ft.index = [data.shape[0]]
  data = data.append(ft)
    
data.to_csv('imagens.csv') # e salvo o csv :)
```
