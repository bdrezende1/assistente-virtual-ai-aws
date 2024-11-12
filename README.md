# assistente-virtual-ai-aws
- Criando um Assistente Virtual Usando Serverless e Amazon Bedrock

# Assistente Virtual de Gestão de Pedidos

Olá! Neste texto, vou explicar como funciona o desenvolvimento de um assistente virtual para a gestão de pedidos, utilizando as tecnologias da AWS. Essa solução foi pensada para ser eficiente e escalável, com o objetivo de melhorar a experiência do cliente e otimizar o processo de pedidos. Vou detalhar o papel de cada tecnologia envolvida, mas de forma simples, para que todos possam entender.

## AWS Step Functions

Para começar, usamos o **AWS Step Functions** como o "orquestrador" das operações. Pense no Step Functions como um maestro que coordena cada etapa do processo de pedidos. Em um fluxo de pedidos, várias tarefas precisam ser realizadas em uma sequência específica: verificar estoque, processar pagamento, preparar o pedido para envio, entre outras. O Step Functions garante que todas essas etapas ocorram de forma organizada e sem erros, além de fornecer controle e monitoramento em tempo real. Com ele, conseguimos visualizar e acompanhar cada fase do processo e rapidamente identificar e corrigir problemas, se houver algum.

## Amazon Lambda

Outro elemento essencial do nosso assistente virtual é o **Amazon Lambda**, que funciona como o "executor" de tarefas específicas dentro do fluxo do Step Functions. O Lambda é uma solução de computação sem servidor (serverless), ou seja, não precisamos gerenciar servidores para que ele funcione. No nosso assistente, o Lambda é acionado para tarefas específicas, como consultar o banco de dados para verificar a disponibilidade de um produto ou enviar notificações automáticas para o cliente. Cada vez que uma dessas tarefas é necessária, o Lambda é ativado automaticamente, executa o que precisa ser feito e depois desliga, economizando recursos e tempo.

## DynamoDB

Para armazenar e gerenciar os dados do assistente, utilizamos o **DynamoDB**, um banco de dados rápido e altamente escalável da AWS. Todos os dados sobre os pedidos são armazenados no DynamoDB: informações dos clientes, produtos, status de cada pedido e muito mais. O DynamoDB permite que o assistente acesse rapidamente essas informações, o que é essencial para que o sistema funcione em tempo real. Além disso, o DynamoDB é extremamente confiável e pode lidar com um grande volume de dados sem perder desempenho, o que é ideal para uma aplicação que precisa lidar com muitos pedidos ao mesmo tempo.

## Amazon Bedrock

Para tornar o assistente ainda mais inteligente, integramos o **Amazon Bedrock**, uma plataforma de IA da AWS que nos dá acesso a modelos de inteligência artificial de última geração. O Bedrock possibilita que nosso assistente entenda melhor as intenções dos clientes e adapte as respostas de acordo com as necessidades. Por exemplo, ele permite que o sistema interprete e organize pedidos recebidos por texto, seja por mensagem ou chat, identificando exatamente o que o cliente quer. Isso traz uma camada de inteligência que torna o processo mais rápido e o atendimento mais personalizado.

## Modelo de IA: Anthropic Claude 3 Haiku

Dentro da plataforma Amazon Bedrock, usamos o modelo **Anthropic Claude 3 Haiku** para processar e interpretar a linguagem natural dos clientes. Esse modelo é especializado em compreender e responder a perguntas em linguagem natural, ou seja, ele "conversa" com o cliente de maneira natural e amigável. Se o cliente faz uma pergunta ou consulta sobre seu pedido, o modelo interpreta a mensagem e fornece uma resposta precisa, como se fosse uma pessoa real. Isso melhora a experiência do cliente, que sente que está falando com alguém que entende suas dúvidas.

## Como Funciona na Prática?

Vamos ver como tudo isso funciona junto em um exemplo prático. Imagine que você faz um pedido em uma loja online que utiliza esse assistente. Assim que o pedido é feito, o Step Functions coordena todas as etapas, desde verificar o estoque (com o Lambda), registrar o pedido no banco de dados (DynamoDB) até enviar uma confirmação para o cliente. Se o cliente tiver dúvidas, o assistente virtual, usando o modelo Anthropic Claude 3 Haiku, responde suas perguntas. Todo esse processo ocorre de forma automatizada e sincronizada, permitindo uma experiência rápida e confiável para o cliente.

## Conclusão

Essa solução completa, com **AWS Step Functions**, **Amazon Lambda**, **DynamoDB**, **Amazon Bedrock** e o modelo **Anthropic Claude 3 Haiku**, foi projetada para facilitar a gestão de pedidos de forma ágil e automatizada. Com ela, as empresas podem oferecer um atendimento eficiente e rápido, ao mesmo tempo que os clientes acompanham o status de seus pedidos em tempo real. Espero que essa explicação tenha ajudado a entender como cada parte desse sistema contribui para criar uma experiência completa e satisfatória.

-----------------------------------------------------------------------------------------------------

# Explicação do Arquivo JSON de Máquina de Estados para Assistente Virtual

O arquivo JSON define uma **máquina de estados** utilizando o AWS Step Functions, que organiza e executa uma sequência de tarefas para processar uma conversa. Ele usa o DynamoDB para armazenamento de dados e o Amazon Bedrock para gerar respostas com IA. Abaixo está uma explicação detalhada de cada etapa do fluxo de trabalho.

## Estrutura do Código

1. **StartAt - Seed the DynamoDB Table**:
   - Este é o primeiro estado, uma tarefa de inicialização que executa a função Lambda chamada `MyLambdaFunction`. A função tem o objetivo de "semear" a tabela do DynamoDB com dados iniciais para a conversa. O resultado dessa execução é armazenado em `$.List`, uma lista de mensagens.

2. **Initialize Conversation History**:
   - Este estado inicializa o histórico de conversação com um valor vazio (`""`). Isso garante que o histórico de mensagens comece limpo e preparado para receber o conteúdo das mensagens.

3. **For Loop Condition**:
   - Este estado é uma **escolha condicional** (Choice) que verifica a condição de continuidade do loop. Se o primeiro elemento de `$.List` não for "DONE", ele avança para o próximo estado (`Read Next Message from DynamoDB`). Caso contrário, a execução finaliza com sucesso (`Succeed`).

4. **Read Next Message from DynamoDB**:
   - Aqui, o sistema lê a próxima mensagem da tabela DynamoDB utilizando o `MessageId` especificado em `$.List[0]`. O item recuperado é armazenado em `$.DynamoDB`, contendo a mensagem atual a ser processada.

5. **Update Conversation History with Message**:
   - Este estado "Pass" adiciona a mensagem recuperada ao histórico de conversa, formatando-a e concatenando com o conteúdo anterior de `$.ConversationHistory`. Assim, o histórico é atualizado com cada nova mensagem lida.

6. **Truncate Conversation History**:
   - A função Lambda `TruncateHistoryFunction` é chamada para truncar o histórico de conversação, garantindo que o tamanho máximo seja de 2000 caracteres. Isso evita que o histórico cresça demais e afete a performance.

7. **Invoke Model with Message**:
   - Nesta etapa, o modelo de IA `cohere.command-text-v14` do Amazon Bedrock é acionado para gerar uma resposta baseada no histórico da conversa (`$.ConversationHistory`). A resposta gerada é armazenada em `$.ModelResult`, permitindo que o assistente virtual responda de acordo com o contexto.

8. **Update Conversation History with Response**:
   - O histórico de conversação é atualizado com a resposta gerada pela IA, permitindo que o sistema mantenha o contexto e registre o andamento da interação com o usuário.

9. **Truncate Conversation History After Response**:
   - Após a resposta ser adicionada ao histórico, uma nova tarefa de truncamento é executada para manter o histórico dentro do limite de 2000 caracteres, garantindo consistência e performance.

10. **Pop Element from List**:
    - Esse estado remove o primeiro elemento de `$.List`, avançando para a próxima mensagem. Isso permite que o loop continue processando o restante das mensagens.

11. **For Loop Condition** (loop):
    - Após processar uma mensagem e truncar o histórico, o fluxo retorna ao estado de condição de loop para processar o próximo item da lista, até que não haja mais elementos ou o valor "DONE" seja alcançado.

12. **Succeed**:
    - Este estado finaliza o fluxo com sucesso quando todas as mensagens são processadas ou quando o estado "DONE" é encontrado.

## Resumo

Essa máquina de estados automatiza a leitura de mensagens, a atualização do histórico e a geração de respostas, criando um **assistente virtual** que consegue manter e processar uma conversa de forma eficiente e contextualizada. Com a combinação do AWS Step Functions, DynamoDB, Amazon Lambda e Amazon Bedrock, o sistema processa mensagens e gera respostas inteligentes para interações mais eficazes com o usuário.


Se tiver dúvidas ou quiser saber mais sobre o funcionamento desse assistente, estou à disposição!
