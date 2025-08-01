---
lab:
  title: Analisar e depurar seu aplicativo de IA generativa com rastreamento
  description: Saiba como depurar seu aplicativo de IA generativa rastreando seu fluxo de trabalho da entrada do usuário até a resposta do modelo e o pós-processamento.
---

# Analisar e depurar seu aplicativo de IA generativa com rastreamento

Este exercício levará aproximadamente **30** minutos.

> **Observação**: este exercício pressupõe alguma familiaridade com a Fábrica de IA do Azure, e é por isso que algumas instruções são intencionalmente menos detalhadas para incentivar uma exploração mais ativa e o aprendizado prático.

## Introdução

Neste exercício, você executará um assistente de IA generativa de várias etapas que recomenda caminhadas e sugere equipamentos para atividades ao ar livre. Você usará os recursos de rastreamento do SDK de Inferência de IA do Azure para analisar como seu aplicativo é executado e identificar os principais pontos de decisão feitos pelo modelo e pela lógica circundante.

Você interagirá com um modelo implantado para simular uma jornada real do usuário, rastrear cada estágio do aplicativo, desde a entrada do usuário até a resposta do modelo até o pós-processamento e exibir os dados de rastreamento na Fábrica de IA do Azure. Isso ajudará você a entender como o rastreamento aprimora a observabilidade, simplifica a depuração e dá suporte à otimização de desempenho de aplicativos de IA generativa.

## Configurar o ambiente

Para concluir as tarefas neste exercício, será necessário:

- Um hub da Fábrica de IA do Azure,
- Um projeto da Fábrica de IA do Azure,
- Um modelo implantado (como GPT-4o),
- Um recurso do Application Insights conectado.

### Implantar um modelo em um projeto da Fábrica de IA do Azure

Para configurar rapidamente um projeto da Fábrica de IA do Azure, instruções simples de uso da interface de usuário do portal da Fábrica de IA do Azure são fornecidas abaixo.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure.
1. Na home page, na seção **Explorar modelos e recursos**, pesquise pelo modelo `gpt-4o`, que usaremos em nosso projeto.
1. Nos resultados da pesquisa, selecione o modelo **gpt-4o** para ver os detalhes e, na parte superior da página do modelo, clique em **Usar este modelo**.
1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.
1. Selecione **Personalizar** e especifique as seguintes configurações para o seu projeto:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *Selecione qualquer **Local compatível com os Serviços de IA***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto, incluindo a implantação do modelo gpt-4 selecionado.
1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto.
1. Na área **Pontos de extremidade e chaves**, verifique se a biblioteca da **Fábrica de IA do Azure** está selecionada e visualize o **ponto de extremidade do projeto da Fábrica de IA do Azure**.
1. **Salve** o ponto de extremidade em um bloco de notas. Você usará esse ponto de extremidade para se conectar ao projeto em um aplicativo cliente.

### Conectar ao Application Insights

Conecte o Application Insights ao seu projeto na Fábrica de IA do Azure para iniciar os dados coletados para análise.

1. Use o menu à esquerda e selecione a página **Rastreamento**.
1. **Crie um novo** recurso do Application Insights para se conectar ao seu aplicativo.
1. Insira um nome de recurso do Application Insights e selecione **Criar**.

O Application Insights agora está conectado ao seu projeto, e os dados começarão a ser coletados para análise.

## Executar um aplicativo de IA generativa com o Cloud Shell

Você se conectará ao seu projeto da Fábrica de IA do Azure do Azure Cloud Shell e interagirá programaticamente com um modelo implantado como parte de um aplicativo de IA generativo.

### Interagir com um modelo implantado

Comece recuperando as informações necessárias a serem autenticadas para interagir com seu modelo implantado. Em seguida, você acessará o Azure Cloud Shell e atualizará o código do seu aplicativo de IA generativa.

1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente).
1. Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.
1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.
1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para Versão Clássica**.

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira e execute os seguintes comandos:

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Esse comando clona o repositório GitHub que contém os arquivos de código para este exercício.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-genaiops/Files/08
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Digite o seguinte comando para abrir o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código:

    1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do projeto (copiado da página **Visão Geral** no portal da Fábrica de IA do Azure).
    1. Substitua o espaço reservado **your_model_deployment** pelo nome que você atribuiu à sua implantação do modelo GPT-4o (por padrão`gpt-4o`).

1. *Após* substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **clique com o botão direito > Salvar** para **salvar suas alterações** e, em seguida, use o comando **CTRL+Q** ou **clique com o botão direito > Sair** para fechar o editor de código mantendo a linha de comando do Cloud Shell aberta.

### Atualizar o código do seu aplicativo de IA generativa

Agora que seu ambiente está configurado e seu arquivo .env está configurado, é hora de preparar seu script do assistente de IA para execução. Além de se conectar a um projeto de IA e habilitar o Application Insights, você precisa:

- Interagir com um modelo implantado
- Definir a função para especificar seu prompt.
- Defina o fluxo principal que chama todas as funções.

Você adicionará essas três partes a um script inicial.

1. Execute o seguinte comando para **abrir o script** fornecido:

    ```
   code start-prompt.py
    ```

    Você verá que várias linhas-chave foram deixadas em branco ou marcadas com # Comentários vazios. Sua tarefa é concluir o script copiando e colando as linhas corretas abaixo nos locais apropriados.

1. No script, localize **Nº da função para chamar o modelo e manipular o rastreamento**.
1. Cole o seguinte código abaixo desse comentário :

    ```
   # Function to call the model and handle tracing
   def call_model(system_prompt, user_prompt, span_name):
       with tracer.start_as_current_span(span_name) as span:
           span.set_attribute("session.id", SESSION_ID)
           span.set_attribute("prompt.user", user_prompt)
           start_time = time.time()
    
           response = chat_client.chat.completions.create(
               model=model_deployment,
               messages=[
                   { 
                       "role": "system", 
                       "content": system_prompt 
                   },
                   { 
                       "role": "user", 
                       "content": user_prompt
                   }
               ]
           )
    
           duration = time.time() - start_time
           output = response.choices[0].message.content
           span.set_attribute("response.time", duration)
           span.set_attribute("response.tokens", len(output.split()))
           return output
    ```

1. No script, localize **Nº da função para recomendar uma caminhada com base nas preferências do usuário**.
1. Cole o seguinte código abaixo desse comentário :

    ```
   # Function to recommend a hike based on user preferences 
   def recommend_hike(preferences):
        with tracer.start_as_current_span("recommend_hike") as span:
            prompt = f"""
            Recommend a named hiking trail based on the following user preferences.
            Provide only the name of the trail and a one-sentence summary.
            Preferences: {preferences}
            """
            response = call_model(
                "You are an expert hiking trail recommender.",
                prompt,
                "recommend_model_call"
            )
            span.set_attribute("hike_recommendation", response.strip())
            return response.strip()
    ```

1. No script, localize **# ---- Fluxo Principal ----**.
1. Cole o seguinte código abaixo desse comentário :

    ```
   if __name__ == "__main__":
       with tracer.start_as_current_span("trail_guide_session") as session_span:
           session_span.set_attribute("session.id", SESSION_ID)
           print("\n--- Trail Guide AI Assistant ---")
           preferences = input("Tell me what kind of hike you're looking for (location, difficulty, scenery):\n> ")

           hike = recommend_hike(preferences)
           print(f"\n✅ Recommended Hike: {hike}")

           # Run profile function


           # Run match product function


           print(f"\n🔍 Trace ID available in Application Insights for session: {SESSION_ID}")
    ```

1. **Salve as alterações** feitas no script.
1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
   az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.
    
1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python start-prompt.py
    ```

1. Dê uma descrição do tipo de caminhada que você está procurando, por exemplo:

    ```
   A one-day hike in the mountains
    ```

    O modelo gerará uma resposta, que será capturada com o Application Insights. Você pode visualizar os rastreamentos no portal** do Portal da Fábrica de IA do Azure**.

> **Observação**: pode levar alguns minutos para que os dados de monitoramento sejam exibidos no Azure Monitor.

## Visualize os dados de rastreamentos no Portal da Fábrica de IA do Azure

Depois de executar o script, você capturou um rastreamento da execução do aplicativo de IA. Agora você vai explorá-lo usando o Application Insights na Fábrica de IA do Azure.

> **Observação:** posteriormente, você executará o código novamente e exibirá os rastreamentos no portal da Portal do Fábrica de IA do Azure novamente. Vamos primeiro explorar onde encontrar os rastreamentos para visualizá-los.

### Navegue até o Portal da Fábrica de IA do Azure.

1. **Mantenha seu Cloud Shell aberto.** Você voltará a isso para atualizar o código e executá-lo novamente.
1. Navegue até a guia em seu navegador com o portal **Portal da Fábrica de IA do Azure** aberto.
1. Use o menu à esquerda, selecione **Rastreamento**.
1. sentado*Se* nenhum dado for mostrado, **atualize** sua visualização.
1. Selecione o **train_guide_session** de rastreamento para abrir uma nova janela que mostra mais detalhes.

### Examinar seu rastreamento

Esta exibição apresenta o rastreamento de uma sessão completa do Trail Guide AI Assistant.

- **Intervalo de nível superior** : trail_guide_session Este é o intervalo pai. Ele representa toda a execução do seu assistente do início ao fim.

- **Intervalos filho aninhados**: cada linha recuada representa uma operação aninhada. Você encontrará:

    - **recommend_hike** que captura sua lógica para decidir sobre uma caminhada.
    - **recommend_model_call** que é o intervalo criado por call_model() dentro de recommend_hike.
    - **chat gpt-4o,** que é instrumentado automaticamente pelo SDK de Inferência de IA do Azure para mostrar a interação real do LLM.

1. Você pode clicar em qualquer extensão para visualizar:

    1. Sua duração.
    1. Seus atributos como prompt do usuário, tokens usados, tempo de resposta.
    1. Quaisquer erros ou dados personalizados anexados com **span.set_attribute(...)**.

## Adicionar mais funções ao seu código

1. Navegue até a guia no navegador com o **Portal do Azure** aberto.
1. Execute o comando a seguir para **reabrir o script:**

    ```
   code start-prompt.py
    ```

1. No script, localize **Nº da função para gerar um perfil de viagem para a caminhada recomendada**.
1. Cole o seguinte código abaixo desse comentário :

    ```
   def generate_trip_profile(hike_name):
       with tracer.start_as_current_span("trip_profile_generation") as span:
           prompt = f"""
           Hike: {hike_name}
           Respond ONLY with a valid JSON object and nothing else.
           Do not include any intro text, commentary, or markdown formatting.
           Format: {{ "trailType": ..., "typicalWeather": ..., "recommendedGear": [ ... ] }}
           """
           response = call_model(
               "You are an AI assistant that returns structured hiking trip data in JSON format.",
               prompt,
               "trip_profile_model_call"
           )
           print("🔍 Raw model response:", response)
           try:
               profile = json.loads(response)
               span.set_attribute("profile.success", True)
               return profile
           except json.JSONDecodeError as e:
               print("❌ JSON decode error:", e)
               span.set_attribute("profile.success", False)
               return {}
    ```

1. No script, localize **Nº da função para combinar o equipamento recomendado com os produtos no catálogo**.
1. Cole o seguinte código abaixo desse comentário :

    ```
   def match_products(recommended_gear):
       with tracer.start_as_current_span("product_matching") as span:
           matched = []
           for gear_item in recommended_gear:
               for product in mock_product_catalog:
                   if any(word in product.lower() for word in gear_item.lower().split()):
                       matched.append(product)
                       break
           span.set_attribute("matched.count", len(matched))
           return matched
    ```

1. No script, localize **Nº da Execução da função de perfil** .
1. Abaixo e **alinhado com** este comentário, cole o seguinte código:

    ```
           profile = generate_trip_profile(hike)
           if not profile:
               print("Failed to generate trip profile. Please check Application Insights for trace.")
               exit(1)

           print(f"\n📋 Trip Profile for {hike}:")
           print(json.dumps(profile, indent=2))
    ```

1. No script, localize **Nº da execução da função de produto correspondente**.
1. Abaixo e **alinhado com** este comentário, cole o seguinte código:

    ```
           matched = match_products(profile.get("recommendedGear", []))
           print("\n🛒 Recommended Products from Lakeshore Retail:")
           print("\n".join(matched))
    ```

1. **Salve as alterações** feitas no script.
1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para **executar o aplicativo**:

    ```
   python start-prompt.py
    ```

1. Dê uma descrição do tipo de caminhada que você está procurando, por exemplo:

    ```
   I want to go for a multi-day adventure along the beach
    ```

<br>
<details>
<summary><b>Script da solução</b>: Caso seu código não esteja funcionando.</summary><br>
<p>Se você inspecionar o rastreamento LLM quanto à função generate_trip_profile, observará que a resposta do assistente inclui acentos graves e a palavra json para formatar a saída como um bloco de código.

Embora isso seja útil para exibição, causa problemas no código porque a saída não é mais JSON válida. Isso leva a um erro de análise durante o processamento posterior.

O erro provavelmente é causado pela forma como o LLM é instruído a aderir a um formato específico para sua saída. Incluir as instruções no prompt do usuário parece mais eficaz do que colocá-lo no prompt do sistema.</p>
</details>


> **Observação**: pode levar alguns minutos para que os dados de monitoramento sejam exibidos no Azure Monitor.

### Exibir os novos rastreamentos no Portal da Fábrica de IA do Azure

1. Navegue de volta até o Portal da Fábrica de IA do Azure.
1. Um novo rastreamento com o mesmo nome **trail_guide_session** deve aparecer. Atualize sua exibição, se necessário.
1. Selecione o novo rastreamento para abrir a exibição mais detalhada.
1. Examine os novos intervalos filho aninhados **trip_profile_generation** e **product_matching**.
1. Selecione **product_matching** e revise os metadados exibidos.

    Na função product_matching, você incluiu **span.set_attribute("matched.count", len(matched)).** Ao definir o atributo com o par chave-valor **matched. Count** e o comprimento da variável correspondida, você adicionou essas informações ao rastreamento **product_matching** . Você pode encontrar esse par de chave-valor em **atributos** nos metadados.

## (OPCIONAL) Rastrear um erro

Se você tiver tempo extra, poderá revisar como usar rastreamentos quando tiver um erro. Um script que provavelmente gerará um erro é fornecido a você. Execute-o e revise os rastreamentos.

Este é um exercício projetado para desafiá-lo, o que significa que as instruções são intencionalmente menos detalhadas.

1. No Cloud Shell, abra o **script error-prompt.py** . Esse script está localizado no mesmo diretório que o **script start-prompt.py** . Revise seu conteúdo.
1. Execute o script **error-prompt.py**. Forneça uma resposta na linha de comando quando solicitado.
1. *Felizmente*, a mensagem de saída inclui **Falha ao gerar o perfil de viagem. Verifique o Application Insights quanto a rastreamento.**
1. Navegue até o rastreamento do **trip_profile_generation** e inspecione por que houve um erro.

<br>
<details>
<summary><b>Obtenha a resposta sobre</b>: Por que você pode ter encontrado um erro...</summary><br>
<p>Se você inspecionar o rastreamento LLM quanto à função generate_trip_profile, observará que a resposta do assistente inclui acentos graves e a palavra json para formatar a saída como um bloco de código.

Embora isso seja útil para exibição, causa problemas no código porque a saída não é mais JSON válida. Isso leva a um erro de análise durante o processamento posterior.

O erro provavelmente é causado pela forma como o LLM é instruído a aderir a um formato específico para sua saída. Incluir as instruções no prompt do usuário parece mais eficaz do que colocá-lo no prompt do sistema.</p>
</details>

## Onde encontrar outros laboratórios

Você pode explorar laboratórios e exercícios adicionais no [Portal de Aprendizagem da Fábrica de IA do Azure](https://ai.azure.com) ou consultara seção** de laboratório do **curso para outras atividades disponíveis.
