# DataOps - Laboratório 3

GitHub e CodePipeline

Controlar versões com GitHub e implantar utilizando CodePipeline e CloudFormation para criar uma instância EC2 com Jenkins.

As instruções do laboratório estão em português. Para alterar o idioma, procure a opção na barra inferior do console AWS.


## Objetivos

* Utilização básica dos comandos git para clonar o repositório, enviar e atualizar arquivos
* Utilizar Github Actions para provisionar EC2 utilizando entrega contínua (CD)
* Configurar uma instância EC2 com Jenkins para utilizar com CI/CD


## Arquitetura da solução

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem1.png" height='330'/>


## Template CloudFormation

1. Abra o Cloud9 no seu ambiente AWS.	

2. Crie uma pasta "lab3"

3.	Na nova pasta (lab3) crie um template do CloudFormation chamado "ec2-jenkins.yaml"

4.	No template, provisione um Security Group que possibilita a conexão por SSH na instância EC2 (porta 22) e uma instância EC2 com as seguintes características (código a seguir):

    a.	Nome: ec2-jenkins

    b.	Imagem: Amazon Linux 2 (ami-0dc2d3e4c0f9ebd18)

    c.	Tipo de instância: t2.micro

    d.	User data: bash script para instalar e iniciar o Jenkins


```yaml
${code/ec2-jenkins-p1.yaml}
```

# Configurar Github

Para o github fazer o deploy no CloudFormation é necessário cinfigurar as chaves de acesso a AWS no repositório

1.  Na tela de iniciar o laborátio no AWS Academy, clique no botão `AWS Details`

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem19.png" width='100%'/>

2. Clique no botão `Show` para ver as credenciais de acesso. Elas serão utilizadas no github

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem20.png" width='100%'/>

3. Acesse o Github, crie uma conta (se ainda não tiver) e crie um repositório chamadoo `dataops-lab3`

    * Veja o material de aula para revisar como criar uma conta no Github e como criar o repositório

4. Abra o repositório `dataops-lab3` no Github e acesse a aba `Settings`

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem21.png" width='100%'/>

5. Clique em `Secrets and variables` e depois em `Actions` no menu lateral esquerdo

6. Clique no botão `New repository secret`

    6.1. Em `Name`, coloque `AWS_ACCESS_KEY_ID`

    6.2. Em `Secret` coloque o valor do campo `aws_access_key_id` que está no AWS Academy (veja passos 1 e 2 dessas seção)

    6.3. Clique em `Add secret`

7. Faça o mesmo para os outros dois campos:

    7.1. Name: `AWS_SECRET_ACCESS_KEY`;  Secret: valor de `aws_secret_access_key` no AWS Academy

    7.2. Name: `AWS_SESSION_TOKEN`;  Secret: valor de `aws_session_token` no AWS Academy

Você deve ver o seguinte resultado no Github


<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem22.png" width='100%'/>


**IMPORTANTE**

Essas credenciais mudam a cada vez que inicia o ambiente da AWS. Portanto, voc~e deve atualizá-las na tela anterior sempre que iniciar ambiente novamente.


## Script Github Action

1. Volte ao Cloud9 na AWS

2. Na pasta `dataops-lab3` crie uma nova pasta chamada `.github` (preste atenção no ponto no início da pasta)

3. Na pasta `.github` crie uma pasta `workflows`

4. na pasta `workflows` clie um arquivo `actions.yml` e coloque o seguinte conteúdo:


```yaml
${code/actions.yml}
```

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem23.png" width='100%'/>

5. Identifique o terminal de linha de comando no CLou9

    a. O terminal de linha de comando fica na parte inferior da IDE.

    <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem25.png" width='100%'/>

    b. caso não esteja vendo o terminal, clique em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem26.png" height='27'/> na parte inferior da IDE e depois em `New Terminal`

    <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem27.png" height='350'/>


6. Pelo terminal do Cloud9, vincule a pasta `lab3` ao repositório `dataops-lab3`. Revise o material de aula para instruções de como fazer.


    a. Você deve estar na pasta `lab3` no terminal para fazer isso. Se você criou a pasta `lab3` conforme as instruções deste lab, basta digitar `cd lab3` no terminal para acessar a pasta. Vincule o git e envie os arquivos a partir daí


     <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem28.png" height='80'/>


7. Verifique se todos os arquivos estão salvos no Cloud9.

8. Envie o código para seu repositório do Github da forma como fizemos em aula

    a.	Execute os comandos add, commit e push na branch develop

    b.	No Github faça o pull request e junte (merge) na branch main

**OBS:** Será solicitado o usuário e senha do github. O usuário é o seu username definido quando criou a conta. A senha deve ser um personal access token. Veja como criar aqui: [https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic). Verifique se está criando o `Tokens (classic)`. Pode colocar todas as permissões. Guarde este token para utilizar posteriormente em outros laboratórios.

9. Ao fazer o pull request e juntar na branch `main`, a `action` configurada será executada. Você pode acompanhar a execução acessando a aba `Actions` no repositório do Github. Quano o pipeline terminar com sucesso, você verá a seguinte tela:

<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem24.png"width='100%'/>


10.	Acesse o Serviço do EC2 para ver a instância criada
 
<img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem17.png" height='120'/>

11.	Verifique se a serviço do Jenkins está funcionando

    a.	No painel do EC2, selecione a instância que foi criada clicando no checkbox

    b.	Abaixo, em "Detalhes", copie o IP da instância clicando em <img src="https://raw.github.com/fesousa/dataops-lab3/master/images/Imagem18.png" height='18'/> abaixo de "Endereço IPv4 público" 
  
    c.	Abra uma nova do navegador, cole o IP copiado e complete com a porta 8080. Pressione "Enter" para abrir o Jenkins
  
    Por exemplo: 12.345.67.890:8080

    d.	A instância não responde. É preciso adicionar a regra de entrada no Security Group para possibilitar o acesso a porta 8080

12.	Volte ao arquivo "ec2-jenkins.yaml" do seu projeto no Cloud9

13.	Coloque uma nova regra de entrada na propriedade "SecurityGroupIngress" do Securi-tyGroup para possibilitar a conexão via HTTP na porta 8080, conforme exemplo abaixo (2 trechos entre ##### NOVO - INÍCIO ##### e ##### NOVO - FIM #####). Vamos aproveitar e criar um IP Elástico para a instância, assim quando a ins-tância for reiniciada o IP não vai mudar

```yaml
${code/ec2-jenkins-p2.yaml}
```

14.	Faça o push da atualização para o repositório do GitHub como feito anteriormente para atualizar o arquivo no repositório

  a.	add, commit, push, pull request e merge

15.	A atualização do arquivo no GitHub vai disparar a execução do Github Actions para fazer o deploy no CloudFormation. Acesse o CloudFormation na AWS e verifique a atualização em andamento

16.	Espere a finalização e tente acessar o Jenkins novamente pelo navegador. O IP continua o mesmo. O serviço deve funcionar



<div class="footer">
    &copy; 2023 Fernando Sousa
    <br/>
    {{update}}
</div>