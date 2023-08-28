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
