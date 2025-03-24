---
lab:
  title: Explorar a engenharia rápida com o Prompty
---

## Explorar a engenharia rápida com o Prompty

Durante a ideação, você quer testar e melhorar rapidamente diferentes prompts com o seu modelo de linguagem. Há várias maneiras de abordar a engenharia de prompt: por meio do playground no portal da Fábrica de IA do Azure ou usando o Prompty para uma abordagem que priorize mais o código.

Neste exercício, você explorará a engenharia de prompt com o Prompty no Visual Studio Code usando um modelo implantado por meio da Fábrica de IA do Azure.

Este exercício levará aproximadamente **40** minutos.

## Cenário

Imagine que você queira criar um aplicativo para ajudar os alunos a aprender a codificar em Python. No aplicativo, você quer um tutor automatizado que possa ajudar os alunos a escrever e avaliar o código. No entanto, você não quer que o aplicativo de chat simpleste dê todas as respostas. Você quer que os alunos recebam dicas personalizadas que os incentivem a pensar em como proceder.

Você selecionou um modelo GPT-4 para começar a experimentar. Agora você quer aplicar a engenharia de prompt para orientar o comportamento do chat para ser um tutor que gera dicas personalizadas.

Vamos começar implantando os recursos necessários para trabalhar com esse modelo no portal da Fábrica de IA do Azure.

## Criar um projeto e hub de IA do Azure

> **Observação**: se você já tiver um hub e um projeto de IA do Azure, poderá ignorar este procedimento e usar seu projeto existente.

Você pode criar um hub de IA do Azure e projetar manualmente por meio do portal da Fábrica de IA do Azure, bem como implantar o modelo usado no exercício. No entanto, você também pode automatizar esse processo usando um aplicativo de modelo com o [Azure Developer CLI (azd)](https://aka.ms/azd).

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
   
## Configurar seu ambiente de desenvolvimento

Para experimentar e iterar rapidamente, você usará o Prompty no VS (Visual Studio) Code. Vamos deixar o VS Code pronto para uso na criação de ideias locais.

1. Abra o VS Code e **clone** o seguinte repositório Git: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Armazene o clone em uma unidade local e abra a pasta após a clonagem.
1. No painel de extensões do VS Code, pesquise e instale a extensão **Prompty**.
1. No Explorador do VS Code (painel esquerdo), clique com o botão direito do mouse na pasta **Files/03**.
1. Selecione **Novo Prompty** no menu suspenso.
1. Abra o arquivo recém-criado chamado **basic.prompty**.
1. Execute o arquivo Prompty clicando no botão **executar** no canto superior direito (ou pressione F5).
1. Quando solicitado a entrar, selecione **Permitir**.
1. Selecione e entre na sua conta do Azure.
1. Volte para o VS Code, onde um painel **Saída** será aberto com uma mensagem de erro. A mensagem de erro informará que o modelo implantado não está especificado ou não pode ser encontrado.

Para corrigir o erro, você precisa configurar um modelo para o Prompty usar.

## Atualizar metadados de prompt

Para executar o arquivo Prompty, você precisa especificar o modelo de linguagem a ser usado para gerar a resposta. Os metadados são definidos no *frontmatter* do arquivo Prompty. Vamos atualizar os metadados com a configuração do modelo e outras informações.

1. Abra o painel do terminal do Visual Studio Code.
1. Copie o arquivo **basic.prompty** (na mesma pasta) e renomeie a cópia como `chat-1.prompty`.
1. Abra **chat-1.prompty** e atualize os seguintes campos para alterar algumas informações básicas:

    - **Nome:**

        ```yaml
        name: Python Tutor Prompt
        ```

    - **Descrição**:

        ```yaml
        description: A teaching assistant for students wanting to learn how to write and edit Python code.
        ```

    - **Modelo implantado**:

        ```yaml
        azure_deployment: ${env:AZURE_OPENAI_CHAT_DEPLOYMENT}
        ```

1. Em seguida, adicione o seguinte espaço reservado para a chave de API no parâmetro **azure_deployment**.

    - **Chave do ponto de extremidade**:

        ```yaml
        api_key: ${env:AZURE_OPENAI_API_KEY}
        ```

1. Salve o arquivo Prompty atualizado.

O arquivo Prompty agora tem todos os parâmetros necessários, mas alguns parâmetros usam espaços reservados para obter os valores necessários. Os espaços reservados são armazenados no arquivo **.env** na mesma pasta.

## Atualizar configuração do modelo

Para especificar qual modelo o Prompty usa, você precisa fornecer as informações do modelo no arquivo .env.

1. Abra o arquivo **.env** na pasta **Files/03**.
1. Atualize cada um dos espaços reservados com os valores copiados anteriormente da saída da implantação do modelo no Portal do Azure:

    ```yaml
    - AZURE_OPENAI_CHAT_DEPLOYMENT="gpt-4"
    - AZURE_OPENAI_ENDPOINT="<Your endpoint target URI>"
    - AZURE_OPENAI_API_KEY="<Your endpoint key>"
    ```

1. Salve o arquivo .env.
1. Execute o arquivo **chat-1.prompty** novamente.

Agora você receberá uma resposta gerada por IA, embora não relacionada ao seu cenário, pois ela usa apenas a entrada de exemplo. Vamos atualizar o modelo para torná-lo um assistente de ensino de IA.

## Editar a seção de exemplo

A seção de exemplo especifica as entradas para o Prompty e fornece valores padrão a serem usados se nenhuma entrada for fornecida.

1. Edite os campos dos seguintes parâmetros:

    - **firstName**: escolha qualquer outro nome.
    - **context**: eemova esta seção inteira.
    - **pergunta**: substitua o texto fornecido por:

    ```yaml
    What is the difference between 'for' loops and 'while' loops?
    ```

    A seção de **amostra** agora estará assim:
    
    ```yaml
    sample:
    firstName: Daniel
    question: What is the difference between 'for' loops and 'while' loops?
    ```

    1. Execute o arquivo Prompty atualizado e revise a saída.

