---
lab:
  title: Monitorar seu aplicativo de IA generativa
  description: Saiba como monitorar interações com seu modelo implantado e obter insights sobre como otimizar seu uso com seu aplicativo de IA generativa.
---

# Monitorar seu aplicativo de IA generativa

Este exercício levará aproximadamente **30** minutos.

> **Observação**: este exercício pressupõe alguma familiaridade com a Fábrica de IA do Azure, e é por isso que algumas instruções são intencionalmente menos detalhadas para incentivar uma exploração mais ativa e o aprendizado prático.

## Introdução

Neste exercício, você habilita o monitoramento de um aplicativo de conclusão de chat e exibe seu desempenho no Azure Monitor. Você interage com seu modelo implantado para gerar dados, visualizar os dados gerados por meio do painel de aplicativos do Insights para IA Generativa e configurar alertas para ajudar a otimizar a implantação do modelo.

## Configurar o ambiente

Para concluir as tarefas neste exercício, será necessário:

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

Conecte o Application Insights ao seu projeto na Fábrica de IA do Azure para começar a coletar dados para monitoramento.

1. Use o menu à esquerda e selecione a página **Rastreamento**.
1. **Crie um novo** recurso do Application Insights para se conectar ao seu aplicativo.
1. Insira um nome de recurso do Application Insights e selecione **Criar**.

O Application Insights agora está conectado ao seu projeto, e os dados começarão a ser coletados para análise.

## Interagir com um modelo implantado

Você interagirá com seu modelo implantado programaticamente configurando uma conexão com seu projeto da Fábrica de IA do Azure usando o Azure Cloud Shell. Isso permitirá que você envie um prompt para o modelo e gere dados de monitoramento.

### Conectar-se a um modelo por meio do Cloud Shell

Comece recuperando as informações necessárias a serem autenticadas para interagir com seu modelo. Em seguida, você acessará o Azure Cloud Shell e atualizará a configuração para enviar os prompts fornecidos para seu próprio modelo implantado.

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

    > **Dica**: ao colar comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-genaiops/Files/07
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

### Enviar prompts para o modelo implantado

Agora você irá executar vários scripts que enviam prompts diferentes para o modelo implantado. Essas interações geram dados que você pode observar posteriormente no Azure Monitor.

1. Execute o seguinte comando para **visualizar o primeiro script** fornecido:

    ```
   code start-prompt.py
    ```

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

    O modelo gerará uma resposta, que será capturada com o Application Insights para análise posterior. Vamos variar nossos prompts para explorar seus efeitos.

1. **Abra e revise o script**, onde o prompt instrui o modelo a **responder apenas com uma frase e uma lista**:

    ```
   code short-prompt.py
    ```

1. **Execute o script** digitando o seguinte comando na linha de comando:

    ```
   python short-prompt.py
    ```

1. O próximo script tem um objetivo semelhante, mas inclui as instruções para a saída na **mensagem do sistema** em vez da mensagem do usuário:

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

1. **Execute o script**digitando o seguinte comando na linha de comando. Observe que é muito **provável que você encontre um erro.**

    ```
   python error-prompt.py
    ```

Agora que você interagiu com o modelo, pode examinar os dados no Azure Monitor.

> **Observação**: pode levar alguns minutos para que os dados de monitoramento sejam exibidos no Azure Monitor.

## Exibir dados de monitoramento no Azure Monitor

Para exibir os dados coletados de suas interações de modelo, você acessará o painel vinculado a uma pasta de trabalho no Azure Monitor.

### Navegue até o Azure Monitoral a partir do Portal da Fábrica de IA do Azure

1. Navegue até a guia em seu navegador com o portal **Portal da Fábrica de IA do Azure** aberto.
1. Use o menu à esquerda, selecione **Rastreamento**.
1. Selecione o link na parte superior, que diz **Confira o painel de aplicativos do Insights for IA Generativa**. O link abrirá o Azure Monitor em uma nova guia.
1. Examine a **Visão geral** que fornece dados resumidos das interações com o modelo implantado.

## Interpretar métricas de monitoramento no Azure Monitor

Agora é hora de se aprofundar nos dados e começar a interpretar o que eles dizem.

### Revise o uso do token

Concentre-se primeiro na seção de **uso do token** e analise as seguintes métricas:

- **Tokens de prompt**: o número total de tokens usados na entrada (os prompts que você enviou) em todas as chamadas de modelo.

> Pense nisso como o *custo de fazer* uma pergunta ao modelo.

- **Tokens de conclusão**: o número de tokens que o modelo retornou como saída, essencialmente o comprimento das respostas.

> Os tokens de conclusão gerados geralmente representam a maior parte do uso e do custo do token, especialmente para respostas longas ou detalhadas.

- **Total de tokens**: o total combinado de tokens de prompt e tokens de conclusão.

> Métrica mais importante para faturamento e desempenho, pois impulsiona a latência e o custo.

- **Total de chamadas**: o número de solicitações de inferência separadas, que é quantas vezes o modelo foi chamado.

> Útil para analisar a taxa de transferência e entender o custo médio por chamada.

### Compare os prompts individuais

Role para baixo para encontrar o **Gen AI Spans**, que é visualizado como uma tabela em que cada prompt é representado como uma nova linha de dados. Revise e compare o conteúdo das seguintes colunas:

- **Status**: se uma chamada de modelo foi bem-sucedida ou falhou.

> Use isso para identificar prompts problemáticos ou erros de configuração. O último prompt provavelmente falhou porque o prompt era muito longo.

- **Duração**: mostra quanto tempo o modelo levou para responder, em milissegundos.

> Compare entre linhas para explorar quais padrões de prompt resultam em tempos de processamento mais longos.

- **Entrada**: exibe a mensagem do usuário que foi enviada ao modelo.

> Use esta coluna para avaliar quais formulações imediatas são eficientes ou problemáticas.

- **Sistema**: mostra a mensagem do sistema usada no prompt (se houver).

> Compare as entradas para avaliar o impacto do uso ou da alteração de mensagens do sistema.

- **Saída**: contém a resposta do modelo.

> Use-o para avaliar o detalhamento, a relevância e a consistência. Especialmente em relação à contagem e duração de tokens.

## (OPCIONAL) Criar um alerta

Se você tiver tempo extra, tente configurar um alerta para notificá-lo quando a latência do modelo exceder um determinado limite. Este é um exercício projetado para desafiá-lo, o que significa que as instruções são intencionalmente menos detalhadas.

- No Azure Monitor, crie uma **nova regra de alerta** para o projeto e o modelo da Fábrica de IA do Azure.
- Escolha uma métrica como **Duração da solicitação (ms)** e defina um limite (por exemplo, maior que 4000 ms).
- Crie um novo grupo**de ações** para definir como você será notificado.

Os alertas ajudam você a se preparar para a produção, estabelecendo um monitoramento proativo. Os alertas que você configurar dependerão das prioridades do seu projeto e de como sua equipe decidiu medir e atenuar os riscos.

## Onde encontrar outros laboratórios

Você pode explorar laboratórios e exercícios adicionais no [Portal de Aprendizagem da Fábrica de IA do Azure](https://ai.azure.com) ou consultara seção** de laboratório do **curso para outras atividades disponíveis.
