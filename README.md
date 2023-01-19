# configs-localstack-amazon-sqs

<h1>CONFIGURAÇÕES E COMANDOS PARA CRIAR SERVIÇO SQS AMAZON LOCALMENTE:</h1>

ARQUIVO YML DISPONIVEL NO REPOSITORIO PARA CRIAÇÃO DO CONTAINER AMAZON SQS SERVICE:

BASTA BAIXAR COLOCAR EM UMA PASTA, ABRIR CMD DENTRO DA PASTA E DIGITAR DOCKER COMPOSE UP.

CRIAR FILA PELO CMD:

aws --endpoint-url http://localhost:9324/ sqs create-queue --queue-name nomefila

MOSTRAR FILAS:

aws --endpoint=http://localhost:9324/ sqs list-queues

ENVIANDO OBJETOS PARA FILA SQS - METODO JAVA


public void enviarClienteComClassificacaoFaixaRentabilidade(List<ClientDTO> list) {

        String listToJson = gson.toJson(list);

        SqsClient client = SqsClient.builder()
                .region(Region.US_EAST_1)
                .endpointOverride(URI.create("http://localhost:9324"))
                .build();

        SendMessageRequest sendClientetoSQS = SendMessageRequest.builder()
                .queueUrl("http://localhost:9324/queue/account_rentabilizador")
                .messageBody(listToJson)
                .delaySeconds(5).build();

        client.sendMessage(sendClientetoSQS);
    }

RECEBER MESSAGE DA FILA - METODO JAVA

public ReceiveMessageResponse receberFiladeClientesClassificados() {

        SqsClient client = SqsClient.builder()
                .region(Region.US_EAST_1)
                .endpointOverride(URI.create("http://localhost:9324"))
                .build();

        var queueUrl = "http://localhost:9324/queue/account_rentabilizador";

        ReceiveMessageRequest receiveMessageRequest = ReceiveMessageRequest.builder()
                .queueUrl(queueUrl)
                .waitTimeSeconds(5)
                .maxNumberOfMessages(5)
                .build();

        return client.receiveMessage(receiveMessageRequest);
    }

PEGANDO A MESSAGE DA FILA E CONVERTER NO OBJETO (CONVERTENDO JSON PARA OBJETO USANDO gson.fromJson) - METODO JAVA

public List<ClientDTO> receberListaDeClientesClassificadosDaFila() {

        List<ClientDTO> listClientesClassificadosNaFila = new ArrayList<>();
        Type founderListType = new TypeToken<ArrayList<ClientDTO>>() {
        }.getType();

        ReceiveMessageResponse receiveMessageResponse = receberFiladeClientesClassificados();

        receiveMessageResponse
                .messages()
                .forEach(message -> {
                    listClientesClassificadosNaFila.addAll(gson.fromJson(message.body(), founderListType));
                });

        return listClientesClassificadosNaFila;
    }

COMO A FILA SQS NÃO É APAGADA AO CONSUMIR UTILIZAR METODO PARA APAGAR - JAVA SQS

public void apagarFilaClientesClassificados() {

        SqsClient client = SqsClient.builder()
                .region(Region.US_EAST_1)
                .endpointOverride(URI.create("http://localhost:9324"))
                .build();

        var queueUrl = "http://localhost:9324/queue/account_rentabilizador";

        for(Message m : receberFiladeClientesClassificados().messages()){
            DeleteMessageRequest deleteMessageRequest = DeleteMessageRequest.builder()
                    .queueUrl(queueUrl)
                    .receiptHandle(m.receiptHandle())
                    .build();
            client.deleteMessage(deleteMessageRequest);
        }
    }



