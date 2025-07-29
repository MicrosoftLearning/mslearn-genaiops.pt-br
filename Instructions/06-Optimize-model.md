---
lab:
  title: Otimizar o modelo usando um conjunto de dados sintético
  description: Saiba como criar conjuntos de dados sintéticos e usá-los para melhorar o desempenho e a confiabilidade do seu modelo.
---

## Otimizar o modelo usando um conjunto de dados sintético

A otimização de um aplicativo de IA generativa envolve utilizar conjuntos de dados para aprimorar o desempenho e a confiabilidade do modelo. Ao usar dados sintéticos, os desenvolvedores podem simular uma ampla variedade de cenários e casos extremos que podem não estar presentes nos dados reais. Além disso, a avaliação dos resultados do modelo é crucial para se obter aplicativos de IA confiáveis e de alta qualidade. Todo o processo de otimização e avaliação pode ser gerenciado com eficiência usando o SDK de Avaliação de IA do Azure, que fornece ferramentas e estruturas robustas para simplificar essas tarefas.

Este exercício levará aproximadamente **30** minutos\*

> \* Essa estimativa não inclui a tarefa opcional ao final do exercício.
## Cenário

Imagine que você deseja criar um aplicativo de guia inteligente da plataforma IA para melhorar as experiências dos visitantes em um museu. O aplicativo visa responder a perguntas sobre figuras históricas. Para avaliar as respostas do aplicativo, você precisa criar um conjunto de dados sintético abrangente de perguntas e respostas que abranja vários aspectos dessas personalidades e seu trabalho.

Você selecionou um modelo GPT-4 para fornecer respostas generativas. Agora você deseja montar um simulador que gere interações contextualmente relevantes, avaliando o desempenho da IA em diferentes cenários.

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
    cd ~/mslearn-genaiops/Files/06/
     ```

1. Insira os seguintes comandos para ativar um ambiente virtual e instalar as bibliotecas necessárias:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-evaluation azure-ai-projects promptflow wikipedia aiohttp openai==1.77.0
    ```

1. Digite o seguinte comando para abrir o arquivo de configuração que foi fornecido:

    ```powershell
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua os espaços reservados **your_azure_openai_service_endpoint** e **your_azure_openai_service_api_key** pelos valores do ponto de extremidade e da chave que você copiou anteriormente.
1. *Após* substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **clique com o botão direito > Salvar** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** ou **clique com o botão direito > Sair** para fechar o editor de código mantendo a linha de comando do Cloud Shell aberta.

## Gerar dados sintéticos

Agora você vai executar um script que gera um conjunto de dados sintético e o usa para avaliar a qualidade do seu modelo pré-treinado.

1. Execute o seguinte comando para **editar o script** fornecido:

    ```powershell
   code generate_synth_data.py
    ```

1. No script, localize **# Definir função de retorno de chamada**.
1. Cole o seguinte código abaixo desse comentário :

    ```
    async def callback(
        messages: List[Dict],
        stream: bool = False,
        session_state: Any = None,  # noqa: ANN401
        context: Optional[Dict[str, Any]] = None,
    ) -> dict:
        messages_list = messages["messages"]
        # Get the last message
        latest_message = messages_list[-1]
        query = latest_message["content"]
        context = text
        # Call your endpoint or AI application here
        current_dir = os.getcwd()
        prompty_path = os.path.join(current_dir, "application.prompty")
        _flow = load_flow(source=prompty_path)
        response = _flow(query=query, context=context, conversation_history=messages_list)
        # Format the response to follow the OpenAI chat protocol
        formatted_response = {
            "content": response,
            "role": "assistant",
            "context": context,
        }
        messages["messages"].append(formatted_response)
        return {
            "messages": messages["messages"],
            "stream": stream,
            "session_state": session_state,
            "context": context
        }
    ```

    Você pode usar qualquer ponto de extremidade do aplicativo para simular, especificando uma função de retorno de chamada de destino. Nesse caso, você irá usar um aplicativo que é um LLM com um arquivo prompty `application.prompty`. A função de retorno de chamada acima processa cada mensagem gerada pelo simulador executando as seguintes tarefas:
    * Recupera a mensagem de usuário mais recente.
    * Carrega um prompt flow do application.prompty.
    * Gera uma resposta usando o prompt flow.
    * Formata a resposta para aderir ao protocolo de chat do OpenAI.
    * Acrescenta a resposta do assistente à lista de mensagens.

    >**Observação**: Para obter mais informações sobre como usar o prompty, consulte a [documentação do prompty](https://www.prompty.ai/docs).

1. Em seguida, localize **# Executar o simulador**.
1. Cole o seguinte código abaixo desse comentário :

    ```
    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
    }
    
    simulator = Simulator(model_config=model_config)
    
    outputs = asyncio.run(simulator(
        target=callback,
        text=text,
        num_queries=1,  # Minimal number of queries
    ))
    
    output_file = "simulation_output.jsonl"
    with open(output_file, "w") as file:
        for output in outputs:
            file.write(output.to_eval_qr_json_lines())
    ```

   O código acima irá inicializar o simulador e o executar para gerar conversas sintéticas com base em um texto extraído anteriormente da Wikipédia.

1. Em seguida, localize **# Avaliar o modelo**.
1. Cole o seguinte código abaixo desse comentário :

    ```
    groundedness_evaluator = GroundednessEvaluator(model_config=model_config)
    eval_output = evaluate(
        data=output_file,
        evaluators={
            "groundedness": groundedness_evaluator
        },
        output_path="groundedness_eval_output.json"
    )
    ```

    Agora que você tem um conjunto de dados, pode avaliar a qualidade e a eficácia do aplicativo de IA generativa. No código acima, você irá usar a base como sua métrica de qualidade.

1. Salve suas alterações.
1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para **executar o aplicativo**:

    ```
   python generate_synth_data.py
    ```

    Depois que o script for concluído, você poderá baixar os arquivos de saída executando `download simulation_output.jsonl` e `download groundedness_eval_output.json` e examinando seu conteúdo. Se a métrica de fundamentação não estiver próxima de 3,0, você pode alterar os parâmetros do LLM, como `temperature`, `top_p`, `presence_penalty` ou `frequency_penalty` no arquivo `application.prompty` e executar novamente o script para gerar um novo conjunto de dados para avaliação. Você também pode alterar o `wiki_search_term` para obter um conjunto de dados sintético com base em um contexto diferente.

## (OPCIONAL) Ajustar seu modelo

Se você tiver tempo extra, pode usar o conjunto de dados gerado para ajustar seu modelo na Fábrica de IA do Azure. O ajuste depende dos recursos de infraestrutura em nuvem, que podem levar um tempo variável para provisionar, dependendo da capacidade do data center e da demanda simultânea.

1. Abra uma nova aba no navegador, acesse o [portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre com suas credenciais do Azure.
1. Na home page da Fábrica de IA, selecione o projeto que você criou no início do exercício.
1. Navegue até a página **Ajustar** na seção **Criar e personalizar**, usando o menu à esquerda.
1. Selecione o botão para adicionar um novo modelo de ajuste fino, selecione o modelo **gpt-4o** e, em seguida, selecione **Avançar**.
1. **Ajuste** o modelo usando a seguinte configuração:
    - **Versão do modelo**: *selecione a versão padrão*
    - **Método de personalização**: Supervisionado
    - **Sufixo do modelo**: `ft-travel`
    - **Recurso de IA conectado**: *selecione a conexão que foi criada quando você criou seu hub. Já deve estar selecionado por padrão.*
    - **Dados de treinamento**: carregar arquivos

    <details>  
    <summary><b>Dica de solução de problemas</b>: erro de permissões</summary>
    <p>Se você receber um erro de permissões, tente o seguinte para solucionar o problema:</p>
    <ul>
        <li>No portal do Azure, selecione o recurso Serviços de IA.</li>
        <li>Na guia Identidade, em Gerenciamento de recursos, confirme se é uma identidade gerenciada atribuída pelo sistema.</li>
        <li>Navegue até a conta de armazenamento associada. Na página do IAM, adicione a atribuição de função <em>Proprietário de Dados do Blob de Armazenamento</em>.</li>
        <li>Em <strong>Atribuir acesso a</strong>, escolha <strong>Identidade Gerenciada</strong>, <strong>+Selecionar membros</strong>, selecione <strong>Todas as identidades gerenciadas atribuídas pelo sistema</strong> e selecione o recurso dos Serviços de IA do Azure.</li>
        <li>Revise e atribua para salvar as novas configurações e repita a etapa anterior.</li>
    </ul>
    </details>

    - **Carregar arquivo**: escolha o arquivo JSONL que você baixou na etapa anterior.
    - **Dados de validação**: nenhum
    - **Parâmetros da tarefa**: *mantenha as configurações padrão*
1. O ajuste será iniciado e pode levar algum tempo para ser concluído.

    > **Observação**: o ajuste e a implantação podem levar um tempo considerável (30 minutos ou mais), portanto, talvez seja necessário verificar periodicamente. Você pode ver mais detalhes do progresso até agora selecionando o trabalho de modelo de ajuste fino e exibindo a guia **Logs**.

## (OPCIONAL) Implantar o modelo ajustado

Quando o ajuste for concluído, você poderá implantar o modelo ajustado.

1. Selecione o link do trabalho de ajuste fino para abrir sua página de detalhes. Depois, selecione a guia **Métricas** e explore as métricas de ajuste.
1. Implante o modelo ajustado com as seguintes configurações:
    - **Nome da implantação**: *Um nome válido para a implantação de modelo*
    - **Tipo de implantação**: Padrão
    - **Limite de taxa de fichas por minuto (milhares)**: 5 mil *(ou o máximo disponível na sua assinatura, se for menos que 5 mil)
    - **Filtro de conteúdo**: Padrão
1. Aguarde a conclusão da implantação antes de testá-la, isso pode demorar um pouco. Verifique o **estado de provisionamento** até que ele seja bem-sucedido (talvez seja necessário atualizar o navegador para ver o status atualizado).
1. Quando a implantação estiver pronta, navegue até o modelo ajustado e selecione **Abrir no playground**.

    Agora que você implantou seu modelo ajustado, pode testá-lo no ambiente de playground de Chat, assim como faria com qualquer modelo base.

## Conclusão

Neste exercício, você criou um conjunto de dados sintético simulando uma conversa entre um usuário e um aplicativo de conclusão de chat. Usando este conjunto de dados, você pode avaliar a qualidade das respostas do seu aplicativo e ajustá-lo para alcançar os resultados desejados.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou abra novamente a [portal do Azure](https://portal.azure.com?azure-portal=true) em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
