PROPOSTA, FASE 1: DEFINIÇÃO E DADOS
Sistematização, Visão Computacional e Reconhecimento de Padrões
Disciplina: Visão Computacional e Reconhecimento de Padrões, UniCEUB
Professor: Prof. Dr. Romes Heriberto Pires de Araújo

Integrantes: Higo Soares do Lago, Paulo Victor Torres Martins, Eduardo Deodoro de Moraes Florindo, Lúcio Flávio Vilar de Azevedo

1. Problema

O grupo assume o papel de um time de engenharia de IA contratado para construir um sistema de visão computacional que detecta e segmenta vagas de estacionamento, classificando cada vaga como ocupada ou livre a partir de imagens e vídeo. O cenário escolhido é Cidades Inteligentes, com foco em monitoramento automatizado de estacionamentos, um problema real de gestão urbana e de operação de pátios e prédios comerciais.

2. Classes-alvo

Duas classes: busy (vaga ocupada) e free (vaga livre).

3. Fonte dos dados

Dataset público Parking Places, publicado no Roboflow Universe pelo usuário vaRDas, sob licença CC BY 4.0. Contém 314 imagens anotadas em bounding box para as classes busy e free, já divididas pelo Roboflow em conjuntos de treino, validação e teste. Disponível em: https://universe.roboflow.com/vardas/parking-places-yaul1

4. Ferramenta de anotação

O dataset já vem anotado pelo autor original na própria plataforma Roboflow, não sendo necessário anotar as 314 imagens da base de detecção do zero. Para a etapa de segmentação, avaliamos anotar manualmente uma amostra com a ferramenta de polígono do Roboflow, mas, dado o prazo do projeto, optamos por adaptar um modelo de segmentação já pré-treinado (ver seção de metodologia e limitações no relatório final), em vez de treinar um segmentador específico no domínio das vagas.

5. Vídeo de demonstração

Para a etapa de inferência em vídeo (Fase 4), será usado um vídeo real de um estacionamento, gravado por um dos integrantes do grupo, com duração superior ao mínimo de 30 segundos exigido pelo enunciado.
