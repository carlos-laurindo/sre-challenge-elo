
## Bem-vindo

Estamos muito felizes por você estar pensando em se juntar a nós! Esse é um teste feito para conhecer um pouco mais de cada candidato. Não se trata de um teste objetivo, capaz de gerar uma nota ou uma taxa de acerto, mas sim de um estudo de caso com o propósito de conhecer os conhecimentos, experiências e modo de trabalhar de um candidato. Não experamos que tudo seja feito perfeitamente, pois valorizamos o seu tempo. Sinta-se livre para desenvolver sua solução para o problema proposto.

Este desafio está dividido em 4 partes:

1. Implementação
2. Depuração
3. Melhorias
4. Perguntas

## Requisitos para o desafio

- Github Account

Se você encontrar possíveis melhorias a serem feitas neste desafio, fique a vontade para descreve-lás.

## Sobre o Desafio:

A ELO executa a maior parte de sua infraestrutura em Kubernetes. É um monte de microserviços conversando entre si e realizando diversas tarefas.

Nesse repositório, fornecemos a você:

- 'sre-challenge-app/': Uma aplicação com CRUD que armazena dados de funcionários em um Banco de dados MySQL 8.

### Configure o ambiente do desafio

1. Instale qualquer cluster K8s local (ex: Minikube) em sua máquina e documente sua configuração, para que possamos executar sua solução.

### Parte 1 - Configure os aplicativos

VirtualBox:

Baixe e instale a versão mais recente do VirtualBox.

Durante a instalação, ative a opção para adicionar o VirtualBox ao PATH (opcional, mas recomendado).
Docker Desktop (para gerenciar imagens):

Baixe e instale o Docker Desktop.

Certifique-se de ativar a integração com WSL2 durante a instalação.
Após instalar, inicie o Docker Desktop e verifique se ele está funcionando corretamente.

Minikube:

Baixe o binário do Minikube em Minikube Releases.
Renomeie o arquivo baixado para minikube.exe.
Copie o arquivo para uma pasta no PATH do Windows (ex.: C:\Windows\System32).
kubectl (CLI para gerenciar clusters Kubernetes):

Baixe o kubectl em kubectl releases.
Copie o arquivo para uma pasta no PATH, assim como o Minikube.

Configuração do PATH:

Certifique-se de que as pastas onde estão os executáveis (minikube.exe, kubectl.exe, VBoxManage.exe) estão no PATH do sistema.

2. Configurar e iniciar o Minikube

2.1. Iniciar o cluster Minikube:
Abra o Prompt de Comando ou PowerShell como Administrador.

Execute o comando para iniciar o Minikube com o VirtualBox como driver:

minikube start --driver=virtualbox

2.2. Verificar o status:
Após o comando, verifique se o cluster está funcionando:

kubectl get nodes
Se tudo estiver correto, verá um nó listado com o status Ready.

Gostariamos que essa aplicação sre-challenge-app e seu banco de dados fossem executados em um cluster K8s.

Requisitos

1. A aplicação deve ser acessível de fora do cluster.
   ![image](https://github.com/user-attachments/assets/de178dee-18ea-4fc9-8d14-399895334cec)

3. Manifestos de implantação do kubernetes para executar com limitação de requests e usando HPA.

   [app-deployment.yaml](app-deployment.yaml)

### Parte 2 - Corrigir o problema

A aplicação tem um problema. Encontre e corrija! Você saberá que corrigiu o problema quando o estado dos pods no namespaces for semelhante a este:

```
NAME                                 READY   STATUS    RESTARTS   AGE
db-5877fd4d4d-qmngl                  1/1     Running   0          6m50s
sre-challenge-app-59fd5ffc57-lm2xs   1/1     Running   0          7s
```

Requisitos

Escreva aqui sobre o problema, a solução, como você a encontrou e qualquer outra coisa que queira compartilhar sobre ela.
![image](https://github.com/user-attachments/assets/cc1a248d-d1a4-4586-b71f-3abe0181977d)

No arquivo de configuração application.properties:
spring.datasource.url=jdbc:mysql://test:3306/emp?allowPublicKeyRetrieval=true&useSSL=false
A configuração acima indica que a aplicação espera que o banco de dados esteja acessível no host test, na porta 3306, com o banco de dados chamado emp.
No entanto, ajustei essa configuração para utilizar o Service do Kubernetes como host, nomeado como db. Com isso, a configuração ficou assim:
spring.datasource.url=jdbc:mysql://db:3306/emp?allowPublicKeyRetrieval=true&useSSL=false
Essa alteração permitiu que a aplicação fosse iniciada corretamente, pois agora ela consegue se conectar ao banco de dados por meio do serviço no Kubernetes.

### Parte 3 - Melhores práticas

Essa aplicação tem uma falha de segurança e gostariamos que as credenciais do MYSQL fossem armazenadas em uma secret do Kubernetes.

[mysql-secret.yaml](mysql-secret.yaml)
   
Requisitos
1. Manifesto do kubernetes usando a API de secret com as credenciais do Banco para implantação.
   [mysql-deployment.yaml](mysql-deployment.yaml)
   
3. Manifesto do kunernetes da aplicação com as informações da secret criada anteriormente.
    [app-deployment.yaml](app-deployment.yaml)
   
2. Configuração do código da aplicação utilizando uma variável que foi referenciada no secrets do K8s (Application Properties do Java)
   [application.propertiesl](application.properties)


### Parte 4 - Perguntas

Sinta-se à vontade para expressar seus pensamentos e compartilhar suas experiências com exemplos do mundo real com os quais você trabalhou no passado.

Requisitos
O que você faria para melhorar essa configuração e torná-la “pronta para produção”?
Existem 2 microsserviços mantidos por 2 equipes diferentes. Cada equipe deve ter acesso apenas ao seu serviço dentro do cluster. Como você abordaria isso?
Como você evitaria que outros serviços em execução no cluster se comunicassem com o sre-challenge-app?

Segurança:

Use Secrets para senhas: Já configurado para usar Secrets. Validar se todas as variáveis sensíveis estão armazenadas em Secrets.
Habilitar HTTPS: Utilize TLS/SSL para criptografar as comunicações entre os microsserviços e com o banco de dados.
RBAC: Configure RBAC para garantir que apenas usuários e pods autorizados possam acessar recursos sensíveis.

Escalabilidade e Desempenho:
Defina limites de recursos para garantir que os pods não consumam recursos excessivos.
Habilite Horizontal Pod Autoscaler para escalar automaticamente os microsserviços conforme necessário.
Liveness e Readiness probes para garantir que a aplicação seja monitorada e reiniciada quando necessário.

Monitoramento e Logs:
Implemente monitoramento com ferramentas como Prometheus e Grafana.
Configure logs estruturados e auditoria.
Controle de Acesso entre Microsserviços:

Namespaces:
Crie Namespaces para isolar os serviços de cada equipe. Por exemplo, equipe-1 e equipe-2

RBAC:
Use Roles e RoleBindings dentro dos namespaces para garantir que cada equipe tenha acesso apenas ao seu serviço.
Evitar Comunicação Indesejada com o sre-challenge-app:

Network Policies:
Defina Network Policies que restrinjam a comunicação entre namespaces e serviços, permitindo o tráfego apenas do namespace autorizado.
Service tipo ClusterIP:


## O que é importante para nós?

É claro que esperamos que a solução funcione, mas também queremos saber como você trabalha e o que é importante para você como engenheiro. Portanto, fique à vontade para criar novos arquivos, refatorar, renomear, ...

Idealmente, gostaríamos de ver sua progressão através de commits, verbosidade em suas respostas e todos os requisitos atendidos. Não se esqueça de atualizar o README.md para explicar seu processo de pensamento.

## Entrega do desafio: Carlos Alberto Laurindo Junior // CCE ELO 
# Desafio repassado ao CCE para treino, espero ter chegado próximo de um bom resultado, mas o aprendizado que estou tendo durante o desafio é incrível, espero realizar outros.
![image](https://github.com/user-attachments/assets/83fc8372-bef7-4d12-ac9a-6050f211574f)

Ao terminar o desafio, convide o 'ELO-SRE' para contribuir com o seu repositório de desafios para que possamos fazer a avaliação. Boa Sorte

<p align="center">
  <img src="ca.jpg" alt="Challange accepted" />
</p>

