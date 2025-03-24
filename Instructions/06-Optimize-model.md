---
lab:
  title: Otimizar o modelo usando um conjunto de dados sintético
---

## Otimizar o modelo usando um conjunto de dados sintético

A otimização de um aplicativo de IA generativa envolve utilizar conjuntos de dados para aprimorar o desempenho e a confiabilidade do modelo. Ao usar dados sintéticos, os desenvolvedores podem simular uma ampla variedade de cenários e casos extremos que podem não estar presentes nos dados reais. Além disso, a avaliação dos resultados do modelo é crucial para se obter aplicativos de IA confiáveis e de alta qualidade. Todo o processo de otimização e avaliação pode ser gerenciado com eficiência usando o SDK de Avaliação de IA do Azure, que fornece ferramentas e estruturas robustas para simplificar essas tarefas.

## Cenário

Imagine que você deseja criar um aplicativo de guia inteligente da plataforma IA para melhorar as experiências dos visitantes em um museu. O aplicativo visa responder a perguntas sobre figuras históricas. Para avaliar as respostas do aplicativo, você precisa criar um conjunto de dados sintético abrangente de perguntas e respostas que abranja vários aspectos dessas personalidades e seu trabalho.

Você selecionou um modelo GPT-4 para fornecer respostas generativas. Agora você deseja montar um simulador que gere interações contextualmente relevantes, avaliando o desempenho da IA em diferentes cenários.

Vamos começar implantando os recursos necessários para criar esse aplicativo.

## Criar um projeto e hub de IA do Azure

Você pode criar um hub de IA do Azure e projetar manualmente por meio do portal Fábrica de IA do Azure, bem como implantar os modelos usados no exercício. No entanto, você também pode automatizar esse processo usando um aplicativo de modelo com o [Azure Developer CLI (azd)](https://aka.ms/azd).

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copie esses valores, pois eles serão usados posteriormente.

## Configurar seu ambiente de desenvolvimento

Para experimentar e iterar rapidamente, você usará um notebook com código Python no VS (Visual Studio) Code. Vamos deixar o VS Code pronto para uso na criação de ideias locais.

1. Abra o VS Code e **clone** o seguinte repositório Git: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Armazene o clone em uma unidade local e abra a pasta após a clonagem.
1. No Explorador do VS Code (painel esquerdo), abra o notebook **06-Optimize-your-model.ipynb** na pasta **Files/06**.
1. Execute todas as células no notebook.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou abra novamente a [portal do Azure](https://portal.azure.com?azure-portal=true) em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
