---
lab:
  title: Orquestrar um sistema RAG
  description: Saiba como implementar sistemas de RAG (Geração Aumentada por Recuperação) em seus aplicativos para aumentar a precisão e a relevância das respostas geradas.
---

## Orquestrar um sistema RAG

Os sistemas RAG (Geração Aumentada por Recuperação) combinam o poder de grandes modelos de linguagem com mecanismos de recuperação eficientes para aumentar a precisão e a relevância das respostas geradas. Ao usar o LangChain para orquestração e a Fábrica de IA do Azure para funcionalidades de IA, podemos criar um pipeline robusto que recupera informações relevantes de um conjunto de dados e gera respostas coerentes. Neste exercício, você percorrerá as etapas de configuração do ambiente, pré-processamento de dados, criação de inserções e criação de um índice, permitindo que você implemente um sistema RAG de forma eficaz.

Este exercício levará aproximadamente **30** minutos.

## Cenário

Imagine que você deseja criar um aplicativo que forneça recomendações sobre hotéis em Londres. No aplicativo, você quer um agente que possa não apenas recomendar hotéis, mas também responder a perguntas que os usuários possam ter sobre eles.

Você selecionou um modelo GPT-4 para fornecer respostas generativas. Agora você deseja montar um sistema RAG que fornecerá dados de base para o modelo com base nas avaliações de outros usuários, orientando o comportamento do chat para fornecer recomendações personalizadas.

Vamos começar implantando os recursos necessários para criar esse aplicativo.

## Criar um projeto e hub de IA do Azure

Você pode criar um hub de IA do Azure e projetar manualmente por meio do portal Fábrica de IA do Azure, bem como implantar os modelos usados no exercício. No entanto, você também pode automatizar esse processo usando um aplicativo de modelo com o [Azure Developer CLI (azd)](https://aka.ms/azd).

1. Em um navegador da Web, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre usando suas credenciais do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para Versão Clássica**.

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório deste exercício:

    ```powershell
   rm -r mslearn-genaiops -f
   git clone https://github.com/MicrosoftLearning/mslearn-genaiops
    ```

1. Depois que o repositório for clonado, insira os comandos a seguir para inicializar o modelo Starter. 
   
    ```powershell
   cd ./mslearn-genaiops/Starter
   azd init
    ```

1. Quando solicitado, dê um nome ao novo ambiente, pois ele será usado como base para dar nomes exclusivos a todos os recursos provisionados.
        
1. Em seguida, insira o comando a seguir para executar o modelo Starter. Ele provisionará um Hub de IA com recursos dependentes, projeto de IA, serviços de IA e um ponto de extremidade online. Ele também implantará os modelos GPT-4 Turbo, GPT-4o e GPT-4o mini.

    ```powershell
   azd up  
    ```

1. Quando solicitado, escolha qual assinatura você deseja usar e escolha um dos seguintes locais para provisionamento de recursos:
   - Leste dos EUA
   - Leste dos EUA 2
   - Centro-Norte dos EUA
   - Centro-Sul dos Estados Unidos
   - Suécia Central
   - Oeste dos EUA
   - Oeste dos EUA 3
    
1. Aguarde a conclusão do script – isso normalmente leva cerca de 10 minutos, mas em alguns casos pode levar mais tempo.

    > **Observação**: os recursos do OpenAI do Azure são restringidos no nível do locatário por cotas regionais. As regiões listadas acima incluem a cota padrão para os tipos de modelos usados neste exercício. Escolher aleatoriamente uma região reduz o risco de uma única região atingir o seu limite de cota. No caso de um limite de cota ser atingido, há a possibilidade de você precisar criar outro grupo de recursos em uma região diferente. Saiba mais sobre a [disponibilidade do modelo por região](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=standard%2Cstandard-chat-completions#global-standard-model-availability)

    <details>
      <summary><b>Dica de solução de problemas</b>: nenhuma cota disponível em uma determinada região</summary>
        <p>Se você receber um erro de implantação para qualquer um dos modelos devido à falta de cota disponível na região escolhida, tente executar os seguintes comandos:</p>
        <ul>
          <pre><code>azd env set AZURE_ENV_NAME new_env_name
   azd env set AZURE_RESOURCE_GROUP new_rg_name
   azd env set AZURE_LOCATION new_location
   azd up</code></pre>
        Substituir <code>new_env_name</code>, <code>new_rg_name</code> e <code>new_location</code> por novos valores. O novo local deve ser uma das regiões listadas no início do exercício, por exemplo <code>eastus2</code>, <code>northcentralus</code>, etc.
        </ul>
    </details>

1. Depois que todos os recursos forem provisionados, use os comandos a seguir para buscar o ponto de extremidade e a chave de acesso para o recurso Serviços de IA. Você deve substituir `<rg-env_name>` e `<aoai-xxxxxxxxxx>` pelos nomes do grupo de recursos e do recurso Serviços de IA. Ambos estão impressos na saída da implantação.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
     ```

     ```powershell
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copie esses valores, pois eles serão usados posteriormente.

## Configurar seu ambiente de desenvolvimento no Cloud Shell

Para experimentar e iterar rapidamente, você usará um conjunto de scripts Python no Cloud Shell.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para navegar até a pasta com os arquivos de código usados neste exercício:

     ```powershell
    cd ~/mslearn-genaiops/Files/04/
     ```

1. Insira os seguintes comandos para ativar um ambiente virtual e instalar as bibliotecas necessárias:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv langchain-text-splitters langchain-community langchain-openai
    ```

1. Digite o seguinte comando para abrir o arquivo de configuração que foi fornecido:

    ```powershell
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua os espaços reservados **your_azure_openai_service_endpoint** e **your_azure_openai_service_api_key** pelos valores do ponto de extremidade e da chave que você copiou anteriormente.
1. *Após* substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **clique com o botão direito > Salvar** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** ou **clique com o botão direito > Sair** para fechar o editor de código mantendo a linha de comando do Cloud Shell aberta.

## Implementar a RAG

Agora você executará um script que ingere e pré-processa os dados, cria inserções e constrói um armazenamento vetorial e um índice, permitindo a implementação eficaz de um sistema RAG.

1. Execute o seguinte comando para **editar o script** fornecido:

    ```powershell
   code RAG.py
    ```

1. No script, localize **# Inicializar os componentes que serão usados do pacote de integrações de LangChain**. Cole o seguinte código abaixo desse comentário :

    ```python
   # Initialize the components that will be used from LangChain's suite of integrations
   llm = AzureChatOpenAI(azure_deployment=llm_name)
   embeddings = AzureOpenAIEmbeddings(azure_deployment=embeddings_name)
   vector_store = InMemoryVectorStore(embeddings)
    ```

1. Examine o script e observe que ele usa um arquivo .csv com avaliações de hotéis como dados de base. Veja o conteúdo desse arquivo executando o comando `download app_hotel_reviews.csv` no painel de linha de comando e abrindo o arquivo.
1. Localize **# Dividir os documentos em partes para inserção e armazenamento de vetor**. Cole o seguinte código abaixo desse comentário :

    ```python
   # Split the documents into chunks for embedding and vector storage
   text_splitter = RecursiveCharacterTextSplitter(
       chunk_size=200,
       chunk_overlap=20,
       add_start_index=True,
   )
   all_splits = text_splitter.split_documents(docs)
    
   print(f"Split documents into {len(all_splits)} sub-documents.")
    ```

    O código acima dividirá em partes menores um conjunto de documentos grandes. Isso é importante porque muitos modelos de inserção (como aqueles usados para bancos de dados vetoriais ou de pesquisa semântica) têm um limite de token e um melhor desempenho em textos mais curtos.

1. Localize **# Inserir o conteúdo de cada parte de texto e colocar essas inserções em um repositório de vetores**. Cole o seguinte código abaixo desse comentário :

    ```python
   # Embed the contents of each text chunk and insert these embeddings into a vector store
   document_ids = vector_store.add_documents(documents=all_splits)
    ```

1. Localize **# Recuperar documentos relevantes do repositório de vetores com base na entrada do usuário**. Abaixo deste comentário, cole o seguinte código observando o recuo adequado:

    ```python
   # Retrieve relevant documents from the vector store based on user input
   retrieved_docs = vector_store.similarity_search(question, k=10)
   docs_content = "\n\n".join(doc.page_content for doc in retrieved_docs)
    ```

    O código acima pesquisa no repositório de vetores os documentos mais parecidos com a pergunta de entrada do usuário. A pergunta é convertida em um vetor usando o mesmo modelo de inserção usado para os documentos. O sistema então compara esse vetor a todos os vetores armazenados e recupera os mais semelhantes.

1. Salve suas alterações.
1. **Execute o script** digitando o seguinte comando na linha de comando:

    ```powershell
   python RAG.py
    ```

1. Depois que o aplicativo estiver em execução, você poderá começar a fazer perguntas como `Where can I stay in London?` e, em seguida, continuar com perguntas mais específicas.

## Conclusão

Neste exercício, você criou um sistema RAG típico com seus principais componentes. Ao usar seus próprios documentos para informar as respostas de um modelo, você fornece dados de base que o LLM usa ao formular uma resposta. Para uma solução empresarial, isso significa que você pode limitar a IA generativa ao conteúdo da sua empresa.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou abra novamente a [portal do Azure](https://portal.azure.com?azure-portal=true) em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
