README, RELATÓRIO TÉCNICO
Sistematização: Sistema de Visão Computacional para Detecção e Segmentação de Vagas de Estacionamento
Disciplina: Visão Computacional e Reconhecimento de Padrões, UniCEUB
Professor: Prof. Dr. Romes Heriberto Pires de Araújo
Integrantes: Higo Soares do Lago, Paulo Victor Torres Martins, Eduardo Deodoro de Moraes Florindo e Lúcio Flávio Vilar de Azevedo

1. Problema e cenário

Este projeto implementa um sistema de visão computacional para o cenário de Cidades Inteligentes, com foco em monitoramento automatizado de vagas de estacionamento. O sistema recebe uma imagem ou vídeo de um estacionamento e classifica cada vaga como ocupada (busy) ou livre (free), útil para aplicações de gestão de pátios, sinalização dinâmica de vagas disponíveis e otimização de fluxo de veículos em ambientes urbanos.

2. Dataset e análise exploratória (EDA)

Usamos o dataset público Parking Places (Roboflow Universe, vaRDas, licença CC BY 4.0), com 314 imagens anotadas em bounding box para as classes busy e free, já divididas pelo Roboflow em treino, validação e teste. Fonte: https://universe.roboflow.com/vardas/parking-places-yaul1

A análise exploratória cobriu três frentes:

- Distribuição de classes: no split de teste (39 imagens), há 740 instâncias de busy contra 435 de free, uma proporção de aproximadamente 63% para 37%. Esse desbalanceamento se mostrou relevante para o desempenho do modelo (ver seção de análise de erros).
- Distribuição de resolução das imagens: analisada para garantir consistência de escala de entrada do modelo.
- Condições de iluminação: o brilho médio de cada imagem foi medido numa escala de 0 (preto) a 255 (branco), sobre 727 imagens (treino, validação e teste combinados; esse total é maior que as 314 imagens originais provavelmente por causa de aumento de dados aplicado pelo Roboflow na geração da versão 3 do dataset). A média de brilho ficou em 85,2 (desvio padrão 31,8), com mínimo de 12,2 e máximo de apenas 171,5. Ou seja, o dataset concentra a maioria das imagens nas faixas escura e média, sem imagens realmente bem iluminadas ou superexpostas, um viés de iluminação que pode reduzir a robustez do modelo em cenários de luz forte (por exemplo, estacionamento ao meio-dia sob sol direto).

3. Metodologia

3.1 Detecção de objetos

Fine-tuning do modelo YOLOv8n (Ultralytics), pré-treinado no COCO, usando o dataset Parking Places. Hiperparâmetros: 50 épocas, imagem de entrada 640x640, batch size 16, splits fixos gerados pelo Roboflow (treino/validação/teste).

3.2 Segmentação

O dataset de origem só tem anotação de bounding box, sem máscara de segmentação, nenhuma das três versões publicadas pelo autor no Roboflow Universe inclui polígono. Diante disso, avaliamos duas rotas: anotar manualmente uma amostra com ferramenta de polígono (ex.: Smart Polygon do Roboflow), ou adaptar um modelo de segmentação já pré-treinado ao domínio do projeto. Optamos pela segunda, por uma razão de prazo (o projeto tem menos de um mês entre abertura e entrega, e anotar uma amostra com qualidade suficiente para treinar um segmentador consumiria a maior parte do tempo restante) e por uma razão técnica (o enunciado da sistematização permite explicitamente treinar OU adaptar o modelo de segmentação no mesmo domínio, não exige fine-tuning obrigatório).

A adaptação feita: modelo YOLOv8n-seg (Ultralytics), pré-treinado no COCO, aplicado sobre as mesmas imagens e o mesmo vídeo do estacionamento (mesmo domínio do cenário), restringindo a inferência às classes de veículo do COCO (carro, ônibus, caminhão, classes 2, 5 e 7) e com limiar de confiança ajustado para 0,10, calibrado empiricamente para capturar veículos parcialmente visíveis ou em ângulos incomuns sem gerar excesso de ruído. O resultado é uma segmentação de instância de veículos (carro estacionado ou em movimento), não uma segmentação direta de status de vaga (busy/free); a comparação visual entre as caixas do detector treinado e as máscaras do segmentador adaptado, no mesmo frame, está disponível no notebook. Essa decisão e sua limitação estão detalhadas na seção 6.

3.3 Vídeo e rastreamento

Inferência de detecção aplicada frame a frame sobre um vídeo real de estacionamento (mínimo de 30 segundos), gerando um vídeo anotado com as classes busy/free. Como item de bônus, implementamos também rastreamento de objetos no vídeo usando ByteTrack, aplicado sobre o modelo de detecção treinado.

4. Resultados

Métricas de detecção no conjunto de teste (39 imagens, 1175 instâncias):

- mAP@0.5: 0,818
- mAP@0.5:0.95: 0,439
- Precisão média: 0,859
- Recall médio: 0,760

Métricas por classe:

- busy: precisão 0,904, recall 0,918, mAP@0.5 0,961, mAP@0.5:0.95 0,626
- free: precisão 0,814, recall 0,603, mAP@0.5 0,675, mAP@0.5:0.95 0,253

A segmentação, por ser um modelo adaptado e não treinado no domínio, não tem métrica de mAP de máscara: a avaliação dela é qualitativa (comparação visual entre caixas e máscaras, disponível no notebook).

5. Análise de erros

O recall da classe free (0,603) é bem mais baixo que o da busy (0,918), o principal ponto fraco do modelo. Duas hipóteses foram levantadas a partir dos exemplos de falso negativo revisados no notebook:

- Desbalanceamento de classe: mais exemplos de busy do que de free no dataset (63%/37% no split de teste), o que tende a enviesar o modelo para a classe majoritária.
- Similaridade visual com o fundo: uma vaga ocupada tem contorno, cor e textura de veículo, bem diferentes do asfalto; uma vaga livre é composta majoritariamente por chão, faixa de demarcação e sombra, mais fácil de confundir com o fundo, sobretudo em vagas na borda da imagem ou parcialmente sombreadas, padrão observado nos exemplos com maior número de falsos negativos.

6. Limitações e próximos passos

- Segmentação não fine-tunada: a segmentação usada no projeto é um modelo pré-treinado adaptado (restrito a classes de veículo), não treinado no domínio busy/free do projeto, por falta de anotação de máscara no dataset de origem e por restrição de tempo. Como próximo passo, o ideal seria anotar uma amostra representativa com ferramenta de polígono (ex.: Smart Polygon do Roboflow) e treinar um YOLOv8-seg específico para vaga ocupada/livre.
- Desbalanceamento de classe: a classe free tem desempenho sensivelmente pior. Próximos passos possíveis: oversampling da classe minoritária, uso de pesos de classe no treino, ou mais épocas de treino combinadas com data augmentation direcionado a variação de sombra e iluminação.
- mAP@0.5:0.95 moderado (0,439): indica que, mesmo quando o modelo acerta a classe e a posição aproximada, a precisão de localização da caixa (IoU alto) ainda tem espaço para melhorar, algo que poderia ser endereçado com mais épocas de treino ou um backbone maior (YOLOv8s).
- Viés de iluminação: o dataset não tem imagens realmente claras (brilho máximo de 171,5 em 255), então o modelo não foi exposto a cenas de estacionamento sob luz forte ou superexposição. Próximo passo natural seria complementar o dataset com imagens mais claras, reais ou via data augmentation de brilho/contraste.

7. Declaração de Uso de Inteligência Artificial Generativa

Este trabalho utilizou apoio de assistente de IA generativa (Claude, Anthropic) em várias etapas do desenvolvimento: depuração de erros de execução no notebook (incluindo um erro de estouro de memória de GPU no processamento de vídeo), redação e revisão do relatório técnico e da proposta da Fase 1, comentários explicativos no código, e organização do repositório GitHub. As decisões de modelagem (escolha do dataset, do cenário, da estratégia de segmentação adaptada em vez de fine-tuning) foram tomadas pelo grupo; a IA foi usada como ferramenta de apoio à implementação, depuração e escrita, com revisão humana de todo o conteúdo antes da entrega. O grupo permanece com inteira autoria e responsabilidade científica, ética e legal sobre o trabalho.
