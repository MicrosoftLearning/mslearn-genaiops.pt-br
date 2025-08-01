---
lab:
  title: Explorar a engenharia rápida com o Prompty
  description: 'Aprenda a usar o prompty para testar rapidamente e aprimorar diferentes prompts com seu modelo de linguagem, garantindo que eles sejam construídos e orquestrados para obter os melhores resultados.'
---

## Explorar a engenharia rápida com o Prompty

Este exercício levará, aproximadamente, **45** minutos.

> **Observação**: este exercício pressupõe alguma familiaridade com a Fábrica de IA do Azure, e é por isso que algumas instruções são intencionalmente menos detalhadas para incentivar uma exploração mais ativa e o aprendizado prático.

## Introdução

Durante a ideação, você quer testar e melhorar rapidamente diferentes prompts com o seu modelo de linguagem. Há várias maneiras de abordar a engenharia de prompt: por meio do playground no portal da Fábrica de IA do Azure ou usando o Prompty para uma abordagem que priorize mais o código.

Neste exercício, você irá explorar a engenharia de prompt com o Prompty no Azure Cloud Shell usando um modelo implantado por meio da Fábrica de IA do Azure.

## Configurar o ambiente

Para concluir as tarefas neste exercício, será necessário:

- Um hub da Fábrica de IA do Azure,
- Um projeto da Fábrica de IA do Azure,
- Um modelo implantado (como GPT-4o).

### Criar um projeto e hub de IA do Azure

> **Observação**: Se você já tiver um projeto de IA do Azure, poderá ignorar este procedimento e usar seu projeto existente.

Você pode criar um projeto de IA do Azure manualmente por meio do portal da Fábrica de IA do Azure, bem como implantar o modelo usado no exercício. No entanto, você também pode automatizar esse processo usando um aplicativo de modelo com o [Azure Developer CLI (azd)](https://aka.ms/azd).

1. Em um navegador da Web, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre usando suas credenciais do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

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

1. Em seguida, insira o comando a seguir para executar o modelo Starter. Ele provisionará um Hub de IA com recursos dependentes, projeto de IA, serviços de IA e um ponto de extremidade online.

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copie esses valores, pois eles serão usados posteriormente.

### Configurar seu ambiente virtual no Cloud Shell

Para experimentar e iterar rapidamente, você usará um conjunto de scripts Python no Cloud Shell.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para navegar até a pasta com os arquivos de código usados neste exercício:

     ```powershell
    cd ~/mslearn-genaiops/Files/03/
     ```

1. Insira os seguintes comandos para ativar um ambiente virtual e instalar as bibliotecas necessárias:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai tiktoken azure-ai-projects prompty[azure]
    ```

1. Digite o seguinte comando para abrir o arquivo de configuração que foi fornecido:

    ```powershell
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua os espaços reservados **ENDPOINTNAME** e **APIKEY** pelos valores do ponto de extremidade e da chave que você copiou anteriormente.
1. *Após* substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **clique com o botão direito > Salvar** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** ou **clique com o botão direito > Sair** para fechar o editor de código mantendo a linha de comando do Cloud Shell aberta.

## Otimize o prompt do sistema

Minimizar o comprimento dos prompts do sistema e manter a funcionalidade na IA generativa é fundamental para implantações em larga escala. Prompts mais curtos podem levar a tempos de resposta mais rápidos, pois o modelo de IA processa menos tokens e também usa menos recursos computacionais.

1. Digite o seguinte comando para abrir o arquivo de aplicativo que foi fornecido:

    ```powershell
   code optimize-prompt.py
    ```

    Revise o código e observe que o script executa o arquivo de modelo `start.prompty` que já tem um prompt do sistema predefinido.

1. Execute `code start.prompty` para examinar o prompt do sistema. Considere como você pode encurtá-lo mantendo sua intenção clara e eficaz. Por exemplo:

   ```python
   original_prompt = "You are a helpful assistant. Your job is to answer questions and provide information to users in a concise and accurate manner."
   optimized_prompt = "You are a helpful assistant. Answer questions concisely and accurately."
   ```

   Remova palavras redundantes e concentre-se nas instruções essenciais. Salve o prompt otimizado no arquivo.

### Testar e validar sua otimização

Testar as alterações de prompt é importante para garantir que você reduza o uso de tokens sem perder a qualidade.

1. Execute `code token-count.py` para abrir e examinar o aplicativo de contador de tokens fornecido no exercício. Se você usou um prompt otimizado diferente do fornecido no exemplo acima, também poderá usá-lo neste aplicativo.

1. Execute o script com `python token-count.py` e observe a diferença na contagem de tokens. Verifique se o prompt otimizado ainda produz respostas de alta qualidade.

## Analisar as interações do usuário

Entender como os usuários interagem com seu aplicativo ajuda a identificar padrões que aumentam o uso de tokens.

1. Analise um conjunto de dados de exemplo com prompts de usuários:

    - **"Resuma o enredo de *Guerra e Paz*."**
    - **"Quais são algumas curiosidades sobre gatos?"**
    - **"Escreva um plano de negócios detalhado para uma startup que usa IA para otimizar cadeias de suprimentos."**
    - **"Traduza 'Olá, como vai você?' para o francês."**
    - **"Explique o emaranhamento quântico para uma criança de 10 anos."**
    - **"Me dê 10 ideias criativas para um conto de ficção científica."**

    Para cada um, identifique se é provável que resulte em uma resposta **curta**, **média** ou **longa/complexa** da IA.

1. Revise suas categorizações. Que padrões você percebe? Considere:

    - O **nível de abstração** (por exemplo, criativo versus factual) afeta o comprimento da resposta?
    - Os **prompts abertos** tendem a gerar respostas mais longas?
    - Como a **complexidade instrucional** (por exemplo, “explique como se eu tivesse 10 anos”) influencia a resposta?

1. Insira o seguinte comando para executar o aplicativo **optimize-prompt**:

    ```
   python optimize-prompt.py
    ```

1. Use alguns dos exemplos fornecidos acima para verificar sua análise.
1. Agora, use o seguinte prompt de formato longo e analise sua resposta:

    ```
   Write a comprehensive overview of the history of artificial intelligence, including key milestones, major contributors, and the evolution of machine learning techniques from the 1950s to today.
    ```

1. Reescreva este prompt para:

    - Limitar o escopo
    - Definir expectativas para brevidade
    - Usar formatação ou estrutura para orientar a resposta

1. Compare as respostas para verificar se você obteve uma resposta mais concisa.

> **OBSERVAÇÃO**: Você pode usar `token-count.py` para comparar o uso de tokens em ambas as respostas.
<br>
<details>
<summary><b>Exemplo de um prompt reescrito:</b></summary><br>
<p>“Faça um resumo em tópicos dos 5 principais marcos da história da IA.”</p>
</details>

## [**OPCIONAL**] Aplicar suas otimizações em um cenário real

1. Imagine que você esteja criando um chatbot de suporte ao cliente que deve fornecer respostas rápidas e precisas.
1. Integre seu prompt do sistema otimizado e o modelo ao código do chatbot (*você pode usar `optimize-prompt.py` como ponto de partida*).
1. Teste o chatbot com várias consultas de usuários para garantir que ele responda de forma eficiente e eficaz.

## Conclusão

A otimização de prompts é uma habilidade fundamental para reduzir os custos e melhorar o desempenho em aplicativos de IA generativa. Ao reduzir os prompts, usar modelos e analisar as interações do usuário, você pode criar soluções mais eficientes e escalonáveis.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou abra novamente a [portal do Azure](https://portal.azure.com?azure-portal=true) em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
