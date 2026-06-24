🚀 Minha Primeira Stack com AWS CloudFormation

Repositório criado como entrega do desafio de projeto da DIO, documentando minha experiência implementando e evoluindo uma Stack na AWS utilizando o AWS CloudFormation.

📌 Sobre o desafio

O objetivo deste laboratório foi sair da teoria e colocar a mão na massa: criar, executar e evoluir templates de Infraestrutura como Código (IaC) no CloudFormation, entendendo na prática como a AWS provisiona recursos de forma automatizada — e também como diagnosticar e corrigir erros reais durante o processo.

☁️ O que é o AWS CloudFormation?

O AWS CloudFormation é um serviço de Infraestrutura como Código (IaC) que permite descrever, em arquivos de template (YAML ou JSON), todos os recursos da AWS que você precisa — e o serviço se encarrega de criar, atualizar ou remover tudo isso de forma automatizada e repetível.

Vantagens que pude observar na prática:


Repetibilidade: o mesmo template recriou minha infraestrutura várias vezes, sem erro manual.
Versionamento: como é só um arquivo de texto, posso guardar a infraestrutura no Git, igual a código de aplicação.
Rollback automático: quando um dos meus templates falhou (veja a seção de desafios abaixo), o CloudFormation desfez tudo automaticamente, sem deixar recursos "órfãos" cobrando indevidamente.


🗂️ Estrutura do laboratório

O laboratório foi dividido em 4 templates incrementais, cada um adicionando uma nova camada de complexidade:

ArquivoRecursos criadosO que aprendi01-EC2.yamlInstância EC2 básicaA estrutura mínima de um template: Resources → tipo do recurso → Properties.02-Apache.yamlEC2 + instalação automática do Apache via UserDataO script de UserData roda como um shell script na primeira inicialização da instância, automatizando a instalação e configuração de software sem intervenção manual.03-Firewall.yamlEC2 + Apache + AWS::EC2::SecurityGroup liberando a porta 80Como o !Ref conecta recursos entre si (a instância referencia o Security Group), e por que esse recurso era necessário para o Apache ficar acessível externamente.04 (Stack UserGroup)EC2 + Bucket S3 + Grupo e Usuário IAMComo combinar recursos de tipos diferentes (computação, armazenamento e identidade) numa única Stack, e a importância da caixa de confirmação de criação de recursos IAM.

🛠️ Passo a passo executado


Acessei o Console da AWS → CloudFormation → Criar pilha.
Para cada template, fiz upload do arquivo .yaml, segui as etapas do assistente (nome da pilha → parâmetros → opções → revisão) e acompanhei a criação na aba Eventos.
Após cada Stack ficar com status CREATE_COMPLETE, validei o resultado:

Acessando a instância EC2 criada pela aba Recursos.
Testando o servidor Apache pelo navegador, usando o IP público da instância.



Ao final dos testes, excluí todas as Stacks pelo Console, para evitar custos desnecessários.


📸 Evidências

Mostrar Imagem

Página servida pelo Apache instalado via UserData, acessada pelo IP público da instância (http://<ip-publico>), confirmando que a instalação automática e o Security Group da porta 80 funcionaram corretamente.

💡 Principais aprendizados e insights


Entendi a diferença entre criar recursos manualmente no Console e via template: com IaC, o mesmo resultado é reproduzível, versionável e revisável, ao invés de depender de cliques manuais.
O UserData é uma forma simples e poderosa de automatizar a configuração inicial de uma instância (nesse caso, instalar e iniciar o Apache sem nenhuma ação manual).
Vi na prática como o Security Group funciona como o "firewall" da instância: sem liberar a porta 80 explicitamente, o Apache instalado não seria acessível pela internet.
Entendi como o !Ref cria dependências entre recursos dentro do mesmo template (ex: a instância EC2 referenciando o Security Group, ou o usuário IAM referenciando o grupo IAM).
Templates que criam recursos do tipo IAM exigem uma confirmação explícita na etapa de revisão ("Reconheço que o AWS CloudFormation pode criar recursos do IAM") — sem isso, a criação falha por falta de permissão.


🧩 Desafios encontrados

1. Erro de Free Tier com t2.micro

Ao criar a primeira Stack (01-EC2.yaml), recebi o erro:


"The specified instance type is not eligible for Free Tier..."



Investigando, descobri que a AWS alterou sua política de Free Tier a partir de 15/07/2025: contas criadas após essa data deixaram de ter o t2.micro como tipo elegível, passando a usar t3.micro como padrão gratuito. Corrigi todos os templates trocando InstanceType: t2.micro por InstanceType: t3.micro, e todas as Stacks passaram a ser criadas com sucesso.

2. Múltiplos problemas no template mais avançado (Stack UserGroup)

O quarto template, que criava EC2 + S3 + IAM, tinha vários valores de exemplo/placeholder que não funcionariam na minha conta:


BucketName com letras maiúsculas (nomes de bucket S3 não podem ter maiúsculas e precisam ser únicos globalmente) → removi a propriedade e deixei a AWS gerar um nome único automaticamente.
KeyName apontando para um par de chaves que não existia na minha conta → removido, já que não era essencial para o objetivo do laboratório.
VpcId fixo de outra conta AWS → removido, deixando o CloudFormation usar a VPC padrão da minha conta.
AMI do Ubuntu combinada com comandos apt-get no UserData, mas usando uma AMI Amazon Linux nos meus outros templates → padronizei usando a mesma AMI Amazon Linux (com yum) que já tinha validado nos templates anteriores.


Esse foi o template que mais me ensinou — entender por que cada erro acontecia foi mais valioso do que simplesmente copiar o arquivo original.

✅ Conclusão

Este desafio me ajudou a sair da teoria do CloudFormation e enxergar, na prática, como a Infraestrutura como Código torna o provisionamento de recursos na AWS mais rápido, seguro e replicável — e também como ler mensagens de erro, investigar a causa raiz e corrigir templates com confiança, em vez de só seguir um passo a passo sem entender.

🔗 Referências


Documentação oficial do AWS CloudFormation
AWS Free Tier — alterações a partir de 15/07/2025
Aulas da trilha da DIO



📍 Projeto desenvolvido como parte da trilha de estudos da Digital Innovation One (DIO).
