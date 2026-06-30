# EC2-RDS

## O que vamos fazer nessa aula?

Iremos apresentar os métodos para modelagem de banco de dados relacional.

Este laboratório foi criado para reforçar o conceito de utilização de uma instância de banco de dados gerenciado pela AWS para atender as necessidades de banco de dados relacional.

## Qual impacto no projeto atual?

A aula de hoje representa um dos primeiros passos da arquitetura de nuvem para resolver o problema da venda de ingressos, que certamente passará por um banco de dados relacional escalável e disponível.

## O que é um RDS?

O Amazon Relational Database Service (Amazon RDS) facilita a configuração, a operação e o escalamento de um banco de dados relacional na nuvem. 

Ele oferece capacidade econômica e redimensionável enquanto gerencia [tarefas demoradas](https://github.com/agodoi/EC2-RDS/blob/main/vantagens) de administração de banco de dados , permitindo que você se concentre nas suas aplicações e nos seus negócios. 

O Amazon RDS fornece seis opções de mecanismos de banco de dados familiares: Amazon Aurora, Oracle, Microsoft SQL Server, PostgreSQL, MySQL e MariaDB. Mas hoje, vamos focar no **MySQL**


## E o que o EC2 está fazendo aqui?

O EC2 é a aplicação que consome o banco do RDS, pois o RDS é só o banco gerenciado (não roda sua aplicação).

Portanto, o EC2 serve como servidor de aplicação (ex.: Flask, Node.js, Java, PHP, Django).

**O fluxo típico é:** usuário → navegador → EC2 (aplicação web/API) → RDS (dados).

Exemplo: você hospeda um site no Apache/Nginx no EC2, e esse site consulta o MySQL no RDS.

Muitas vezes você não quer expor o RDS à internet (por segurança).

Nesse caso, sobe um EC2 na mesma VPC/Subnet e o usa como jump host para administração e importação de dados:

- Você pode rodar mysql CLI, isto é, dar comandos mysql no terminal;
- Você pode executar scripts de carga de dados;
- Você pode usar ferramentas de ETL/Migração;
- Imagine que você recebe arquivos CSV, JSON ou logs. O EC2 pode baixar/processar/transformar esses dados, depois inserir no RDS;
- EC2 e RDS ficam na mesma VPC, a comunicação acontece via endereço privado, não passa pela internet;
- Security Group do RDS permite acesso somente ao SG do EC2.
- Isso reduz drasticamente exposição do banco.

Assim, você conecta primeiro no EC2 (SSH) e de lá acessa o RDS, sem abrir o banco para o mundo.

## Objetivos

Depois de concluir este laboratório, você será capaz de:

* Ter insights para o seu projeto da venda de ingressos, em como criar um banco de com dados de vendas;
* Executar uma instância de banco de dados do Amazon RDS com alta disponibilidade.
* Configurar a instância de banco de dados para permitir conexões do seu servidor web.
* Abrir um aplicativo web e interagir com seu banco de dados.

## Pré-requisitos

* Você precisa entrar no curso AWS Academy Foundation, ir até o **Módulo 8** e subir o **Laboratório 5 - Crie um servidor de banco de dados e interaja com o banco de dados usando um aplicativo**. Espera aparecer **Lab Status: ready** para abrir o console AWS. Você abre o console clicando na bolinha verde **AWS**.
  
* Não use o seu Learner Lab porque ele não estará com as pré-configurações necessárias do IAM. Depois a gente vai falar mais sobre isso.
  
* Sinto lhe informar, mas se demorar muito nessa atividade, a sessão vai se encerrar e todo o trabalho será perdido. Então vamos fazer uma overview primeiro antes de começar as configurações. Teremos 90min para concluir essa instrução antes de tudo cair.

## Arquitetura básica e inicial

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-starting-lab-architecture.png">
   <img alt="Arquitetura Inicial" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-starting-lab-architecture.png)">
</picture>

## Arquitetura final e profissa

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-final-lab-architecture.png">
   <img alt="Arquiteutura Final" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/lab-5-final-lab-architecture.png)">
</picture>

## Passo-01: Criar um grupo de segurança no RDS
### Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8. Fica de olho no cronômetro do laboratório [máx. 90min].
### Mas antes de começar, pense! Por que o primeiro passo é criar um grupo de segurança?

[resposta](https://github.com/agodoi/EC2-RDS/blob/main/resposta1)


**1.1** Dentro do console AWS já logado na sua conta de estudante, pesquise por **VPC** (Virtual Private Cloud) no campo **Pesquisar**.

**1.2** Clique na caixinha **Grupo de Segurança** no meio da sua tela. É provável que você já veja uma lista de grupos criados, mas isso é devido às suas tarefas anteriores e às pré-configurações (feitos pela AWS) nesse exercício.

**- (a)** Clique em **Criar grupo de segurança** (botão laranja)

**- (b)** Em **Nome do grupo de segurança**: digite **Grupo Seguranca DB** (não use **ç**)

**- (c)** Em **Descrição**: **Permite acesso do grupo de seguranca Web** (não use **ç**)

**- (d)** Em **VPC**: escolha Lab VPC. Mas se não existir essa opção é porque você está usando o Learner Lab e o correto é você está no **Módulo 8** do curso **AWS Cloud Foundation**, porque a AWS já preparou algumas pré-configurações para você usar. Essas pré-configurações são do EC2. Note que nessa instrução, não estamos subindo o EC2 manualmente, e sim, herdando...

Na mesma tela (não saia daquela tela) você adicionará uma regra ao grupo de segurança para permitir solicitações de **entrada** do banco de dados.

**(1.3)** Logo abaixo, tem **Regras de entrada**, e ele deve estar vazio. Nessa etapa, clique em **Adicionar regra** para adicionar uma regra para permitir acesso pelo **Grupo de segurança da Web**. Defina as seguintes configurações:

**- (a)** Em **Tipo**: MySQL/Aurora (3306)

**- (b)** Em **Origem**: deixa personalizado, e na lupinha,  pegue a opção **Web Security Group | sg- nº IP**

**- (c)** Em **Descrição**: digite "MinhaEntradaWebBD"

Isso configura o grupo de segurança de banco de dados para permitir tráfego de entrada na porta 3306 de qualquer instância do EC2 associada ao Web Security Group (Grupo de segurança da Web).

**- (d)** Pula a parte **Regras de saída** e **Tags opcionais**.

**- (e)** Clique no botão final **Criar grupo de segurança**.

Você usará esse grupo de segurança ao executar o banco de dados do Amazon RDS.

## Passo 02 - Criar um grupo de sub-redes de banco de dados
### Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8. Fica de olho no cronômetro do laboratório [máx. 90min].

Nesta tarefa, você criará um grupo de sub-redes de banco de dados e amarrar ao grupo de segurança que acabou de criar na etapa anterior, isto é, vamos informar ao RDS quais sub-redes podem se conectar a ele. Em outras palavras, quais sub-redes podem usar o banco de dados. 

Volte na arquitetura (desenho inicial daqui dessa instrução) para entender onde vão as sub-redes. Cada grupo de sub-redes de banco de dados requer sub-redes em pelo menos duas zonas de disponibilidade (de novo, observe a arquitetura inicial, que há 2 zonas A e B). Essa regra é uma regra da Multi-AZ da AWS.

**(2.1)** No menu **Serviços** do console AWS, digite ou clique em **RDS**.

**(2.2)** No painel de navegação esquerdo vertical, clique em **Grupos de sub-redes**.

**(2.3)** Clique no botão laranja **Criar grupo de sub-redes de banco de dados** (ou Create DB Subnet Group) e configure:

**- (a)** Nome: **Grupo-Subrede-DB**

**- (b)** Descrição: **Grupo de sub-redes de banco de dados**

**- (c)** VPC: **Lab VPC**

**- (d)** Em **Adicionar sub-redes**, expanda a lista de valores em **Zonas de disponibilidade** e selecione as duas primeiras zonas: **us-east-1a** e **us-east-1b**. Note que a AWS permite disponibilizar até 6 zonas distintas do seu banco de dados. Isso vai de encontro com a alta disponibilidade do seu banco de dados. Quanto mais zonas, mais caro ficará o serviço final.

**- (e)** Expanda a lista de valores em **Sub-redes** e selecione as sub-redes associadas aos intervalos de CIDR **10.0.1.0/24** para us-east-1a e **10.0.3.0/24** para us-east-1b. Essas sub-redes devem agora ser mostradas na tabela Sub-redes logo abaixo aí do seu console.

**- (f)** Clique no botão laranja **Criar**.

Você usará esse grupo de sub-redes de banco de dados ao criar o banco de dados na próxima tarefa.

## Calma! Repense sobre o que você acabou de fazer!

Nesta tarefa, você criou um grupo de sub-redes de banco de dados, que é usado para informar ao RDS quais sub-redes podem ser usadas com o banco de dados. Volte na arquitetura (desenho inicial daqui dessa instrução) para entender onde vão as sub-redes. Cada grupo de sub-redes de banco de dados requer sub-redes em pelo menos duas zonas de disponibilidade e na arquitetura inicial, há 2 zonas A e B. Mas por que 2 zonas? A resposta virá logo a seguir...

Note que o EC2 possui um grupo de segurança e o RDS, outro grupo de segurança.

Importantíssimo!!! Seu RDS *não está com entrada 0.0.0.0/0* que significa qualquer um de qualquer lugar. Se você fizesse isso, seu banco de dados seria invadido em alguns minutos por alguém na Internet. Portanto seu RDS possui como regra de entrada, um endereço interno na AWS que coincide com a saída do EC2 (que já está pronto).


## Passo 03: Criar uma instância de banco de dados do Amazon RDS
### Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8. Fica de olho no cronômetro do laboratório [máx. 90min].

Nesta tarefa, você configurará e executará uma instância de banco de dados **Multi-AZ** do Amazon RDS for MySQL que vai criar 2 zonas.

As implantações **Multi-AZ** do Amazon RDS proporcionam melhor disponibilidades e durabilidade para instâncias de banco de dados, o que as torna a solução ideal para cargas de trabalho de banco de dados de produção. Quando você provisiona uma instância de banco de dados Multi-AZ, o Amazon RDS cria automaticamente uma instância de banco de dados principal e replica os dados de maneira síncrona para uma instância de espera em uma zona de disponibilidade (AZ) diferente [isso responde a pergunta acima: Mas por que 2 zonas?], certo?

**Cuidado** >> RDS consome seus créditos, portanto, caso esteja fazendo essa instrução no Learner Lab, **suspenda-o** quando finalizar as tarefas. E você pode consultar o **AWS Cost Explorer** para saber dos custos. Mas como estamos fazendo no laboratório pré-configura do Módulo 8, tudo será desligado assim que você fechar seu navegador.

**(3.1)** No painel de navegação esquerdo, clique em **Banco de dados**.

**(3.2)** Clique em **Criar Banco de Dados**.

Se aparecer esse alerta *Switch to the new database creation flow* (alternar para o novo fluxo de criação de banco de dados) na parte superior da tela, ok, clique nele.

**(3.3)** Deixei em **Criação padrão** que significa que você vai criar esse banco na força do ódio... rsrsr 

**(3.4)** Selecione  **MySQL**.

**(3.5)** Em **Versão do mecanismo**, você pode verificar que geralmente está pré-configura para a penúltima versão, que é a considerada estável. Deixe como está mesmo.

**(3.6)** Em **Modelos**, escolha **Nível gratuito**.

**(3.7)** Em **Configurações**, faça o seguinte:

**- (a)** Em **Identificador da instância de banco de dados**: digite **lab-db**

**- (b)** Em **Nome do usuário principal**: digite **admin**

**- (c)** Em **Senha principal**: digite **lab12345**

**- (d)** Em **Confirmar sua senha**: digite **lab12345**

**- (e)** Em **Configuração da instância**, deixe como **classes com capacidade de intermitência (inclui classes t)** e selecione **db.t3.micro**

**- (f)** Em **Armazenamento**, deixe como **SSD de uso geral (gp3)**, e em **armazenamento alocado** deixe **20 GB**

**- (g)** Em **Escalabilidade**, deixe tudo como está. Note que seu banco poderá armazenar até 1000GB .

**- (h)** Em **Conectividade**, deixe em **Não se conectar a um recurso de computação EC2**. Isso significa que você irá fazer algumas configurações manuais agora.

**- (i)** Em **Nuvem privada virtual (VPC)**: selecione **Lab VPC**. Pule as demais configurações até o próximo item a seguir.

**- (j)** Em **Grupos de segurança de VPC existentes**, marque **Grupo Seguranca DB** e desmarque **default** (padrão).

**- (k)** Pule alguns campos até chegar no campo a seguir.

**- (l)** Confirme se está desmarque **Habilitar monitoramento avançado**.

**- (m)** Expanda  **Configuração Adicional**. Mas atenção! É o Configuração Adicional logo abaixo do Monitoramento, OK?

**- (n)** Em **Nome do banco de dados inicial**: digite **lab**. Essa opção está dando um apelido para o seu banco RDS.

**- (o)** Desmarque **Habilitar backups automatizados**.

Isso desativará os backups, o que normalmente não é recomendado, mas agilizará a implantação do banco de dados para este laboratório.

**- (p)** Desmarque **Habilitar criptografia**.

**(3.8)** Deixe tudo como está até o fim, e clique no botão laranja **Criar banco de dados**. Seu banco de dados agora será executado.

Caso apareça algumas telas de configurações extras ou sugestões como a figura a seguir, ignore-as!

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/sugestao_ignorar.png">
   <img alt="Sugestão Ignoradas" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/sugestao_ignorar.png)">
</picture>

Parece que nada aconteceu, mas se você subir sua tela até o top, verá uma faixa verde indicando sucesso na sua criação do banco de dados AWS RDS.

E você verá uma faixa azul indicando que o seu banco de dados **lab-db** está sendo criado. Aguarde até 4 minutos para a conclusão, pois o processo está implantando um banco de dados em duas zonas de disponibilidade diferentes.

Quando a faixa azul ficar verde, significa que seu banco **lab-db** está pronto.


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/banco_sucesso.png">
   <img alt="Sugestão Ignoradas" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/banco_sucesso.png)">
</picture>


**(3.9)** Role para baixo até a seção **Conectividade e segurança** e copie o campo **Endpoint**. Ele será semelhante a: **lab-db.css9whrgr6ve.us-east-1.rds.amazonaws.com**

Cole o valor do endpoint em um editor de texto. Você o usará isso mais tarde aqui no laboratório.
 

## Passo-04: Interagir com seu banco de dados
### Atenção: essa etapa será perdida ao encerrar sua sessão no laboratório do Módulo 8. Fica de olho no cronômetro do laboratório [máx. 90min].

Nesta tarefa, você abrirá uma aplicação Web em execução no servidor da Web e o configurará para usar o banco de dados.

**(4.1)** Você precisa pegar o endereço IP do WebServer já pré-criado pela AWS. Para isso, vá em EC2 no console e pegue o endereço IP4 público. Geralmente é um IP da faixa de 1 a 127 no primeiro octeto.

**(4.2)** Abra uma nova guia do navegador Web, cole o endereço IP de WebServer [enter]. A aplicação Web será exibida com informações sobre a instância do EC2 (figura a seguir). Se sua rede local bloquear essa etapa, roteie o sinal WiFi do seu celular para resolver e fique tranquilo que seu laboratório não será encerrado, desde que você não atualize nenhuma página enquanto chaveia o seu WiFi para o seu celular.

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/telafinal.png">
   <img alt="Aplicação Final" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/telafinal.png)">
</picture>

**(4.3)** Clique no botão RDS da aplicação (na parte superior da página). Agora, você configurará a aplicação para se conectar ao banco de dados. Defina as seguintes configurações:

**- (a):** Endpoint: cole o endpoint que você copiou em um editor de texto anteriormente (**Passo 3.9**)

**- (b):** Database: **lab** (nome que você deu no **Passo 3.7 letra n**)

**- (c):** Username: **admin** (nome que você deu no **Passo 3.7 etapa a**)

**- (d):** Password: **lab12345** (senha que você deu no **Passo 3.7 etapa c**)

**- (e):** Clique no botão **Submit** ou **Enviar**

Uma mensagem será exibida explicando que a aplicação está executando um comando para copiar informações para o banco de dados. Após alguns segundos, a aplicação exibirá um [Address Book] (Catálogo de endereços). Veja a figura abaixo. Você deve observar algo semelhante.

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/EC2-RDS/blob/main/imgs/resultadofinal.png">
   <img alt="Aplicação Final" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/EC2-RDS/blob/main/imgs/resultadofinal.png)">
</picture>

A aplicação Address Book (Catálogo de endereços) está usando o banco de dados do RDS para armazenar informações.

**(4.4)** Adicione, edite e remova contatos para testar o aplicativo web.
.
Os dados estão sendo mantidos no banco de dados e são replicados automaticamente para a segunda zona de disponibilidade.


# Laboratório concluído
Parabéns! Você concluiu o laboratório.

# Questionário [valendo a ponderada]
## Faça uma investigação das pré-configurações desse laboratório para entender melhor o que aconteceu:

### * Grupo de Segurança chamado Web Security Group;
### * EC2;
### * VPC;
### * IAM;
### * Subredes

Procedimentos:
* Pesquise cada palavra-chave acima na lupa do console e tome nota em um caderno dos campos configurados e pule os campos padrões.
* Não precisa mostrar para o professor, mas servirá para você completar o seu aprendizado.
* Alguns acessos dentre as palavras-chave acima estarão com bloqueio de acesso devido ao grupo de segurança pré-criado. Isso serve para a gente não modificar a aplicação do servidor :(
* O WebServer foi desenvolvido em Bootstrap pela AWS. [Bootstrap](https://getbootstrap.com/)
