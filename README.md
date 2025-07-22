# Desafio Técnico: Pipeline de Inteligência Artificial para Consulta de Documentos

## Visão Geral do Projeto

Este repositório contém a solução desenvolvida para o **Desafio Técnico de Inteligência Artificial** proposto pelo Núcleo de Visão Computacional e Engenharia. O objetivo principal é construir um pipeline funcional de **Retrieval Augmented Generation (RAG)** capaz de processar documentos heterogêneos (PDFs com e sem texto, e imagens), indexá-los em uma base vetorial e, posteriormente, utilizar uma Large Language Model (LLM) para responder a perguntas com base no conteúdo desses documentos.

O projeto foi desenvolvido em notebooks Jupyter (Google Colab) para facilitar a execução e a demonstração das etapas.

## Funcionalidades Implementadas

O pipeline é modular e composto pelas seguintes etapas interconectadas:

1.  **Extração de Conteúdo (e OCR quando necessário)**:
    * **Objetivo**: Converter documentos em diversos formatos para texto bruto pesquisável.
    * **Capacidades**:
        * Leitura de **PDFs com texto nativo** ("CÓDIGO DE OBRAS.pdf").
        * Leitura de **PDFs digitalizados** e **imagens** (JPEG, PNG, WebP) com aplicação de **OCR (Optical Character Recognition)** ("tabela.webp" foi convertida para .jpeg para melhorar a leitura).
    * **Escolhas Técnicas**:
        * Para **PDFs nativos**, a biblioteca `pdfplumber` foi utilizada devido à sua alta precisão na extração de texto e capacidade de preservar a estrutura.
        * Para **PDFs digitalizados e imagens**, `pytesseract` (interface para o motor Tesseract-OCR) foi a escolha principal, em conjunto com `pdf2image` para conversão de PDF para imagem. O idioma **`'por'` (Português)** foi especificado para o Tesseract para otimizar o reconhecimento em documentos brasileiros.
        * Foi implementada uma lógica de fallback: se a extração nativa de um PDF resulta em pouco ou nenhum texto significativo, o OCR é acionado automaticamen
    * **Implementação**: Notebook `Extracao.ipynb`.
    * **Saída**: Textos extraídos salvos em arquivos `.txt` no diretório `data/extracted_texts/`.

2.  **Indexação e Geração de Base Vetorial**:
    * **Objetivo**: Preparar os textos extraídos para uma recuperação de informações eficiente, transformando-os em uma base de dados pesquisável semanticamente.
    * **Sub-etapas**:
        * **Chunking (Divisão em Trechos)**: Os textos brutos são divididos em pedaços menores ("chunks").
        * **Geração de Embeddings (Vetores Semânticos)**: Cada chunk é convertido em uma representação numérica de alta dimensão (embedding).
        * **Armazenamento em Banco Vetorial**: Os embeddings são armazenados em um banco de dados otimizado para busca por similaridade.
    * **Escolhas Técnicas**:
        * A biblioteca `LangChain` foi central para orquestrar esta etapa.
        * `RecursiveCharacterTextSplitter` foi utilizado para o chunking, permitindo dividir o texto de forma inteligente, respeitando a estrutura do documento (com `chunk_size=1000` caracteres e `chunk_overlap=100` caracteres para manter o contexto). Separadores explícitos (`"\n\n"`, `"\n"`, `"."`, `" "`, `""`) foram definidos para melhor controle das quebras.
        * O modelo de embeddings `sentence-transformers/all-MiniLM-L6-v2` foi escolhido por seu equilíbrio entre tamanho, eficiência e qualidade semântica para tarefas gerais de similaridade.
        * `FAISS` (Facebook AI Similarity Search) foi implementado como o banco vetorial, devido à sua alta performance para buscas de similaridade em memória e facilidade de uso em ambientes como o Colab.
    * **Implementação**: Notebook `Indexacao.ipynb`.
    * **Saída**: Arquivos do índice FAISS (`index.faiss` e `index.pkl`) salvos no diretório `faiss_index/`.

3.  **Recuperação de Informações e Geração de Resposta (LLM)**:
    * **Objetivo**: Receber uma pergunta em linguagem natural, encontrar as informações mais relevantes nos documentos indexados e gerar uma resposta coerente e fundamentada utilizando uma LLM.
    * **Sub-etapas**:
        * **Recebimento da Pergunta**: O sistema aceita uma pergunta em linguagem natural do usuário.
        * **Busca de Relevância**: A pergunta é vetorizada e usada para consultar o índice FAISS, que retorna os trechos de documentos mais semanticamente relevantes (os "chunks de contexto").
        * **Geração de Resposta com LLM**: Os chunks recuperados são passados para uma Large Language Model (LLM) como contexto. A LLM é então instruída a gerar a resposta.
    * **Escolhas Técnicas**:
        * A integração com a LLM é feita utilizando as cadeias de `RetrievalQA` da `LangChain`.
        * Um `PromptTemplate` rigoroso foi cuidadosamente elaborado para instruir a LLM a:
            * **Usar SOMENTE o contexto fornecido.**
            * **Responder "Não sei"** se a informação não estiver explicitamente no contexto, evitando alucinações.
            * Manter a resposta concisa e diretamente relevante.
        * Como LLM, foi utilizado o modelo `google/gemma-2b-it` (via `HuggingFacePipeline` do HuggingFace Transformers), uma escolha open-source que demonstra a capacidade do pipeline de funcionar com modelos acessíveis, embora com desafios inerentes a modelos menores (como a aderência estrita a instruções complexas).
    * **Implementação**: Notebook `Recuperacao_e_Resposta.ipynb`.
    * **Saída**: Resposta em linguagem natural para a pergunta do usuário e a indicação da `Fonte` (nome do arquivo original) dos trechos de onde a informação foi recuperada.

## Recursos Fornecidos

O projeto foi desenvolvido e testado utilizando um pequeno conjunto de documentos fornecidos no desafio:

* `CÓDIGO DE OBRAS.pdf` (PDF com texto nativo)
* `Tabela de custos de materiais de construção` (utilizado o arquivo `tabela.png` após conversão, uma imagem com texto)

## Estrutura do Repositório
