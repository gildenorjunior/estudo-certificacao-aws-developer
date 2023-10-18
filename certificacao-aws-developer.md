## EBS Volume Types

#### EBS Volumes tem 6 tipos

- **gp2 / gp3 (SSD)**: É para um propósito geral, um volume SSD que tem um balanço de performance e preço e abrange uma grande variedade de cargas de trabalho.
- **io1 / io2 (SSD)**: É utilizado para cargas de trabalho que exigem grande performance e de grande criticidade com baixa latência ou um throughput alto.
- **st1 (HDD)**: Um HDD de baixo custo, feito para cargas de trabalho que exigem acesso frequente.
- **sc1 (HDD)**: O HDD mais barato, feito para cargas de trabalho que não exigem um acesso tão frequente.

#### EBS Volumes são caracterizados por Tamanho | Throughput | IOPS (I/O Ops Per Sec - Entrada e Saída por segundo)

#### Apenas gp2/gp3 e io1/io2 podem ser usados para volumes de boot

## EBS Volumes Types Use cases

### General Purpose SSD
- É um armazenamento econômico, com baixa latência
- Pode ser usado para boot volumes, Virtual desktops, ambientes de desenvolvimento e de testes
- O tamanho pode variar entre 1GB e 15 TB

**gp2**
- É uma geração mais antiga
- Tem pequenos volumes que podem romper IOPS até 3 mil
- O tamanho do volume e IOPS **são ligados**, maximo IOPS é 16 mil

**gp3**
- É a geração mais nova
- Da uma base de 3 mil IOPS e um throughput de 125 megabytes por segundo
- Podemos aumentar o IOPS até 16 mil e o throughput até 1000 MB/s **independentemente**
  
### Provisioned IOPS (PIOPS) SSD
- É um bom caso para quando se tem uma aplicação comercial crítica que necessita de sustentação de performance de IOPS
- Ou aplicações que necessitem de mais de 16 mil IOPS
- Bom uso para Banco de Dados que são sensiveis para o armazenamento, performance e consistência

**io1 / io2 (4GB - 16TB)**
- Maximo PIOPS: 64 mil para instancias Nitro e 32 mil para outras
- Pode aumentar PIOPS **independentemente** do tamanho da instância
- io2 tem mais durabilidade e mais IOPS por GB (e o mesmo preço da io1)

**io2 Block Express (4GB - 64TB)**
- Uma latência de menos de milisegundos
- Maximo de PIOPS 256 mil com um IOPS GB ratio de 1 mil
- Suporta EBS Multi-attach

### Hard Disk Drives (HDD)
- Não servem para ser volumes de boot
- Tamanhos de 125 GB até 16 TB

**Throughput Optimized HDD (st1)**
- Bons para Big Data, Data Warehouses, Log Processing
- Maximo de throughout 500 MB, maximo IOPS 500

**Cold HDD (scl)**
- Bom para dados que não são acessados frequentemente
- Cenários onde o baixo custo é mais importante
- Maximo throughput 250 MB, maximo IOPS 250

![sumário de comparação entre tipos](./imagens/Captura%20de%20tela%20de%202023-08-22%2021-59-13.png)

## EBS Multi-Attach - io1 / io2 family

- Com esta familia podemos atachar o mesmo volume EBS a multiplas instancias EC2 no mesmo AZ.
- Cada instância tem full permissão de leitura e escrita para uma performance maior do volume.

**Caso de uso:** 

Uma aplicação que com maior disponibilidade no caso de um cluster Linux, exemplo: Teradata

Ou um caso que deve gerenciar multiplas operações de escrita em paralelo

- Um volume só pode ser atachado até no máximo 16 instâncias ao mesmo tempo

## Amazon EFS - Elastic File System

É um gerenciador de sistema de arquivos de rede (network file system) e que pode ser monstado em várias instâncias EC2

- Essas instâncias podem estar em diferentes AZs
- Tem alta disponibilidade, escalabilidade, tem um alto custo (3x gp2), e o pagamento é por uso

![funcionamento efs](./imagens/funcionamento-efs.png)

- **Casos de uso:** Gerenciamento de conteúdo, web serving, compartilhamento de dados, Wordpress
- Usa o protocolo NFSv4
- Usa o security group para o controle de acesso ao EFS
- Compativel com Linux based AMI (não compativel com Windows)
- Criptografa os dados em repouso usando KMS
- Utiliza o sistema de arquivos POSIX (~Linux) que possui um arquivo API padrão
- Não é preciso planejar com antecedência a capacidade, ele escala automaticamente
- Pago por uso de GB, não por capacidade

### Storage Classes

#### Storage Tiers (gerenciamento do ciclo de vida, move um arquivo depois de N dias)
- Padrão: para arquivos acessados recentemente
- Infrequent access (EFS-IA): Recuperar arquivos infrequentes, o preço é menos para esse armazenamento e para habilitar esse armazenamento você deve usar uma política de ciclo de vida

![efs-ia exemplo](./imagens/efs-ia-exemplo.png)

#### Disponibilidade e durabilidade
- Padrão: Multi-AZ, bom para cenários de produção
- One Zone: Uma zona de disponibilidade, bom para ambiente de dev, backup habilitado por padrão, compativel com IA (EFS One Zone-IA)

Pode ter descontos de até 90%

## EBS vs EFS

### EBS
- Só podem ser anexados a uma instância de cada vez, exceto o multi-attach da familia **io1 / io2**
- São travados por Zona de disponibilidade, só pode atachar um volume em uma instância da no mesmo AZ
- gp2 IO aumenta se o tamanho do disco aumentar
- io1 pode aumentar o IO **independentemente**
- Para migrar um volume entre zonas diferentes é preciso tirar uma snapshot e restaura-lo na outra zona de destino
- EBS backups usa o IO e não deve ser excutado enquanto sua aplicação estiver lidando com bastante trafego por que isso pode afetar o desempenho
- Em volumes Root a opção para terminated quando a instancia for 'terminated' vem habilitada por padrão, mas podemos desabilitar esta opção

![ebs entre azs](./imagens/ebs-entre-azs.png)

### EFS
- O objetivo é trabalhar entre diferentes AZs
- Podemos compartilhar por exemplo arquivos de um website WordPress
- É apenas para instancias Linux por causa do sistema (POSIX)
- Tem um custo maior que o EBS, mas podemos economizar usando o EFS-IA (Elastic File System Infrequent Access)

![exemplo de funcionamento de EFS](./imagens/exemplo-efs.png)

Relembrando **Instance Store** é a armazenagem fisica instantanea, ou seja se a instância parar você perde esse armazenamento.

## ELB + ASG

#### Scalability & High Availability
Escalabilidade significa que a aplicação pode e deve se adpatar a maiores cargas de trabalho. <br> 
Existe dois tipo de escalabilidade: Vertical e Horizontal. <br>
Escalabilidade e Disponibilidade estão ligadas mas não são a mesma coisa.

#### Escalabilidade Vertical
- Significa você aumentar o tamanho da sua instância basicamente
- Escalabilidade vertical é muito comum em sistemas não distribuidos, como por exemplo, banco de dados, RDS, ElasticCache...
- Essa escalabilidade pode ter limite, esse limite é o hardware.

#### Escalabilidade Horizontal
- Significa escalar com mais instancias
- Esse tipo é usado em sistema distribuidos, bem comum em aplicações modernas, web app por exemplo
- É facil de escalar horizontalmente, basicamente é só subir mais instâncias ec2

#### Alta Disponibilidade
É você deixar seu sistema funcionando mesmo com a perda de algum data center. É você ter redundância. Você rodar em mais de uma zona de disponibilidade ou região.
<br><br>
Existe a disponibilidade passiva e ativa, passiva é quando o serviço que estamos usando já é multi AZ por exemplo. Ativa é quando temos na nossa arquitetura a distribuição em mais de uma zona de disponibilidade.

## Elastic Load Balancing (ELB)
Serve para coordenar o tráfego de requisições e distribuir entre os multiplos servidores.
<br>

![exemplo de funcionamento ELB](./imagens/elb.png)

Temos três instâncias EC2, um balanceador de carga e três usuários fazendo requisição. O balanceador vai interceptar essas requisições e distribuir da melhor forma para cada servidor, não deixando que nenhum deles fique sobrecarregado.

**Por que utilizar um balanceador de carga?**
- Ele distribui a carga entre as multiplas instancias
- É exposto apenas um ponto de acesso (DNS) a aplicação
- Ajuda a lidar com falhas e quedas de instâncias, uma vez que ele só vai mandar carga para instâncias que estiverem de pé
- Ele vai regularmente health checks nas instancias
- Provê terminação SSL (HTTPS) para os websites
- Alta disponibilidade entre zonas
- Separa trafego publico de trafego privado

### Tipos de Load Balancer
A AWS tem 3 tipos de balanceador de carga

- **Application Load Balancer** (v2 - nova geração) - 2016 ALB, HTTP, HTTPS, WebSocket
- **Network Load Balancer** (v2 - nova geração) - 2017 NLB, TCP, TSL, UDP
- **Gateway Load Balancer** - 2020 GWLB, Opera nas três camadas (Camadas de rede), IP Protocol

É recomendado sempre usar o balanceadores da mais nova geração.

#### Load Balancer Security Groups
![Exemplo de como funciona o grupo de segurança do balanceador](./imagens/security-group-elb.png)

O grupo de segurança do load balancer vai dar acesso vindo de qualquer ip de origem, por afinal os usuário vão estar acessando e batendo nesse elb. Aceitando aqui no caso as portas HTTP e HTTPS.

<br>

Já o grupo de segurança da instância nesse caso, vai usar o protocolo HTTP na porta 80, permitindo trafego de origem apenas do grupo de segurança do ELB e não mais de um range de ip. Porque aqui só queremos aceitar requisições vindas do ELB e não diretamente do usuário.

---

### Entrando mais um pouco a fundo em cada tipo de Load Balancer

#### Application Load Balancer (v2)
- Trabalha na camada 7 (HTTP)
- Faz o balanceamento entre multiplas aplicações entre maquinas (target groups)
- Faz o balanceamento entre multiplas aplicações na mesma maquina (ex: containers)
- Tem suporte para HTTP/2 e WebSocket
- Também suporta redirects de HTTP para HTTPS por exemplo
- Consegue fazer o roteamento de rotas em diferentes target groups
    Conseguimos por exemplo, mudar o path da rota URL, o hostname da rota do site, os query strings ou headers também
- Esse tipo é ótimo para micro serviçoes e aplicações baseadas em container, ex: Docker e Amazon ECS
- ALB pode rotear para multiplos target groups

![Como funciona o application load balancer](./imagens/application-load-balancer-taget-group.png)

Aqui um exemplo de como o Application Load Balancer funciona. Podemos ter dois microsserviços diferentes e com rotas também diferentes dentro de target group, no meio o ALB e ele vai conseguir saber diferenciar as requisições com as diferentes rotas para seu respectivo microserviço.

### O que é um target group?

- Podem ser instâncias EC2 (gerenciadas por um Auto Scaling Group) - HTTP
- Podem ser tasks ECS (gerenciadas por ECS mesmo) - HTTP
- Podem ser lambdas functions, a requisição HTTP vai ser traduzida em um evento JSON
- Podem ser um endereço de IP, mas devem ser IPs privados

![Como funciona o application load balancer com query string](./imagens/target-group-query-string.png)

Atráves de query strings o ALB também consegue fazer redirecionamentos.

- APL também consegue trabalhar com hostnames fixos, (xxx.region.elb.amazonaws.com)
- A aplicação não vê qual foi o IP do client diretamente, ele consegue ver o ip nos headers da requisição, X-Fowarded-For
- Da mesma forma a porta, X-Forwarded-Port e protocolo X-Forwarded-Proto

![Como a aplicação vê os ips do client](./imagens/ips-alb.png)

---
### Network Load Balancer (v2)

É um balanceador de carga na camada de rede, (layer 4). Permite que:

- Trafego TCP e UDP para suas instancias
-  É muito eficiente e pode lidar com milhões de requisições por segundo
-  A latência é reduzida em comparação com ALB ~100 ms (vs 400 ms para ALB)
-  NLB tem apenas um IP estático por AZ, e você pode atribuir um IP elástico para cada AZ. (Isso é bom quando você precisa expor sua aplicação com um conjunto de IPs estáticos e estes podem ser IPs elásticos).
-  NLB é usado para quando precisamos de performance extrema, TCP ou UDP.
- **Sua utilização não está incluida no modo free tier da AWS**

<br>
Como funciona o Network Load Balancer:

![Como funciona o Network Load Balancer](./imagens/network-load-balancer.png)

#### Network Load Balancer - Target Groups
- Instâncias EC2

![NLB com instâncias EC2](./imagens/nlb-ec2.png)

- Endereço IP. Devem ser IPs privados

![NLB com IPs privados](./imagens/nlb-ips-privados.png)

- Application Load Balancer

![NLB ALB](./imagens/nlb-alb.png)

*Ponto importante para o exame*, Health Checks suportam os protocolos TCP, HTTP e HTTPS.

---
### Gateway Load Balancer

É usado para implantar, dimensionar e gerenciar uma frota de dispositivos neutros de rede de terceiros na AWS.

**Exemplo de uso:** Você usuaria um GLB se quisesse que todo o tráfego de sua rede passasse por um firewall que você possui ou por um sistema de detecção e prevenção de instrusão para IDPs ou um sistema de inspeção profunda de pacotes ou se deseja modificar algumas cargas úteis.

![Como funciona o gateway load balancer](./imagens/gateway-load-balancer.png)

- GLB opera na camada de pacotes, layer 3 (Network)

*Para o exame o GLB usa o GENEVE protocolo na porta 6081*

#### Gateway Load Balancer - Target Groups

- Instâncias EC2

![GLB instâncias EC2](./imagens/glb-ec2.png)

- Endereços de IP. Devem ser IPs privados

![GLB instâncias IPS](./imagens/glb-ips.png)

<br>

## Cross-Zone Load Balancing

Balanceamento cruzado, significa que entre diferentes zonas de disponibilidade o balanceador vai distribuir a carga igualmente entre cada instância nas diferentes AZs. Isso pode ser um comportamente que você queira ou não.

![Como funciona o Cross-Zone ALB](./imagens/cross-zone-alb.png)

<br>

Agora sem essa função de cross zone: 
![Sem cross zone ALB](./imagens/sem-cross-zone-alb.png)

As cargas são distribuidas apenas entre a zona que a carga chega.


#### Application Load Balancer
- Vem habilitado por padrão o cross-zone, mas pode ser desabilitado no nível do Target Group
- Não há cobrança para mudança de dados entre os AZs

#### Network Load Balancer & Gateway Load Balancer
- O cross-zone vem desabilitado por padrão
- Você paga pelas mudanças dos dados entre as zonas de disponibilidade, se habilitar o cross-zone

## Elastic Load Balancer SSL certificates

Um certificado SSL permite o tráfego entre seus clientes e seu balanceador de carga seja criptografado durante trânsito.
Significa que os dados enquanto estão na rede sejam criptografados e só sejam descriptografados quando estiverem com seu remetente ou receptor.

- **SSL** refere-se a Secure Sockets Layer, usado para criptografar conexões.
- **TLS** refere-se a Transport Layer Security, que é um nova versão do SSL
- Atualmente TSL é usado com mais frequência (mas as pessoas se referem a ele como SSL).
- Os certificados tem data de expiração e devem ser renovados.

<br>

Como funciona o certificado SSL na ALB:
![Como funciona o certificado SSL na ALB](./imagens/ssl-certificado-alb.png)

- Você pode gerenciar certificados usando o ACM (AWS Certificate Manager)
- Você pode fazer o upload do seu próprio certificado alternativamente
- O load balancer usa um X.509 certificado (SSL/TLS server certificate)

#### SNI - Server Name Indication

- SNI resolve um problema que é o carregamente de multiplos certificados SSL dentro de um servidor Web (para servir multiplos websites)
- É um "novo" protocolo, e requer que o cliente indique o hostname do servidor alvo no handshake inicial
- O servidor vai então procurar o certificado correto, ou retornar o padrão.

**SNI só funciona para ALB, NLB (nova geração) e CLoudFront**

<br>

Como funciona o SNI:
![Como funciona o SNI](./imagens/sni.png)

## Elastic Load Balancer - Deregistration Delay

É um tempo para completar as requisições que estão em andamento, enquanto a instância está sendo desregistrada ou está unhealthy


Como funciona:
![Como funciona o Deregistration Delay](./imagens/deregistration-delay.png)

## Auto Scaling Groups (ASG)

**O que é?**
<br>
Basicamente é o esquema de redimensionar horizontalmente e automaticamente a quantidade de instâncias para aumentar ou diminuir a carga.

- Podemos adicionar instâncias
- Podemos diminuir instâncias
- Podemos assegurar uma quantidade minima e máxima de instâncias que ficarão rodando
- Ele consegue re-criar instâncias que foram terminadas, por exemplo se uma intâncias estava unhealthy
- ASG é grátis, só pagará pelos recursos usados abaixo dele, como por exemplo, instâncias EC2.

**Como funciona o ASG:**
![Como funciona o ASG](./imagens/auto-scaling-group.png)

<br>

**Como funciona o ASG com ELB**
![Como funcioan o ASG com ELB](./imagens/asg-e-elb.png)

### Auto Scaling Group Attributes

Para o auto scaling group a gente precisa especificar alguns atributos que vão servir de base para quando novas instâncias forem lançadas.

Alguns desses atributos são:

- AMI + Tipo da instância
- EC2 User Data
- EBS Volumes
- Security Groups
- SSH Key Pair
- IAM Roles para as instâncias EC2
- Informações de Network + Subnets
- Informações de Load Balancer

Além disso o Auto Scaling Group também precisa de um tamanho minimo, um tamanho máximo, uma capacidade inicial e também politicas de dimensionamento.

### Auto Scaling - CloudWatch Alarms & Scaling

Podemos integrar o auto scaling group junto ao cloudwatch para que sempre que tiver algum alarme em algum métrica definida por nós o auto scaling atue tanto aumentando o número de instâncias quanto diminuindo o número de instâncias.

![Auto scaling junto ao CloudWatch](./imagens/asg-e-cloudwatch.png)

### Auto Scaling Groups - Dynamic Scaling Policies

**- Target Tracking Scaling**
  - Mais simples e fácil de configurar
  - Exemplo: Eu quero que a média de CPU no meu ASG fique por volta de 40%

**- Simple / Step Scaling**
  - Quando o cloudwatch é acionado (exemplo: CPU > 70%) então adiciona 2 instâncias

**- Scheduled Actions**
  - Antecipar o escalonamento baseado em padrões conhecidos de uso
  - Exemplo: Aumente a capacidade minima para 10 as 5 horas as Sextas feiras

### Auto Scaling Groups - Predictive Scaling

- Basicamente uma analise preditiva de escalonamento, a carga será analisada de acordo com o histórico passado

### Boas métricas para escalas

- Utilização de CPU
- RequestCountPerTarget
- Média de Network In / Out
- Qualquer métrica que você colocou no cloudwatch

### Auto Scaling Groups - Scaling Cooldowns

- Depois que uma atividade de escalonamento acontece, você fica em um periodo de cooldown padrão de 300 segundos.
- Durante esse periodo o ASG não vai lançar ou terminar instâncias para permitir que as métricas se estabilizem.

### Auto Scaling - Instance Refresh

O objetivo é que você queria atualizar um grupo inteiro graças a um novo template que você lançou e então re-criar todas instâncias EC2 novamente.

Basicamente definimos uma porcentagem minima de unhealthy para as instâncias e conforme elas vão sendo encerradas, novas instâncias vão se criando com o novo modelo que pedimos.

![Como funciona o instance refresh](./imagens/instance-refresh.png)

## Amazon RDS Overview

RDS significa Relation Database Service. É um serviço gerenciado de banco de dados para bancos de dados que usam SQL como uma linguagem de consulta.
<br>
Permite que você crie bancos de dados na nuvem que serão gerenciados pela AWS.

Os motores gerenciados pela AWS são:
- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Aurora (da própria AWS)

#### Vantagens de usar RDS vs Deployar seu próprio DB em um EC2

Pelo RDS ser um serviço gerenciado ele provê:

- Provisionamento automatico de OS
- Backup e Restore de um timestamp especifico (Point in Time Restore)
- Monitoração de dashboards
- Leitura de replicas para melhorar a performance de leitura
- Configuração de Multi AZ para Disaster Recovery
- Manutenção de atualizações de Windows
- Escalabilidade tanto vertical quanto horizontal
- Storage pelos EBS (gp2 ou ioL)

### RDS Storage Auto Scaling

É uma feature do RDS que ajuda a aumentar seu armazenamento do SB dinamicamente. Quando o RDS detecta que você está rodando sem espaço livre de armazenamento, ele escala automáticamente.
<br><br>
Isso evita que você escale manualmente os armazenamentos.
Você tem que configurar o **Maximum Storage Threshold** limite máximo para o armazenamento do DB.

Ele automaticamente modifica o armazenamento se:
- Espaço livre for menos que 10%
- Está em capacidade baixa pelos ultimos 5 minutos
- Se passou 6 horas desde a última modificação

Isso é muito bom para aplicações que tem carga de trabalho imprevisiveis.

Tem suporte para todas as engines de DB do RDS (MariaDB, MySQL, PostgreSQL, SQL Server, Oracle)

**Como funciona o auto scaling do RDS:**
![Como funciona o auto scaling do RDS](./imagens/storage-auto-scaling-rds.png)

---

### RDS Read Replicas for read scalability

Basicamente, read replicas são mais instâncias que são criadas para ajudar na leitura da aplicação em acesso aos dados. Para que não se crie gargalo na instância do DB em relação a aplicação.

- Podemos ter até 15 replicas
- Elas podem atuar dentro de AZs, Cross AZs ou Cross Regiões
- A replicação acontece de forma assincrona
- As replicas podem ser promovidas para ser o próprio DB
- Replicas servem apenas para **leitura**

**Como funciona o RDS replicas:**

![Como funciona o RDS replicas](./imagens/rds-replicas.png)

#### Exemplo de caso de uso para RDS Replicas

Você tem uma aplicação produtiva consumindo de uma instância RDS, a carga de trabalho é uma carga normal.
Você agora quer rodar uma outra aplicação de report de dados para ter alguns analiticos.
Então você cria uma nova Read Replica para rodar esse novo worload lá. A sua aplicação antiga não vai ser afetada por esse novo trafego.

**Diagrama de exemplo:**
![Exemplo de caso de uso Read Replicas](./imagens/rds-replicas-use-case.png)

### Read Replicas - Custos

Normalmente a AWS cobra por dados que cruzam de um AZ para outro. Mas com serviços que são gerenciados isso não acontece. Para o RDS Read Replicas que estiverem dentro da mesma região, você não paga. 

Aqui está um diagrama de exemplo de custos, replicas em diferentes AZs, mas na mesma região o custo é grátis. Agora replicas que estão em diferentes regiões, tem um custo.

![Exemplo de custos com rds replicas](./imagens/rds-replcias-custos.png)

### RDS Multi AZ (Disaster Recovery)

- SYNC replicação
- Um DNS name
- Aumentamos a disponibilidade

![Exemplo de RDS multi AZ](./imagens/rds-multi-az.png)

### Como passa de um Single-AZ para Multi-AZ RDS

- Aqui não precisamos ter tempo de parada no banco de dados
- Apenas com um click você modifica

Por trás dos panos o que acontece é o seguinte, é tirado um snapshot do db, e é restaurado no novo db em um novo AZ. A sincronização é estabelecidade entra os dois databases.

![Como funciona por de trás dos panos o single az para multi az](./imagens/rds-from-single-az-to-multi-az.png)

## Amazon Aurora

É um serviço de banco de dados relacional criado pela AWS.

- Não é open source 
- Postgres e MySQL são suportados como Aurora, significa que seus drivers vão funcionar no Aurora assim como funcioanavam no Postgre ou MySQL.
- Aurora é otimizado para a nuvem
- Ele cresce automaticamente conforme o storage precisa
- Aurora pode ter mais de 15 replicas 
- Failover no Aurora é instantaneo
- Ele tem um custo maior que o RDS, mas é mais eficiente.

### Aurora Alta disponibilidade e Read scaling

- Ele faz 6 cópias dos seus dados entre 3 AZs
- Ele faz um espécie de auto correção caso os dados estejam corrompidos

### RDS & Aurora Security

**Para criptografia em repouso:**
- Você poder ter seus dados criptografados, para isso no DB master e Replicas usa-se AWS KMS. Isso é definido logo no primeiro inicio de launch do banco de dados.
- Se o DB master não foi criptografado por algum motivo, as replicas também não poderão ser criptografadas.
- Se você tem um DB que não está criptografado e você quer criptografa-lo, o que você tem que fazer é criar uma snapshot e restaurá-la em outro DB criptografado.


**Para criptografia em trânsito**

- Os DB do Aurora já vem por padrão com essa criptografia em trânsito, para isso os clients devem utilizar os certificados root AWS TLS

**Sobre Autenticação**

- Tem o padrão de usuário e senha, mas também podemos usar IAM roles para conectar ao banco de dados

**Segurança**

- Podemos utilizar Security groups para controlar o acesso a rede do nosso banco de dados, por exemplo qual porta habilitamos ou qual IP.

**Não é disponivel utilizar SSH no Aurora ou RDS, exceto se for RDS custom**

**Audit logs podem ser habilitado e mandados para o CloudWatch para ter uma retenção desses logs por mais tempo**

## Amazon RDS Proxy

**TODO: pesquisar sobre**

## Amazon ElasticCache Overview

ElasticCache é para gerenciar Redis ou Memcached

Caches são banco de dados em memória com alto desempenho e baixa latência

Ele ajuda a reduzir as cargas ao banco de dados com altas cargas de leitura, ou seja, consultas comuns a ideia é que elas sejam armazenadas em cache para que todas as vezes tenha essa carga.

Ajuda também em tornar sua aplicação stateless, sendo assim tirando essa parte de estado da sua aplicação e levando para a AWS.

**Usar o ElasticCache envolve fazer grandes alterações de código**

**Exemplo de caso de uso com Elastic Cache:**
![Como funcioan o Elastic Cache](./imagens/elastic-cache-como-funciona.png)


**Aqui um exemplo de uso guardando dados de sessão de usuário:**
![Como funciona o Elastic Cache](./imagens/elastic-cache-user-data.png)

### ElasticCache Estratégias

- É seguro armazenar dados em cache? Em geral sim, mas pode acontecer do dado estar desatualizado, porntanto podem haver inconsistências. Portanto, não é para todo grupo de dados.
- Cache é efetivo: Depende, se seus dados tem poucas mudanças, poucas chaves sim é efetivo, por outro lado, se seus dados mudam com frequência e precisam de grande espaço não é efetivo.
- Estruturas como chave e valor e agregação são boas para o armazenamento em cache

### Lazy Loading / Cache-Aside / Lazy Population

![Como funciona o padrão de lazy loading](./imagens/elastic-cache-lazy-loading.png)

### Write Through

![Como funciona o Write Through](./imagens/write-through.png)

### Cache Evictions e Time-to-live (TTL)

Cache não é ilimitado. Então você pode explicitamente deletar um item

## DNS

DNS é basicamente uma tradução amigavel para os humanos do endereço de IP da máquina.

www.google.com => 172.217.18.36

O DNS é o backbone da internet.

![URL](./imagens/dns.png)

## Amazon Route 53

É um DNS de alta disponibilidade, escalonavel e totalmente gerenciado e autorizado com meios oficiais.

Route 53 também é um Domain Register.
E é o único serviço da AWS que provê 100% de disponibilidade.

### Route 53 - Record Types

**A** - É para mapear o nome do host em um IPv4
</br>
**AAAA** - É para mapear o nome do host em um IPv6
</br>
**CNAME** - É para mapear o nome do host para o nome de um outro host
<br>
**NS** - Esse é para servidores de nomes da zona de hospedagem. (Controla como o trafego é roteado para um domínio).

### Route 53 - Hosted Zones

É um container de registros que define como o trafego vai para um dominio e subdominios.

**Public Hosted Zones** - Contém registros que especifica como a rota trafega pela internet (public domain names).

**Private Hosted Zones** - Contém registros que especifica como você roteia o trafego entre uma ou mais VPCs (private domain names).

**Usar o Route 53 não é grátis, você paga por mês por hosted zone.**

![Funcionamento da Public Hosted Zone](./imagens/public-hosted-zone.png)

![Funcionamento da Private Hosted Zone](./imagens/private-hosted-zone.png)

### Route 53 - Records TTL (Time To Live)

É uma especie de cache para que não se precise ficar chamando o DNS toda vez, assim a gente coloca um delay e consegue cachear esses dados.

![TTL Route 53](./imagens/ttl-route-53.png)

Alto TTL, 24 horas:
- Possibilidade do cliente buscar registros desatualizados
- Menos trafego no Route 53

Baixo TTL, 53 segundos:
- Vai gerar muito trafego no Route 53
- Registros ficarão desatualizados em menos tempo
- Facil para mudar registros no geral

**TTL é obrigatório para todos os registros, exceto o registro de Alias.**

### Routing Policies - Simple

Normalmente direciona o trafego para um único recurso. 
Um exemplo, o cliente vai querer ir para o endereço foo.example.com e o Amazon Route 53 vai responder direcionando para um registro A 11.22.33.44.

![Single Value](./imagens/simple-policie.png)

<br>
No caso dessa policie podemos especificar multiplos valores para o mesmo registro. Se multiplos valores forem retornados um randomico será escolhido para o cliente.

![Multiple Values](./imagens/multiple-values.png)

- Quando você ativa o Alias, você só pode especificar um recurso.
- Não pode ser associado com Health Checks

### Routing Policies - Weighted
A ideia aqui é que você pode ter uma porcentagem de solicitações que você pode controlar para ir para o seu recurso especifico.

Cada registro daremos um peso relativo:

Trafego (%) = Peso de um registro especifico / soma de todos os pesos de todos os registros

- Registros DNS devem ter o mesmo nome e tipo
- Podem ser associados com Health Checks
- Caso de uso: Load Balancing entre regiões, testando uma nova aplicação ou versão
- Você assinar o peso 0 (zero) a um registro faz para de mandar trafego para o recurso
- Se todos os registros tem o peso 0 (zero), então todos os registros vão ser retornados igualmente.

### Routing Policies - Latency-based

A ideia é que você queira redirecionar o recurso que terá a menor latência próximo a nós.
É muito útil quando a latência é sua principal preocupação para seus sites.

Latência será medida com base na rapidez com que os usuários se conectam à região de interesse mais próxima identificada para aquele registro. Por exemplo, usuário da Alemanha e a latência mais baixa for dos EUA.

Pode ser associado com Health Checks.

### Route 53 - Health Checks

HTTP Health Checks são apenas para serviços publicos.

Temos vários tipos de Health Checks, como por exemplo: 
- Health checks que monitoram um endpoint (application, server, ou algum outra serviço ou recurso da AWS)
- Health Checks que monitoram outros health checks (Calculated Health Checks)
- Health Checks que monitoram alarmes do CloudWatch 

![Health Checks Route 53](./imagens/healthCheck-route53.png)

### Routing Policies - Geolocation

É diferente da Latency-based. Essa é baseada onde o usuários está realmente localizado.
Você pode direcionar o cliente de acordo de onde vem a localização dele.

### Geoproximity Routing Policy

Permite que você direcione o trafego de seus recursos com base na localização geográfica de seus usuários e recursos.

![Geo Proximidade](./imagens/geoproximidade.png)

### Routing Policies - IP-based Routing

É baseado no endereço de IP do cliente.

### Routing Policies - Multi-Value

Usado quando quiser rotear o tráfego para vários recursos e a rota.

Route 53 retorna multiplos valores/recursos

Pode ser associado com Health Checks (Retorna apenas um valor para healthy resources)

## VPC Fundamentals

### VPC & Subnets Primer

- **VPC**: é uma numve privada que permite você faze deploy de recursos AWS nela.
- É um recurso Regional, ou seja, se tiver duas regiões você terá duas VPC's diferentes.
- Dentro das VPC's a gente tem as **Subnets**, que permitem particionar sua rede dentro da sua VPC. (Subnets e VPCs são definidas no nível de AZ)
- Uma subnet pública é acessivel pela internet.
- Uma subnet privada é aquela que não é acessivel pela internet
- Para definir o acesso à internet e entre subnets, usamos Route Tables.

![Como funciona VPC](./imagens/como-funciona-vpc.png)

### Internet Gateway & NAT Gateways

Internet Gareways ajuda a nossa VPC a a se conectar com a internet

Subnets públicas tem a rota para o internet gateway. É isso que torna uma subnet pública ou privada, essa conexão com o internet gateway.


Já para subnets privadas usamos NAT Gateways (Gerenciado pela AWS) & NAT Instances (Auto gerenciado).

![Gateways](./imagens/gateways-vpc.png)

### Network ACL & Security Groups

- **NACL** (Network ACL)
  - É um firewall que controla o tráfego ee e para as subnets.
  - Pode ter regras de Allow e Deny de trafego
  - Você atacha os NACL's à Subnet 
  - As regras só incluem endereços de IP

![NACL](./imagens/nacl.png)

- **Security Groups** (Network ACL)

Security groups é um firewall que controla o trafego de e para uma interface de rede ENI / e EC2 Instance

- Pode ter somente ALLOW rules
- Rules incluem endereços de IP e outros security groups

![VPC security groups](./imagens/vpc-security-group.png)

### VPC Peering

Esse usamos quando queremos conectar duas vpc's privadas

![VPC Peering](./imagens/vpc-peering.png)

### VPC Endpoints

Permite que você se conecte aos seviços da AWS usando uma rede privada ao inves de uma rede www publica.

- Só é utilizado com VPC
- Da mais segurança e menor latencia para acessar serviços AWS

![VPC Endpoint](./imagens/vpc-endpoint.png)

### Typical 3 tier solution architecture

![Typical 3 tier solution architecture](./imagens/typical-tier-solution-architecture.png)

## Amazon S3

- Usado para armazenamento e backup
- Disaster Recovery
- Arquivamento
- Storage cloud hibrido
- Application hosting
- Media hosting
- Data lakes & Big Data analytics
- Software delivery
- Static website

<br>

- Amazon S3 permite armazenar objetos (arquivos) em "buckets" (diretórios)
- Buckets devem ter nome globalmente únicos (entre todas regiões e todas as contas)
- Buckets são definidos no nivel de região
- S3 parece ser um serviço global, mas buckets são criados no nivel de região
- Convenção de nomes
  - Sem letras maiúsculas, sem underline
  - 3 - 63 caracteres
  - Sem IP
  - Deve começar com letra minuscula ou numero
  - Não deve começar com prefixo xn--
  - Não deve terminar com sufixo -s3alias

### Amazon S3 Objects

Objects (arquivos) tem uma key. A Key é o caminho completo: s3://my-bucket/**my_file.txt**.

A key é composta pelo prefixo + nome do objeto.

### Amazon S3 Security

**User-Based** - IAM Policies, permissão para um usuário especifico ou grupo de usuários espeficos definidos no IAM

**Resource-Based** - Bucket Policies - permite aplicar regras diretamente ao bucket direto do console e também permite cross account.

**Objects Access Control List (ACL)**

**Bucket Access Control List (ACL)**

### S3 Storage Classes

#### Amazon S3 Standrd - General Purpose

- 99.99% Disponibilidade
- Usado para dados com frequencia de uso
- Baixa lantências e alta taxa de transferência
- Ele pode sustentar duas falhas de instalação simultâneas

#### Infrequent Access

- Para dados que são menos frequentemente acessados, mas querem rapido acesso quando precisa
- Custo é menor do que o S3 Standard

#### Glacier Storage Classes

- Armazenamento de baixo custo, destinado para arquivamento e backup
- Você paga pelo armazenamento + custo de recuperação dos arquivos

#### Intelligent-Tiering

- Permite mover objetos entre camadas ema excesso com base em padrões de uso

## AWS CLI, SDK, IAM Roles e Policies

### AWS EC2 Instance Metada (IMDS)

Permite que instâncias EC2 saibam uma sobre a outra sem usar IAM Role para isso.

A url é http://196.254.169.254/latest/meta-data

### AWS SDK Overview

Se quisermos executar ações diretamente no código dos nossos aplicativos sem abrir o CLI, usamos o SDK.

Detalhe é que se não especificarmos uma região a região padrão vai se us-east-1.

## Advanced Amazon S3
![Essas são as transioções de objetos entre classes que podemos fazer](./imagens/moving-storage-s3.png)

Cada uma dessas classes é definida de acordo com o tempo de armazenamento que você pretende deixar, portando se é objeto que você não acessa frequentemente mova para Standard IA. Para objetos que você não precisa de rápido acesso, mova-os para a camada Glacier ou Glacier Deep Archive... 

Essa movimentação pode ser feita manualmente, mas também podemos automatizar usando o Lifecycle Rules.

### Amazon S3 - Lifecycle Rules

**Transition Actions** - É a configuração de objeto para a transição de uma classe de armazenamento para outra. 

Por exemplo, criar uma ação que mova os objetos da classe Standard IA após 60 dias da criação. Ou que mova para a classe Glacier após 6 meses de criação.

**Expiration Actions** - É uma ação que permite configurarmos o tempo de expiração (deleção) de objetos após um certo tempo.

Por exemplo: Podemos criar regras para que logs de acesso sejam deletados após 365 dias. Ou podemos usar para deletar versões antigas de arquivos se o versionamento estiver ativo. Ou também podemos deletar aqueles arquivos que tiveram o upload incompleto.

Essas regras também podem ser criadas para prefixos especificos. Exemplo: s3://mybucket/mp3/*. Da mesma forma que também podemos criar regras especificas para determinadas tags.

### Amazon S3 Analytics - Storage Class Analysis

Nos ajuda a decidir quando transicionar objetos para a classe certa.

Ele faz recomendações para as classes Standar e Standard IA.


Dessa forma, o bucket s3 que tem a ele ligado o s3 analytics tem um csv com várias recomendações e informações. Os reports são feitos diariamente.
![S3 Analytics](./imagens/s3-analytics.png)


### Event Notifications

A ideia aqui é que o S3 pode criar notificações de determinadas ações que acontecem dentro dele. Por exemplo, quando um objeto é criado, quando um objeto é deletado, restaurado, replicado...

Então dentre esses eventos podemos filtrar por exemplo pelo tipo jpg.

Um caso de exemplo seria gerar thumbnails de imagens que subimos no s3 e se fosse o caso ativar uma lambda, ou mandar para um sns ou um sqs...




## Conteúdos adicionais de apoio para fixação 

[Mapa mental dos conteúdos da certificação](https://www.mindmeister.com/pt/2688053989/aws-developer).