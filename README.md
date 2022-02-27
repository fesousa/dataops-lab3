# DataOps - Laboratório 3

GitHub e CodePipeline

Controlar versões com GitHub e implantar utilizando CodePipeline e CloudFormation para criar uma instância EC2 com Jenkins.

As instruções do laboratório estão em português. Para alterar o idioma, procure a opção na barra inferior do console AWS.


## Objetivos

* Utilização básica dos comandos git para clonar o repositório, enviar e atualizar arquivos
* Utilizar CodePipeline para provisionar EC2 utilizando entrega contínua (CD)
* Configurar uma instância EC2 com Jenkins para utilizar com CI/CD


## Arquitetura da solução

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem1.png" height='330'/>


## Provisionar Jenkins no EC2 com CodePipeline e CloudFormation

1.	Crie uma pasta no seu projeto do VSCode chamada "lab3"

2.	Na nova pasta (lab3) crie um template do CloudFormation chamado "ec2-jenkins.yaml"

3.	No template, provisione um Security Group que possibilita a conexão por SSH na instância EC2 (porta 22) e uma instância EC2 com as seguintes características (código a seguir):

  a.	Nome: ec2-jenkins

  b.	Imagem: Amazon Linux 2 (ami-0dc2d3e4c0f9ebd18)

  c.	Tipo de instância: t2.micro

  d.	User data: bash script para instalar e iniciar o Jenkins


```yaml
# Versão do template - não alterar
AWSTemplateFormatVersion: "2010-09-09"

# Descrição que será utilizada na stack
Description:
  Provisiona maquina EC2 com jenkins

# parâmetro para chave de acesso ao EC2
Parameters:
  key:
    Type: String
  iamprofile:
    Type: String

# Recursos que serão provisionados
Resources:  
  # Security Group para provisionar EC2
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties: 
      GroupDescription: Porta 22
      GroupName: ec2-jenkins-sg
      SecurityGroupIngress: 
        - CidrIp: 0.0.0.0/0
          FromPort: 22
          ToPort: 22
          IpProtocol: tcp  
  # Provisionar EC2
  EC2:
    Type: AWS::EC2::Instance
    Properties:        
      ImageId: ami-0dc2d3e4c0f9ebd18
      InstanceType: t2.micro
      KeyName: !Sub ${key}
      SecurityGroupIds:
        - !Ref SecurityGroup
      IamInstanceProfile: !Sub ${iamprofile}
      Tags:
        - Key: Name
          Value: ec2-jenkins
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash -xe
          wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
          rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
          yum upgrade -y
          amazon-linux-extras install epel -y
          yum install -y jenkins java-1.8.0-openjdk
          sudo systemctl daemon-reload
          sudo systemctl start jenkins

```

4.	Envie o código para seu repositório do Github da forma como fizemos em aula

  a.	Execute os comandos add, commit e push na branch develop

  b.	No Github faça o pull request e junte (merge) na branch master

5.	No console AWS, procure e abra o serviço CodePipeline

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem2.png" height='200'/>
 
6.	Clique em  <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem3.png" height='20'/>             e depois faça a seguinte configuração:

  a.	"Nome do Pipeline": dataops-ec2-pipeline

  b.	"Função de serviço": <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem4.png" height='20'/>

  c.	"ARN da função": clique no campo de texto e escolha <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem5.png" height='20'/>. O número é o ID da sua conta e será diferente 

  d.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem6.png" height='20'/>

  e.	"Provedor de origem": GitHub (versão 1)	

  f.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem7.png" height='20'/>

  g.	Se já estiver autenticado no GitHub basca clicar em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem8.png" height='20'/> na janela aberta

  h.	Se não estiver autenticado, faça o login no GitHub na janela aberta e depois clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem9.png" height='20'/>

  i.	"Nome do repositório": dataops-lab3 (ou o repositório criado em aula)
  
  j.	"Nome da ramificação": master

  k.	"Opções de detecção de alterações": Webhooks do GitHub (recomendado)

  l.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem10.png" height='20'/>

  m.	Em "Adicionar etapa de compilação" clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem11.png" height='20'/>
  
  n.	No popup, clique em  <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem12.png" height='20'/>

  o.	"Provedor da implantação": AWS CloudFormation

  p.	"Modo de ação": "Criar ou atualizar uma pilha"

  q.	"Nome da pilha": dataops-lab3

  r.	"Nome do artefato": SourceArtifact

  s.	"Nome do arquivo": ec2-jenkins.yaml

  t.	"Nome da função": selecione LabRole

  u.	Abra a opção Avançado e na caixa de texto coloque { "key": "vockey", "iamprofile": "LabInstanceProfile" }

  <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem13.png" height='150'/>
 
  v.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem14.png" height='20'/>

  w.	Clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem15.png" height='20'/>
  
  x.	Aguarde o pipeline terminar a execução

  <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem16.png" height='330'/>


7.	O pipeline criado utiliza o arquivo ec2-jenkins.yaml que criamos no VSCode e fizemos o push o GitHub para provisionar os recursos do template

8.	Acesse o Serviço do EC2 para ver a instância criada
 
<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem17.png" height='120'/>

9.	Verifique se a serviço do Jenkins está funcionando

  a.	No painel do EC2, selecione a instância que foi criada clicando no checkbox

  b.	Abaixo, em "Detalhes", copie o IP da instância clicando em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem18.png" height='18'/> abaixo de "Endereço IPv4 público" 
  
  c.	Abra uma nova do navegador, cole o IP copiado e complete com a porta 8080. Pressione "Enter" para abrir o Jenkins
  
  Por exemplo: 12.345.67.890:8080

  d.	A instância não responde. É preciso adicionar a regra de entrada no Security Group para possibilitar o acesso a porta 8080

10.	Volte ao arquivo "ec2-jenkins.yaml" do seu projeto no VSCode

11.	Coloque uma nova regra de entrada na propriedade "SecurityGroupIngress" do Securi-tyGroup para possibilitar a conexão via HTTP na porta 8080, conforme exemplo abaixo (2 trechos entre ##### NOVO - INÍCIO ##### e ##### NOVO - FIM #####). Vamos aproveitar e criar um IP Elástico para a instância, assim quando a ins-tância for reiniciada o IP não vai mudar

```yaml
# Versão do template - não alterar
AWSTemplateFormatVersion: "2010-09-09"

# Descrição que será utilizada na stack
Description:
  Provisiona maquina EC2 com jenkins

# parâmetro para chave de acesso ao EC2
Parameters:
  key:
    Type: String
  iamprofile:
    Type: String  

# Recursos que serão provisionados
Resources:  
  # Security Group para provisionar EC2
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties: 
      GroupDescription: Porta 22
      GroupName: ec2-jenkins-sg
      SecurityGroupIngress: 
        - CidrIp: 0.0.0.0/0
          FromPort: 22
          ToPort: 22
          IpProtocol: tcp
        ##### NOVO - INÍCIO #####
        - CidrIp: 0.0.0.0/0
          FromPort: 8080
          ToPort: 8080
          IpProtocol: tcp
        ##### NOVO - FIM #####
  ##### NOVO - INÍCIO #####
  #provisionar IP Elástico
  IPAddress:
    Type: AWS::EC2::EIP
  IPAssoc:
    Type: AWS::EC2::EIPAssociation
    Properties:
      InstanceId: !Ref 'EC2'
      EIP: !Ref 'IPAddress'
  ##### NOVO - FIM #####
  # Provisionar EC2
  EC2:
    Type: AWS::EC2::Instance
    Properties:        
      ImageId: ami-0dc2d3e4c0f9ebd18
      InstanceType: t2.micro
      KeyName: !Sub ${key}
      SecurityGroupIds:
        - !Ref SecurityGroup
      IamInstanceProfile: !Sub ${iamprofile}
      Tags:
        - Key: Name
          Value: ec2-jenkins
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash -xe
          wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
          rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
          yum upgrade -y
          amazon-linux-extras install epel -y
          yum install -y jenkins java-1.8.0-openjdk
          sudo systemctl daemon-reload
          sudo systemctl start jenkins

```

12.	Faça o push da atualização para o repositório do GitHub como feito anteriormente para atualizar o arquivo no repositório

  a.	add, commit, push, pull request e merge

13.	A atualização do arquivo no GitHub vai disparar a atualização do pipeline no CodePipeli-ne. Acesse o serviço do CodePipeline e verifique a atualização em andamento

14.	Espere a finalização e tente acessar o Jenkins novamente pelo navegador. O IP continua o mesmo. O serviço deve funcionar


<div class="footer">
    &copy; 2022 Fernando Sousa
    <br/>
    
Last update: 2022-02-27 20:15:53
</div>