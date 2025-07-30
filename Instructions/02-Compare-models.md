---
lab:
  title: Comparar modelos de linguagem do catálogo de modelos
  description: Saiba como comparar e selecionar modelos apropriados para seu projeto de IA generativa.
---

## Comparar modelos de linguagem do catálogo de modelos

Depois de definir seu caso de uso, você pode usar o catálogo de modelos para explorar se um modelo de IA resolverá seu problema. Você pode usar o catálogo de modelos para selecionar modelos a serem implantados, que podem ser comparados para explorar qual modelo atende melhor às suas necessidades.

Neste exercício, você comparará dois modelos de linguagem por meio do catálogo de modelos no portal da Fábrica de IA do Azure.

Este exercício levará aproximadamente **30** minutos.

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

## Comparar os modelos

Você sabe que há três modelos que aceitam imagens como entrada cuja infraestrutura de inferência é totalmente gerenciada pelo Azure. Agora, você precisa compará-los para decidir qual é o ideal para o nosso caso de uso.

1. Em uma nova guia do navegador, abra o [portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure.
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

## Configurar seu ambiente de desenvolvimento no Cloud Shell

Para experimentar e iterar rapidamente, você usará um conjunto de scripts Python no Cloud Shell.

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.
1. Na área **Detalhes do projeto**, observe a **Cadeia de conexão do projeto**.
1. Salve a cadeia de caracteres em um bloco de notas. Você usará essa cadeia de conexão para se conectar ao seu projeto em um aplicativo cliente.
1. De volta à guia do Azure Portal, abra o Cloud Shell, caso o tenha fechado anteriormente, e execute o seguinte comando para navegar até a pasta com os arquivos de código usados neste exercício:

     ```powershell
    cd ~/mslearn-genaiops/Files/02/
     ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai matplotlib
    ```

1. Digite o seguinte comando para abrir o arquivo de configuração que foi fornecido:

    ```powershell
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_connection_string** pela cadeia de conexão do seu projeto (copiada da página **Visão geral** do projeto no portal da Fábrica de IA do Azure). Observe que o primeiro e o segundo modelo usados no exercício são **gpt-4o** e **gpt-4o-mini**, respectivamente.
1. *Após* substituir o espaço reservado, no editor de código, use o comando **CTRL+S** ou **clique com o botão direito > Salvar** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** ou **clique com o botão direito > Sair** para fechar o editor de código mantendo a linha de comando do Cloud Shell aberta.

## Enviar prompts para os modelos implantados

Agora você irá executar vários scripts que enviam prompts diferentes para os modelos implantados. Essas interações geram dados que você pode observar posteriormente no Azure Monitor.

1. Execute o seguinte comando para **visualizar o primeiro script** fornecido:

    ```powershell
   code model1.py
    ```

O script irá codificar a imagem usada neste exercício em uma URL de dados. Essa URL será usada para incorporar a imagem diretamente na solicitação de conclusão de chat, juntamente com o primeiro prompt de texto. Em seguida, o script irá produzir a resposta do modelo e adicioná-la ao histórico de chat. Depois, enviará um segundo prompt. O segundo prompt é enviado e armazenado com o objetivo de tornar as métricas observadas posteriormente mais significativas, mas você pode remover marca de comentário da seção opcional do código para que a segunda resposta também seja exibida como saída.

1. No painel de linha de comando do Cloud Shell, abaixo do editor de código, insira o seguinte comando para executar o **primeiro** script:

    ```powershell
   python model1.py
    ```

    O modelo gerará uma resposta, que será capturada com o Application Insights para análise posterior. Vamos usar o segundo modelo para explorar suas diferenças.

1. No painel de linha de comando do Cloud Shell, abaixo do editor de código, insira o seguinte comando para executar o **segundo** script:

    ```powershell
   python model2.py
    ```

    Agora que você tem as saídas de ambos os modelos, elas são diferentes de alguma forma?

    > **Observação**: Opcionalmente, você pode testar os scripts fornecidos como respostas copiando os blocos de código, executando o comando `code your_filename.py`, colando o código no editor, salvando o arquivo e, em seguida, executando o comando `python your_filename.py`. Se o script for executado com êxito, você deverá ter uma imagem salva que possa ser baixada com `download imgs/gpt-4o.jpg` ou `download imgs/gpt-4o-mini.jpg`.

## Comparar o uso de token dos modelos

Por fim, você irá executar um terceiro script que vai plotar o número de tokens processados ao longo do tempo para cada modelo. Esses dados são obtidos do Azure Monitor.

1. Antes de executar o último script, você precisa copiar a ID do recurso para seus Serviços de IA do Azure no portal do Azure. Vá para a página de visão geral do recurso Serviços de IA do Azure e selecione **Exibição JSON**. Copie a ID do recurso e substitua o espaço reservado `your_resource_id` no arquivo de código:

    ```powershell
   code plot.py
    ```

1. Salve suas alterações.

1. No painel de linha de comando do Cloud Shell, abaixo do editor de código, insira o seguinte comando para executar o **terceiro** script:

    ```powershell
   python plot.py
    ```

1. Depois que o script for concluído, insira o seguinte comando para baixar o gráfico de métricas:

    ```powershell
   download imgs/plot.png
    ```

## Conclusão

Após revisar o gráfico e relembrar os valores de referência no gráfico de Precisão versus Custo observado anteriormente, você consegue concluir qual modelo é o mais adequado para o seu caso de uso? A diferença na precisão das respostas compensa a diferença na quantidade de tokens gerados e, consequentemente, no custo?

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou abra novamente a [portal do Azure](https://portal.azure.com?azure-portal=true) em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
