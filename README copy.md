# DataOps - Laboratório 1

Laboratório para ambientação no console AWS.

Armazenamento com S3 e notificação com SNS.

As instruções do laboratório estão em português. Para alterar o idioma, procure a opção na barra inferior.


## Objetivos

* Ambientação no ambiente e console AWS
* Criar um bucket S3 para armazenar arquivos
* Criar um Tópico SNS para enviar mensagens
* Criar uma assinatura de e-mail para o tópico SNS
* Configurar S3 para enviar uma mensagem para SNS quando incluir ou excluir arquivo

## Arquitetura da solução

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/lab1.png" height='330'/>


## Bucket S3 e notificações SNS

1. Inicie seu ambiente da AWS

2. No console da AWS, procure pelo serviço S3 na barra pesquisa superior e clique para abrir o serviço

    a.	O Amazon S3 (Amazon Simple Storage Service) é um serviço de armazenamento de objetos com escalabilidade, disponibilidade, segurança e desempenho. A quantidade de dados que pode ser armazenada é virtualmente ilimitada

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img3.png" height='270'/> 

4.	O primeiro passo é criar um bucket para armazenar objetos. 

    a.	Um bucket é um contêiner de objetos. Um objeto é qualquer arquivo ou documento colocado no bucket. 
    b.	Clique no botão <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img4.png" height='22'/>  para criar um novo bucket

5.	Na tela de criação e configuração do novo bucket, preencha os seguintes campos:

&nbsp;&nbsp;&nbsp;&nbsp;    a.	"Nome do bucket": dataops-dados-***nomesobrenome***
        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; i. Troque ***nomesobrenome*** pelo seu nome e sobrenome. O nome do bucket deve ser único em toda a AWS, independente da conta e região
   
&nbsp;&nbsp;&nbsp;&nbsp;    b. "Região da AWS": Leste dos EUA (Norte da Virgínia) us-east-1

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; i. Preste atenção na região. Sempre vamos utilizar essas nos labs

&nbsp;&nbsp;&nbsp;&nbsp;    c. Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img4.png" height='22'/>


6.	Você será redirecionado para a página inicial com a lista dos buckets, na qual verá o bucket criado 

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img5.png" height='70'/>

7.	O próximo passo é criar um tópico SNS

    a.	O Amazon SNS (Amazon Simple Notification Service) é um serviço de mensagens totalmente gerenciado para comunicação entre aplicações e comunicação entre aplicações e pessoas

    b.	As mensagens são enviadas e armazenadas em um tópico, num sistema de muitos para muitos. Ou seja, uma mensagem pode ser enviada para diversos destinos

    c.	Mensagem são enviadas por um sistema de *push* para destinos que fazem a assinatura de um tópico. Um tópico pode ter vários assinantes
    
    d.	Um assinante pode ser um e-mail, SMS, endpoint HTTP/S, função lambda, fila SQS, entre outros.

8.	No console da AWS, procure pelo serviço SNS na barra pesquisa superior e clique para abrir o serviço


<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img6.png" height='270'/>


9.	Abra o menu lateral clicando em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img7.png" height='22'/>  e escolha a opção <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img8.png" height='22'/>

10.	Clique no botão <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img9.png" height='22'/>

11.	Na tela de criar e configurar o tópico, configure os seguintes campos:

    a.	"Tipo": Padrão 

    b.	"Nome": Topico-Evento-Dados-S3

    c.	Abra a seção "Política de acesso"

    &nbsp;&nbsp;&nbsp;&nbsp; i.	Uma política de acesso define quais as permissões do tópico criado. A permissão padrão define que apenas o proprietário do tópico (conta que criou) pode publicar mensagens. É preciso dar a permissão para o S3 também publicar uma mensagem

    d.	Selecione a opção "Avançado"

    e.	Adicione o seguinte "Statement" antes do fechamento do último colchete (]), separando por uma vírgula (,). Os campos em negrito devem ser substituídos pelos valores da sua conta. Sempre que encontrar um campo em negrito nas instruções, lembre-se de trocar por uma configuração do seu ambiente


```json
{
  "Sid": "Statement-id",
  "Effect": "Allow",
  "Principal": { "Service": "s3.amazonaws.com" },
  "Action": "sns:Publish",
  "Resource": "arn:aws:sns:us-east-1:id-conta:Topico-Evento-Dados-S3",
  "Condition": {
    "ArnLike": {
      "aws:SourceArn": "arn:aws:s3:::dataops-dados-nomesobrenome"
    }
  }
}
```


&nbsp;&nbsp;&nbsp;&nbsp;Troque o nome do bucket (dataops-dados-***nomesobrenome***) pelo nome do bucket que criou nos passos anteriores e idconta pelo id da sua conta (disponível na barra superior, ao clicar no nome do usuário – voclabs/user... e clicar no ícone de copiar)


&nbsp;&nbsp;&nbsp;&nbsp;O documento completo da política deve ficar assim:


```json
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:Publish",
        "SNS:RemovePermission",
        "SNS:SetTopicAttributes",
        "SNS:DeleteTopic",
        "SNS:ListSubscriptionsByTopic",
        "SNS:GetTopicAttributes",
        "SNS:Receive",
        "SNS:AddPermission",
        "SNS:Subscribe"
      ],
      "Resource": "",
      "Condition": {
        "StringEquals": {
          "AWS:SourceOwner": " id-conta"
        }
      }
    },
    {
      "Sid": "Statement-id",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:id-conta:Topico-Evento-Dados-S3",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::dataops-dados-nomesobrenome"
        }
      }
    }
  ]
}

```



&nbsp;&nbsp;&nbsp;&nbsp;f.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img10.png" height='22'/> 

12.	Para que a mensagem seja enviada é preciso criar assinaturas (consumidor) e definir quem vai receber a mensagem enviada para o tópico (produtor)

13.	Na lista dos tópicos (se não estiver vendo, clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img11.png" height='22'/> no menu lateral), clique no nome do tópico criado anteriormente.

14.	Na seção "Assinaturas", clique em  <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img12.png" height='22'/>

15.	Na tela de configuração da nova assinatura preencha os seguintes campos:

    a.	"Protocolo": E-mail

    b.	"Endpoint": coloque o e-mail para receber a notificação

    c.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img12.png" height='22'/>

16.	A assinatura para um e-mail depende da autorização do dono do e-mail para completar a configuração. 

    a.	Acesse a caixa de e-mail que configurou na assinatura

    b.	Procure pelo e-mail com o título "AWS Notification - Subscription Confirmation"

    c.	Clique no link "Confirm subscription"

    d.	Você verá a tela de assinatura confirmada

    e.	Volte ao console do SNS da AWS e clique novamente em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img11.png" height='22'/>

    f.	Selecione o tópico criado anteriormente (Topico-Evento-Dados-S3)

    g.	Na seção assinaturas, veja que o status da assinatura está confirmado

17.	Por fim, precisamos colocar um evento no S3 para enviar uma mensagem para o tópico SNS Topico-Evento-Dados-S3

18.	No console da AWS, procure pelo serviço S3 na barra pesquisa superior e clique para abrir o serviço


<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img13.png" height='270'/>


19.	Selecione o bucket criado anteriormente clicando no nome (dataops-dados-***nomesobrenome***)

20.	Clique na aba <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img14.png" height='22'/>

21.	Procure a seção “Notificações de eventos” e clique em  <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img15.png" height='22'/>


22.	Na tela de criação e configuração do evento, preencha:

    a.	"Nome do evento": Alteração de objeto
    
    b.	"Tipos de evento": Selecione os checkbox de "Todos os eventos de criação de objeto" e “Todos os eventos de exclusão de objeto”
    
    c.	"Destino": Tópico SNS
    
    d.	"Tópico SNS": Selecione Topico-Evento-Dados-S3
    
    e.	Clique em  <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img16.png" height='22'/>
    
    f.	Se receber o alerta abaixo, verifique a política de acesso do SNS


 <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img17.png" height='230'/>
 

23.	Faça um teste de incluir um arquivo no bucket

    a.	No console do S3, selecione o bucket criado neste lab

    b.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img18.png" height='22'/>

    c.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img19.png" height='22'/>

    d.	Procure um arquivo no seu computador 

    e.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img20.png" height='21'/>
    
    f.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img21.png" height='22'/>
    
    g.	Veja o objeto carregado no bucket
    
    h.	Veja no seu e-mail a notificação

24.	Outra forma de carregar um arquivo para o bucket S3 é arrastar e soltar o arquivo para o bucket enquanto estiver na aba <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img22.png" height='22'/> . Faça o teste seguindo os passos do console. Você também deve receber a notificação

25.	A remoção também gera uma notificação. Faça o teste:

    a.	Na aba <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img22.png" height='22'/> do seu bucket, selecione o arquivo carregado clicando no checkbox

    b.	Clique em  <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img23.png" height='23'/> nas opções que estão na parte superior

    c.	Digite "excluir permanentemente" no campo de texto em "Excluir objetos permanentemente?"

    d.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img24.png" height='22'/> e espere a barra superior indicar que finalizou (cor verde)

    e.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img25.png" height='22'/>

    f.	Veja que o arquivo foi excluído

    g.	Veja no seu e-mail a notificação


## Consultar Dados utilizando S3 Select

S3 Select é uma funcionalidade do S3 que possibilita selecionar e retornar somente parte dos dados de um objeto em um bucket S3, utilizando SQL. Ele é utilizado para reduzir o volume de dados transferidos para a aplicação e deixar a consulta de objetos mais rápidas e mais baratas

1.	Acesse o bucket dataops-dados-***nomesobrenome*** criado anteriormente

2.	Faça o upload do arquivo [vacinas_ac_agosto.csv](https://raw.github.com/fesousa/dataops-lab1/master/data/vacinas_ac_agosto.csv) disponibilizado

3.	Quando o upload terminar, acesse novamente o bucket e selecione o objeto que acabou de carregar

<img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img26.png" height='300'/>
 
4.	Com o objeto selecionado, clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img27.png" height='22'/> e depois em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img28.png" height='22'/>

5.	Na tela <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img29.png" height='22'/> configure as opções de seleção:

    a.	Configurações de entrada:

    &nbsp;&nbsp;&nbsp;&nbsp; i.	Formato: <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img30.png" height='22'/>

    &nbsp;&nbsp;&nbsp;&nbsp;ii.	CSV delimitador: <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img31.png" height='22'/>

    &nbsp;&nbsp;&nbsp;&nbsp;ii.	CSV delimitador personalizado: ponto-e-vírgula <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img32.png" height='22'/>

    &nbsp;&nbsp;&nbsp;&nbsp;iv.	Marque <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img33.png" height='35'/>

    b.	Configurações de saída

    &nbsp;&nbsp;&nbsp;&nbsp; i.	Formato: <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img34.png" height='22'/>


    c.	Consulta SQL

    &nbsp;&nbsp;&nbsp;&nbsp;i.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab1/master/images/img35.png" height='22'/> para executar a consulta padrão e retornar os primeiros 4 registros do CSV


```sql

SELECT * FROM s3object s LIMIT 5

```


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ii.	Veja o resultado em Resultados da consulta

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; iii.	Altere a consulta para retornar a quantidade de registros do CSV e execute novamente


```sql

SELECT count(1) FROM s3object s

```

&nbsp;&nbsp;&nbsp;&nbsp; d.	Você pode utilizar a maioria das expressões SQL para consultar os dados





<div class="footer">
    &copy; 2022 Fernando Sousa
    <br/>
    
Last update: 2022-02-06 00:03:16
</div>