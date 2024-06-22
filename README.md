# Desenvolvendo-um-Microsserviço-com-Spring-Boot-Explorando-o-Azure-AKS

Desenvolver um microsserviço com Spring Boot e explorar o Azure AKS (Azure Kubernetes Service) envolve uma série de passos que abrangem desde a configuração inicial do projeto até a implantação e orquestração na nuvem. Vamos detalhar cada etapa para garantir uma compreensão completa do processo.

### Passos Iniciais

1. **Configuração do Ambiente de Desenvolvimento**
   - Instale as ferramentas necessárias: JDK (Java Development Kit), Maven (ou Gradle), Docker, Azure CLI e o Azure DevOps para gerenciamento e integração contínua.

2. **Criação do Projeto Spring Boot**

   Primeiramente, vamos criar um projeto Spring Boot modularizado, o que é uma boa prática para microsserviços, especialmente quando pensamos em escalabilidade e manutenção.

   ```bash
   mkdir meu-microsservico
   cd meu-microsservico
   ```

   Utilize o Spring Initializr (https://start.spring.io/) ou o Maven para configurar o projeto com as dependências necessárias. Aqui está um exemplo mínimo de um `pom.xml` para o Maven:

   ```xml
   <project xmlns="http://maven.apache.org/POM/4.0.0" 
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <groupId>com.example</groupId>
       <artifactId>meu-microsservico</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.6.3</version> <!-- Versão atual do Spring Boot -->
           <relativePath/>
       </parent>
       <dependencies>
           <!-- Adicione as dependências do Spring Boot que você precisar -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <!-- Outras dependências necessárias -->
       </dependencies>
       <properties>
           <java.version>11</java.version>
       </properties>
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

3. **Estruturação do Projeto**

   É importante estruturar o projeto de maneira modular para separar responsabilidades e facilitar a escalabilidade. Por exemplo:

   ```
   meu-microsservico/
   ├── meu-microsservico-core (regras de negócio)
   ├── meu-microsservico-api (controladores REST)
   ├── meu-microsservico-service (serviços)
   ├── meu-microsservico-persistence (persistência de dados)
   └── meu-microsservico-application (classe principal do Spring Boot)
   ```

   Isso ajuda a organizar o código e a definir claramente onde cada parte do microsserviço reside.

### Integração com Azure AKS

1. **Configuração do Azure AKS**

   - Crie um cluster AKS na Azure utilizando a Azure CLI ou o portal web.
   - Instale o `kubectl` e configure-o para acessar o seu cluster AKS.

2. **Dockerização do Microsserviço**

   Para implantar no AKS, é essencial dockerizar o microsserviço. Crie um Dockerfile na raiz do projeto:

   ```dockerfile
   # Dockerfile
   FROM adoptopenjdk/openjdk11:alpine-jre
   ARG JAR_FILE=target/*.jar
   COPY ${JAR_FILE} app.jar
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

   Execute os seguintes comandos para construir e testar a imagem Docker localmente:

   ```bash
   mvn clean package
   docker build -t meu-microsservico .
   docker run -p 8080:8080 meu-microsservico
   ```

3. **Implantação no AKS via Azure DevOps**

   - Configure um pipeline no Azure DevOps para automatizar a construção da imagem Docker e a implantação no AKS.
   - Use um arquivo YAML no Azure Pipelines para definir o pipeline, integrando com o repositório Git do projeto.

   Exemplo de um pipeline básico no `azure-pipelines.yml`:

   ```yaml
   trigger:
     branches:
       include:
         - main

   pool:
     vmImage: 'ubuntu-latest'

   steps:
     - task: Maven@3
       inputs:
         mavenPomFile: 'pom.xml'
         goals: 'package'

     - task: Docker@2
       inputs:
         containerRegistry: 'nomeDoSeuContainerRegistry'
         repository: 'meu-microsservico'
         command: 'buildAndPush'
         Dockerfile: '**/Dockerfile'
         tags: 'latest'

     - task: Kubernetes@1
       inputs:
         connectionType: 'Azure Resource Manager'
         azureSubscriptionEndpoint: 'nomeDoSeuAzureServiceConnection'
         namespace: 'default'
         command: 'apply'
         arguments: '-f deployment.yaml'
   ```

   - O arquivo `deployment.yaml` define como o microsserviço deve ser implantado no AKS, especificando pods, serviços, etc.

### Considerações Finais

Desenvolver um microsserviço com Spring Boot e implantá-lo no Azure AKS envolve uma combinação de boas práticas de desenvolvimento, ferramentas de automação e uma compreensão clara das tecnologias envolvidas. Certifique-se de ajustar os exemplos e comandos de acordo com as especificidades do seu projeto e ambiente.
