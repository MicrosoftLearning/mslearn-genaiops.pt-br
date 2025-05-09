---
lab:
  title: Monitorar seu aplicativo de IA generativa
---

# Monitorar seu aplicativo de IA generativa

Este exercício levará aproximadamente **30** minutos.

> **Observação**: este exercício pressupõe alguma familiaridade com a Fábrica de IA do Azure, e é por isso que algumas instruções são intencionalmente menos detalhadas para incentivar uma exploração mais ativa e aprendizado prático.

## Introdução

Neste exercício, você habilita o monitoramento de um aplicativo de conclusão de chat e exibe seu desempenho no Azure Monitor. Você interage com seu modelo implantado para gerar dados, visualizar os dados gerados por meio do painel de aplicativos de Insights para IA generativa e configurar alertas para ajudar a otimizar a implantação do modelo.

## 1. Configurar o ambiente

Para concluir as tarefas neste exercício, você precisa:

- Um hub da Fábrica de IA do Azure,
- Um projeto da Fábrica de IA do Azure,
- Um modelo implantado (como GPT-4o),
- Um recurso do Application Insights conectado.

### R. Criar um hub e projeto da Fábrica de IA

Para configurar rapidamente um hub e um projeto, instruções simples de uso da interface de usuário do portal da Fábrica de IA do Azure são fornecidas abaixo.

1. Navegue até o portal da Fábrica de IA do Azure: Abra [https://ai.azure.com](https://ai.azure.com).
1. Entre utilizando suas credenciais do Azure.
1. Criar um projeto:
    1. Navegue até **Todos os hubs + projetos**.
    1. Selecione **+ New project**.
    1. Digite o **nome do projeto**.
    1. Quando solicitado, **crie um novo hub**.
    1. Personalize o hub:
        1. Selecione **assinatura**, **grupo de recursos**, **local** etc.
        1. Conecte um **novo recurso dos Serviços de IA do Azure** (ignore a Pesquisa de IA).
    1. Confira os dados e selecione **Criar**.
1. **Aguarde alguns minutos para a conclusão da implantação** (1-2 minutos).

### B. Implantar um modelo

Para gerar dados que você possa monitorar, primeiro, precisa implantar um modelo e interagir com ele. Nas instruções, você é solicitado a implantar um modelo GPT-4o, mas **pode usar qualquer modelo** da coleção do Serviço OpenAI do Azure que estiver disponível para você.

1. Use o menu à esquerda, em **Meus ativos**, selecione a página **Modelos + pontos de extremidade**.
1. Implante um **modelo base** e escolha **gpt-4o**.
1. **Personalize os detalhes da implantação**.
1. Defina a **capacidade** como **5K tokens por minuto (TPM).**

O hub e o projeto estão prontos, com todos os recursos necessários do Azure provisionados automaticamente.

### C. Conectar ao Application Insights

Conecte o Application Insights ao seu projeto na Fábrica de IA do Azure para iniciar os dados coletados para monitoramento.

1. Abra seu projeto no portal da Fábrica de IA do Azure.
1. Use o menu à esquerda e selecione a página **Rastreamento**.
1. **Crie um novo** recurso do Application Insights para se conectar ao seu aplicativo.
1. Insira o **nome do recurso do Application Insights**.

O Application Insights agora está conectado ao seu projeto, e os dados começarão a ser coletados para análise.

## 2. Interaja com um modelo implantado

Você interagirá com seu modelo implantado programaticamente, configurando uma conexão com seu projeto da Fábrica de IA do Azure usando o Azure Cloud Shell. Isso permitirá que você envie um prompt para o modelo e gere dados de monitoramento.

### R. Conectar-se a um modelo por meio do Cloud Shell

Comece recuperando as informações necessárias a serem autenticadas para interagir com seu modelo. Em seguida, você acessará o Azure Cloud Shell e atualizará a configuração para enviar os prompts fornecidos para seu próprio modelo implantado.

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.
1. Na área **Detalhes do projeto**, observe a **Cadeia de conexão do projeto**.
1. **Salve** a cadeia de caracteres em um bloco de notas. Você usará essa cadeia de conexão para se conectar ao seu projeto em um aplicativo cliente.
1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente).
1. Na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure se for solicitado.
1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.
1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para Versão Clássica**.

    **<font color="red">Verifique se você mudou para a Versão Clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira e execute os seguintes comandos:

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Esse comando clona o repositório GitHub que contém os arquivos de código para este exercício.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-foundry/Files/07
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Digite o seguinte comando para abrir o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código:

    1. Substitua o espaço reservado **your_project_connection_string** pela cadeia de conexão do seu projeto (copiada da página **Visão geral** do projeto no Portal da Fábrica de IA do Azure).
    1. Substitua o espaço reservado **your_model_deployment** pelo nome que você atribuiu à sua implantação do modelo GPT-4o (por padrão`gpt-4o`).

1. *Depois* de substituir os espaços reservados, no editor de códigos, use o comando **CTRL+S** ou **clique com o botão direito do mouse > Salvar** para **salvar as alterações**.

### B. Envie prompts para o seu modelo implantado

Agora, você executa vários scripts que enviam prompts diferentes para o seu modelo implantado. Essas interações geram dados que você pode observar posteriormente no Azure Monitor.

1. Execute o seguinte comando para **visualizar o primeiro script** fornecido:

    ```
   code start-prompt.py
    ```

1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para **executar o aplicativo**:

    ```
   python start-prompt.py
    ```

    O modelo gerará uma resposta, que será capturada com o Application Insights para análise posterior. Vamos variar nossos prompts para explorar seus efeitos.

1. **Abra e revise o script**, onde o prompt instrui o modelo a **responder apenas com uma frase e uma lista**:

    ```
   code short-prompt.py
    ```

1. **Execute o script** digitando o seguinte comando na linha de comando:

    ```
   python short-prompt.py
    ```

1. O próximo script tem objetivo semelhante, mas inclui as instruções para saída na **mensagem do sistema** em vez da mensagem do usuário:

    ```
   code system-prompt.py
    ```

1. **Execute o script** digitando o seguinte comando na linha de comando:

    ```
   python system-prompt.py
    ```

1. Por fim, vamos tentar disparar um erro executando um prompt com **muitos tokens**:

    ```
   code error-prompt.py
    ```

1. **Execute o script** digitando o seguinte comando na linha de comando. Observe que é muito **provável que você encontre um erro!**

    ```
   python error-prompt.py
    ```

Agora que você interagiu com o modelo, pode examinar os dados no Azure Monitor.

> **Observação**: pode levar alguns minutos para que os dados de monitoramento sejam exibidos no Azure Monitor.

## 4. Exibir dados de monitoramento no Azure Monitor

Para exibir os dados coletados de suas interações de modelo, você acessará o painel vinculado a uma pasta de trabalho no Azure Monitor.

### R. Do portal da Fábrica de IA do Azure, navegue até o Azure Monitor

1. Navegue até a guia em seu navegador com o **portal da Fábrica de IA do Azure** aberto.
1. Use o menu à esquerda, selecione **Rastreamento**.
1. Selecione o link na parte superior, que diz **Confira o painel de aplicativos de Insights para IA generativa**. O link abrirá o Azure Monitor em uma nova guia.
1. Examine a **Visão geral** que fornece dados resumidos das interações com o modelo implantado.

## 5. Interpretar métricas de monitoramento no Azure Monitor

Agora, é hora de se aprofundar nos dados e começar a interpretar o que eles dizem.

### R. Revise o uso de tokens

Concentre-se primeiro na seção de **uso de tokens** e analise as seguintes métricas:

- **Tokens de prompt**: o número total de tokens usados na entrada (os prompts que você enviou) em todas as chamadas de modelo.

> Pense nisso como o *custo de se fazer* uma pergunta ao modelo.

- **Tokens de conclusão**: o número de tokens que o modelo retornou como saída, essencialmente o comprimento das respostas.

> Os tokens de conclusão gerados geralmente representam a maior parte do uso e do custo dos tokens, especialmente em respostas longas ou detalhadas.

- **Total de tokens**: o total somado de tokens de prompt e tokens de conclusão.

> É a métrica mais importante para faturamento e desempenho, pois impulsiona a latência e o custo.

- **Total de chamadas**: número de solicitações de inferência separadas, que é quantas vezes o modelo foi chamado.

> Útil para se analisar a taxa de transferência e entender o custo médio por chamada.

### B. Compare os prompts individuais

Role para baixo até encontrar o **Gen AI Spans**, que é visualizado como uma tabela em que cada prompt é representado como uma nova linha de dados. Revise e compare o conteúdo das seguintes colunas:

- **Status**: se uma chamada de modelo foi bem-sucedida ou falhou.

> Use isso para identificar prompts problemáticos ou erros de configuração. O último prompt provavelmente falhou porque o prompt era muito longo.

- **Duração**: mostra quanto tempo o modelo levou para responder, em milissegundos.

> Compare entre linhas para explorar quais padrões de prompt resultam em tempos de processamento mais longos.

- **Entrada**: exibe a mensagem do usuário que foi enviada ao modelo.

> Use esta coluna para avaliar quais formulações de prompt são eficientes ou problemáticas.

- **Sistema**: mostra a mensagem do sistema usada no prompt (se houver).

> Compare as entradas para avaliar o impacto do uso ou da alteração de mensagens do sistema.

- **Saída**: contém a resposta do modelo.

> Use-a para avaliar o detalhamento, a relevância e a consistência. Especialmente em relação a contagem e duração de tokens.

## 6. (OPCIONAL) Criar um alerta

Se você tiver tempo extra, tente configurar um alerta para notificá-lo quando a latência do modelo exceder um determinado limite. Este é um exercício projetado para desafiá-lo, o que significa que as instruções são intencionalmente menos detalhadas.

- No Azure Monitor, crie uma **nova regra de alerta** para o seu projeto e modelo da Fábrica de IA do Azure.
- Escolha uma métrica como **Duração da solicitação (ms)** e defina um limite (por exemplo, maior que 4000 ms).
- Crie um **novo grupo de ações** para definir como você será notificado.

Os alertas ajudam você a se preparar para a produção, estabelecendo um monitoramento proativo. Os alertas que você configura dependem das prioridades do seu projeto e de como a sua equipe decidiu medir e mitigar os riscos.

## Onde encontrar outros laboratórios

Você pode explorar laboratórios e exercícios adicionais no [Portal de Aprendizagem da Fábrica de IA do Azure](https://ai.azure.com) ou consultar a **seção de laboratório** do curso para verificar outras atividades disponíveis.
