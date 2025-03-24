---
lab:
  title: Comparar modelos de linguagem do catálogo de modelos
---

## Comparar modelos de linguagem do catálogo de modelos

Depois de definir seu caso de uso, você pode usar o catálogo de modelos para explorar se um modelo de IA resolverá seu problema. Você pode usar o catálogo de modelos para selecionar modelos a serem implantados, que podem ser comparados para explorar qual modelo atende melhor às suas necessidades.

Neste exercício, você comparará dois modelos de linguagem por meio do catálogo de modelos no portal da Fábrica de IA do Azure.

Este exercício levará aproximadamente **25** minutos.

## Cenário

Imagine que você queira criar um aplicativo para ajudar os alunos a aprender a codificar em Python. No aplicativo, você quer um tutor automatizado que possa ajudar os alunos a escrever e avaliar o código. Em um exercício, os alunos precisam criar o código Python necessário para plotar um gráfico de pizza, com base na seguinte imagem de exemplo:

![Gráfico de pizza mostrando as notas obtidas em um exame com as seções matemática (34,9%), física (28,6%), química (20,6%) e inglês (15,9%)](./images/demo.png)

Você precisa selecionar um modelo de linguagem que aceite imagens como entrada e seja capaz de gerar código preciso. Os modelos disponíveis que atendem a esses critérios são GPT-4 Turbo, GPT-4o e GPT-4o mini.

Vamos começar implantando os recursos necessários para trabalhar com esses modelos no portal da Fábrica de IA do Azure.

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

## Comparar os modelos

Você sabe que há três modelos que aceitam imagens como entrada cuja infraestrutura de inferência é totalmente gerenciada pelo Azure. Agora, você precisa compará-los para decidir qual é o ideal para o nosso caso de uso.

1. Em um navegador da Web, abra o [portal do Azure IA Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure.
1. Se solicitado, selecione o projeto de IA criado anteriormente.
1. Navegue até a página **Catálogo de modelos**, usando o menu à esquerda.
1. Selecione **Comparar modelos** (encontre o botão ao lado dos filtros no painel de pesquisa).
1. Remova os modelos selecionados.
1. Um por um, adicione os três modelos que você quer comparar: **gpt-4**, **gpt-4o** e **gpt-4o-mini**. Para o **gpt-4**, certifique-se de que a versão selecionada seja **turbo-2024-04-09**, pois é a única versão que aceita imagens como entrada.
1. Altere o eixo x para **Precisão**.
1. Verifique se o eixo y está definido como **Custo**.

Revise o gráfico e tente responder às seguintes perguntas:

- *Qual modelo é mais preciso?*
- *Qual modelo é mais barato de usar?*

A precisão da métrica do parâmetro de comparação é calculada com base em conjuntos de dados genéricos disponíveis publicamente. A partir do gráfico, já podemos filtrar um dos modelos, pois ele tem o maior custo por token, mas não a maior precisão. Antes de tomar uma decisão, vamos explorar a qualidade das saídas dos dois modelos restantes específicos para seu caso de uso.

## Configurar seu ambiente de desenvolvimento

Para experimentar e iterar rapidamente, você usará um notebook com código Python no VS (Visual Studio) Code. Vamos deixar o VS Code pronto para uso na criação de ideias locais.

1. Abra o VS Code e **clone** o seguinte repositório Git: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Armazene o clone em uma unidade local e abra a pasta após a clonagem.
1. No Explorador do VS Code (painel esquerdo), abra o notebook **02-Compare-models.ipynb** na pasta **Files/02**.
1. Execute todas as células no notebook.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou abra novamente a [portal do Azure](https://portal.azure.com?azure-portal=true) em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
