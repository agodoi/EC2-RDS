# EC2-RDS
Criação de RDS MySQL


# O que vamos fazer nessa aula?

Iremos apresentar os métodos para modelagem de banco de dados relacional, e para isso, este laboratório foi criado para reforçar o conceito de utilização de uma instância de banco de dados gerenciado pela AWS para atender as necessidades de banco de dados relacional.

O Amazon Relational Database Service (Amazon RDS) facilita a configuração, a operação e o escalamento de um banco de dados relacional na nuvem. Ele oferece capacidade econômica e redimensionável enquanto gerencia [tarefas demoradas](https://github.com/agodoi/EC2-RDS/blob/main/vantagens) de administração de banco de dados , permitindo que você se concentre nas suas aplicações e nos seus negócios. O Amazon RDS fornece seis opções de mecanismos de banco de dados familiares: Amazon Aurora, Oracle, Microsoft SQL Server, PostgreSQL, MySQL e MariaDB. Mas hoje, vamos focar no **MySQL**

# Objetivos

Depois de concluir este laboratório, você será capaz de:

* Executar uma instância de banco de dados do Amazon RDS com alta disponibilidade.
* Configurar a instância de banco de dados para permitir conexões do seu servidor web.
* Abrir um aplicativo web e interagir com seu banco de dados.

# Pré-requisitos

* Você precisa entrar no curso AWS Academy Foundation, ir até o Módulo 8 e subir o **Laboratório 5 - Crie um servidor de banco de dados e interaja com o banco de dados usando um aplicativo**. Espera aparecer **Lab Status: ready** para abrir o console AWS. Você abre o console clicando no botão **AWS**.
  
* Não use o seu Leaner Lab porque ele não estará com as pré-configurações necessárias. Depois a gente vai falar mais sobre isso.

# Arquitetura inicial

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-starting-lab-architecture.png">
   <img alt="Região e Zonas AWS" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-starting-lab-architecture.png)">
</picture>

# Arquitetura final

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-final-lab-architecture.png">
   <img alt="Região e Zonas AWS" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-final-lab-architecture.png)">
</picture>

# Passo-01: Criar um grupo de segurança no RDS

## Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8.
## Mas antes de começar, pense! Por que o primeiro passo é criar um grupo de segurança?

[resposta](https://github.com/agodoi/EC2-RDS/blob/main/resposta1)


**(a)** Dentro do console AWS já logado na sua conta de estudante, pesquise por **VPC** (Virtual Private Cloud) no campo **Pesquisar**.

**(b)** Clique na caixinha **Grupo de Segurança** no meio da sua tela. É provável que você já veja uma lista de grupos criados, mas isso é devido às suas tarefas anteriores e às pré-configurações (feitos pela AWS) nesse exercício.

**- (b.1)** Clique em **Criar grupo de segurança** (botão laranja)

**- (b.2)** Em **Nome do grupo de segurança**: digite **Grupo Seguranca DB** (não use ç)

**- (b.3)** Em **Descrição**: **Permite acesso do grupo de seguranca Web** (não use ç)

**- (b.4)** Em **VPC**: escolha Lab VPC. Mas se não existir essa opção é porque você está usando o Leaner Lab e o correto é você está no Módulo 8 do curso AWS Academy Foundation.

Na mesma tela (não saia daquela tela) você adicionará uma regra ao grupo de segurança para permitir solicitações de entrada do banco de dados.

**(c)** Logo abaixo, tem **Regras de entrada**, e ele deve estar vazio. Nessa etapa, clique em **Adicionar regra** para adicionar uma regra para permitir acesso pelo **Grupo de segurança da Web**. Defina as seguintes configurações:

**- (c.1)** Em **Tipo**: MySQL/Aurora (3306)

**- (c.2)** Em **Origem**: deixa personalizado, e na lupinha,  pegue a opção **Web Security Group | sg- nº IP**

**- (c.3)** Em **Descrição**: digite "MinhaEntradaWebBD"

Isso configura o grupo de segurança de banco de dados para permitir tráfego de entrada na porta 3306 de qualquer instância do EC2 associada ao Web Security Group (Grupo de segurança da Web).

**- (c.4)** Pula a parte **Regras de saída** e **Tags opcionais**.

**- (c.5)** Clique no botão final **Criar grupo de segurança**.

Você usará esse grupo de segurança ao executar o banco de dados do Amazon RDS.

# Passo 02 - Criar um grupo de sub-redes de banco de dados
## Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8.

Nesta tarefa, você criará um grupo de sub-redes de banco de dados, que é usado para informar ao RDS quais sub-redes podem ser usadas com o banco de dados. Volte na arquitetura (desenho inicial daqui dessa instrução) para entender onde vão as sub-redes. Cada grupo de sub-redes de banco de dados requer sub-redes em pelo menos duas zonas de disponibilidade (de novo, observe a arquitetura inicial, que há 2 zonas A e B).

**(a)** No menu **Serviços** do console AWS, digite ou clique em **RDS**.

**(b)** No painel de navegação esquerdo vertical, clique em **Grupos de sub-redes**.

**(c)** Clique no botão laranja **Criar grupo de sub-redes de banco de dados** (ou Create DB Subnet Group) e configure:

**- (c.1)** Nome: **Grupo-Subrede-DB**

**- (c.2)** Descrição: **Grupo de sub-redes de banco de dados**

**- (c.3)** VPC: **Lab VPC**

**- (c.4)** Em **Adicionar sub-redes**, expanda a lista de valores em **Zonas de disponibilidade** e selecione as duas primeiras zonas: **us-east-1a** e **us-east-1b**. Note que a AWS permite disponibilizar até 6 zonas distintas do seu banco de dados. Isso vai de encontro com a alta disponibilidade do seu banco de dados. Quanto mais zonas, mais caro ficará o serviço final.

**- (c.5)** Expanda a lista de valores em **Sub-redes** e selecione as sub-redes associadas aos intervalos de CIDR **10.0.1.0/24** para us-east-1a e **10.0.3.0/24** para us-east-1b. Essas sub-redes devem agora ser mostradas na tabela Sub-redes logo abaixo aí do seu console.

**- (c.6)** Clique no botão laranja **Criar**.

Você usará esse grupo de sub-redes de banco de dados ao criar o banco de dados na próxima tarefa.

## Calma, fião! Repense sobre o que você acabou de fazer!

Nesta tarefa, você criou um grupo de sub-redes de banco de dados, que é usado para informar ao RDS quais sub-redes podem ser usadas com o banco de dados. Volte na arquitetura (desenho inicial daqui dessa instrução) para entender onde vão as sub-redes. Cada grupo de sub-redes de banco de dados requer sub-redes em pelo menos duas zonas de disponibilidade e na arquitetura inicial, há 2 zonas A e B. Mas por que 2 zonas? A resposta virá logo a seguir...


# Passo 03: Criar uma instância de banco de dados do Amazon RDS
## Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8.

Nesta tarefa, você configurará e executará uma instância de banco de dados **Multi-AZ** do Amazon RDS for MySQL.

As implantações **Multi-AZ** do Amazon RDS proporcionam melhor disponibilidades e durabilidade para instâncias de banco de dados, o que as torna a solução ideal para cargas de trabalho de banco de dados de produção. Quando você provisiona uma instância de banco de dados Multi-AZ, o Amazon RDS cria automaticamente uma instância de banco de dados principal e replica os dados de maneira síncrona para uma instância de espera em uma zona de disponibilidade (AZ) diferente [isso responde a pergunta acima: Mas por que 2 zonas?, certo?

**Cuidado** >> RDS consome seus créditos, portanto, ao finalizar essa instrução, **suspenda-o** caso esteja fazendo na raça do seu Leaner Lab. E você pode consultar o **AWS Cost Explorer** para saber dos custos.

**(a)** No painel de navegação esquerdo, clique em **Banco de dados**.

**(b)** Clique em **Criar Banco de Dados**.

Se aparecer esse alerta *Switch to the new database creation flow* (alternar para o novo fluxo de criação de banco de dados) na parte superior da tela, ok, clique nele.

**(c)** Deixei em **Criação padrão** que significa que você vai criar esse banco na força do ódio... rsrsr 

**(d)** Selecione  **MySQL**.

**(e)** Em **Versão do mecanismo**, você pode verificar que geralmente está pré-configura para a penúltima versão, que é a considerada estável. Deixe como está mesmo.

**(f)** Em **Modelos**, escolha **Nível gratuito**.

**(g)** Em **Configurações**, faça o seguinte:

**- (g.1)** Em **Identificador da instância de banco de dados**: digite **lab-db**

**- (g.2)** Em **Nome do usuário principal**: digite **admin**

**- (g.3)** Em **Senha principal**: senha-lab

**- (g.4)** Em **Confirmar sua senha**: senha-lab

**- (g.5)** Em **Configuração da instância**, deixe como **classes com capacidade de intermitência (inclui classes t)** e selecione **db.t3.micro**

**- (g.6)** Em **Armazenamento**, deixe como **SSD de uso geral (gp2)**, e em **armazenamento alocado** deixe **20 GB**

**- (g.7)** Em **Escalabilidade**, deixe tudo como está. Note que seu banco poderá armazenar até 1000GB .

**- (g.8)** Em **Conectividade**, deixe em **Não se conectar a um recurso de computação EC2**. Isso significa que você irá fazer algumas configurações manuais agora.

**- (g.9)** Em **Nuvem privada virtual (VPC)**: selecione **Lab VPC**. Pule as demais configurações até o próximo item a seguir.

**- (g.10)** Em **Grupos de segurança de VPC existentes**, marque **Grupo Seguranca DB** e desmarque **default**.

**- (g.11)** Pule alguns campos até chegar no campo a seguir.

**- (g.12)** Desmarque **Habilitar monitoramento avançado** (Habilitar monitoramento aprimorado).

**- (g.13)** Expanda  **Configuração Adicional**. Mas atenção! É o Configuração Adicional logo abaixo do Monitoramento, OK?

**- (g.13)** Em **Nome do banco de dados inicial**: digite **lab**. Essa opção está dando um apelido para o seu banco RDS.

**- (g.14)** Desmarque **Habilitar backups automatizados**.

Isso desativará os backups, o que normalmente não é recomendado, mas agilizará a implantação do banco de dados para este laboratório.

**(h)** Deixe tudo como está até o fim, e clique no botão laranja **Criar banco de dados**. Seu banco de dados agora será executado.

Caso apareça algumas telas de configurações extras ou sugestões, ignore-as!

Parece que nada aconteceu, mas se você subir sua tela até o top, verá uma faixa verde indicando sucesso na sua criação do banco de dados AWS RDS.

E você verá uma faixa azul indicando que o seu banco de dados **lab-db** está sendo criado. Aguarde até 4 minutos para a conclusão, pois o processo está implantando um banco de dados em duas zonas de disponibilidade diferentes.

Quando a faixa azul ficar verde, significa que seu banco **lab-db** está pronto.

**(i)** Role para baixo até a seção **Segurança e Conexão** e copie o campo **Endpoint**. Ele será semelhante a: **lab-db.css9whrgr6ve.us-east-1.rds.amazonaws.com**

Cole o valor do endpoint em um editor de texto. Você o usará isso mais tarde aqui no laboratório.
 

# Passo-04: Interagir com seu banco de dados
## Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8.

Nesta tarefa, você abrirá uma aplicação Web em execução no servidor da Web e o configurará para usar o banco de dados.

**(a)** Você precisa pegar o endereço IP do WebServer já pré-criado pela AWS. Para isso, clique no botão suspenso **Details** na sua tela onde começou o Lab Módulo 8 e, em seguida, clique em **Show**.

**(b)** Abra uma nova guia do navegador Web, cole o endereço IP de WebServer [enter]. A aplicação Web será exibida com informações sobre a instância do EC2. Se sua rede local bloquear essa etapa, roteie o sinal WiFi do seu celular para ver se resolve.

**(c)** Clique no link RDS na parte superior da página. Agora, você configurará a aplicação para se conectar ao banco de dados. Defina as seguintes configurações:

**- (c.1):** Endpoint: cole o endpoint que você copiou em um editor de texto anteriormente

**- (c.2):** Database: **lab**

**- (c.3):** Username: **main**

**- (c.4):** Password: **lab-password**

**- (c.5):** Clique em **Submit** (Enviar)

Uma mensagem será exibida explicando que a aplicação está executando um comando para copiar informações para o banco de dados. Após alguns segundos, a aplicação exibirá um [Address Book] (Catálogo de endereços).

A aplicação Address Book (Catálogo de endereços) está usando o banco de dados do RDS para armazenar informações.

**(d)** Adicione, edite e remova contatos para testar o aplicativo web.

Os dados estão sendo mantidos no banco de dados e são replicados automaticamente para a segunda zona de disponibilidade.


# Laboratório concluído
Parabéns! Você concluiu o laboratório.

# Desafio para casa:
## Faça uma investigação das pré-configurações do Grupo de Segurança chamado Web Security Group, EC2, VPC, IAM e subredes que estão sendo usadas nesse instrução, pois o AWS já deixou isso preparado para você antes de começar o lab.
