POM
O POM é um dos arquivos mais importantes em um projeto Maven, ele descreve uma série de configurações que o projeto terá e quais repositórios e dependências seu projeto irá precisar. No cabeçalho de um POM temos algumas tags básicas que definem qual versão do modelo de POM utilizado. O seu GroupId que seria algo como o prefixo da estrutura de pacotes do projeto. O ArtifactId que define qual é o nome o artefato final .war ou .jar terá quando empacotado. Version define a versão do projeto que irá complementar o nome do artefato. A tag Packaging por sua vez define qual tipo de empacotamento o projeto terá após o processo de build, no nosso caso será um .war. E a tag Name define o nome do projeto.

<modelVersion>4.0.0</modelVersion>
<groupId>br.com.semeru</groupId>
<artifactId>semeru_jsf_maven</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>war</packaging>
<name>semeru_jsf_maven</name>
A tag Properties possibilita definir por exemplo a versão do Spring, ou do JSF adotada para o projeto. Se uma dependência for definida como no trecho de código abaixo ao mudarmos a versão do Spring, por exemplo, para 3.1 todas as dependências serão baixadas para a versão 3.1. Podemos definir o contêiner web no qual será feito o deploy da aplicação (Tomcat) e também o tipo de codificação utilizada pelo projeto, no nosso caso o UTF8 .

<properties>
    <spring.version>3.0.5.RELEASE</spring.version>
    <themes.version>1.0.8</themes.version>
    <jsf.version>2.1.7</jsf.version>
    <jstl.version>1.2</jstl.version>
    <netbeans.hint.deploy.server>Tomcat</netbeans.hint.deploy.server>   
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
A tag Repositories define a lista de repositórios que serão acessados pelo Maven para baixar nossas dependencias. Muitas vezes é necessário colocar um repositório prioritário no inicio do POM para que o Maven inicie a busca pelas dependências a partir dele.

<repositories>
     
    <!-- PRIMEFACES REPOSITORY -->
    <repository>
        <id>prime-repo</id>
        <name>PrimeFaces Maven Repository</name>
        <url>http://repository.primefaces.org</url>
        <layout>default</layout>
    </repository>        
             
    <!-- FACELETS TAGLIBRARIES REPOSITORY -->
     
    <repository>
        <id>org.springframework.security.taglibs.facelets</id>
        <url>http://spring-security-facelets-taglib.googlecode.com/svn/repo/</url>
    </repository>
 
</repositories>
A tag dependencies define quais serão as dependências utilizadas no projeto no trecho de código abaixo declaramos algumas das dependências necessárias para se trabalhar com JavaServer Faces.

<dependencies>
 
    <!-- || DEPENDÊNCIAS DO JAVA SERVER FACES || -->                       
    <!-- ############## JSF-API ################ -->
    <dependency>
        <groupId>com.sun.faces</groupId>
        <artifactId>jsf-api</artifactId>
        <version>${jsf.version}</version>
        <scope>compile</scope>
    </dependency>
 
    <!-- ############## JSF-IMPL ############### -->
    <dependency>
        <groupId>com.sun.faces</groupId>
        <artifactId>jsf-impl</artifactId>
        <version>${jsf.version}</version>
    </dependency>
             
    <!-- ################ JSTL ################# -->
    <dependency>  
        <groupId>javax.servlet</groupId>  
        <artifactId>jstl</artifactId>  
        <version>${jstl.version}</version>  
    </dependency>  
</dependencies>
Muitas vezes algumas dependências precisam ser excluídas. Isso acontece por vários motivos primeiro pode ser que o projeto já utilize uma versão diferente da mesma dependência. Um segundo motivo pode ser o fato da dependência em questão gerar conflitos com outras utilizadas no projeto. A sintaxe para remover uma dependência é como o trecho de código abaixo. Nele estamos declarando a dependência do DOM4J mas estamos dizendo ao Maven para não baixar a dependência da XML-APIS. Se tirarmos essa exclusão o Maven vai analizar o POM da DOM4J e verificará que ela possui uma dependência da XML-APIS e assim entenderá que o projeto necessita dela e irá baixá-la para nosso .m2.

<!-- ############### DOM4J ################# -->
<dependency>
    <artifactId>dom4j</artifactId>
    <groupId>dom4j</groupId>
    <type>jar</type>
    <version>1.6.1</version>
    <exclusions>
        <exclusion>
            <artifactId>xml-apis</artifactId>
            <groupId>xml-apis</groupId>
        </exclusion>
    </exclusions>
</dependency>
O Maven utiliza um diretório local para baixar os artefatos da internet. O diretório padrão fica dentro pasta do usuário, na pasta .m2
