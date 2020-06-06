## Maven

Um dos principais problemas desenvolvendo sistemas é como fazer com que toda a equipe construa o artefato final da mesma maneira com as bibliotecas e configurações corretas. Existem várias maneiras de tratar esse tipo de problema. Uma das maneiras mais comuns é usar alguma ferramenta que entenda alguma linguagem de script e copie as dependências e configurações para os lugares corretos e gere um pacote com a aplicação. 

> Um **artefacto** ou **artefato** é um dos vários tipos de subprodutos concretos produzido durante o desenvolvimento de software. Alguns artefatos (por exemplo, casos de uso, diagramas de classes, requisitos e documentos de projeto) ajudam a descrever a função, arquitetura e o design do software. Outros artefatos estão relacionados com o próprio processo de desenvolvimento - tais como planos de projetos, processos de negócios e avaliações de risco. Podem ser manuais, arquivos executáveis, módulos, também podendo ser classes e bibliotecas de códigos, arquivos de configuração, etc.

Uma ferramenta comum muito usada no mundo Java é o **Ant**. Com o Ant é possível escrever scripts de ‘**build**‘ para copiar bibliotecas de lugares específicos, arquivos de configurações e qualquer outro tipo de recurso para o artefato da aplicação. Com isso garantimos que qualquer desenvolvedor consiga gerar o artefato da aplicação localmente, sabendo que este artefato gerado é o mesmo que o resto da equipe irá usar e que possivelmente será usado em produção. Porém existem alguns problemas com essa abordagem. O primeiro problema que vem a cabeça é, onde guardamos as bibliotecas externas ao nosso projeto para que o Ant as encontre? A solução mais comum é guardá-las no repositório junto com o código fonte, assim todos os desenvolvedores compartilharão as mesmas dependências e os caminhos das pastas são mantidos.

> ***Build***, no contexto do desenvolvimento de software, é uma versão "*compilada*" de um software ou parte dele que contém um conjunto de recursos que poderão integrar o produto final.

Esse tipo de organização é muito adotada e funciona muito bem para projetos pequenos e com equipes bem pequenas. Quando começamos a prolongar um pouco a vida do projeto percebemos que o *overhead* de manutenção do mesmo começa a interferir no andamento do desenvolvimento. Isso porque o número de dependências começa a aumentar e com o tempo vão saindo versões novas de algumas dependências antigas(podemos entender dependências como sendo bibliotecas). Se todas as dependêcias estão na mesma pasta, fica muito difícil fazer o **controle de versão** e de **transitividade** das mesmas. Agora pense em um caso mais complicado: decidimos criar um novo módulo para o projeto. Nesse caso, todas as dependências que este novo módulo necessita e estão no módulo já existente, devem ser duplicadas no repositório. Deve-se fazer isso pois já que estamos falando de módulo, devemos ser capazes de **construir** o tal módulo independente da existência dos outros. Ou seja, duplicamos também o esforço para manter a organização das dependências.

Essa é uma das motivações pricipais pela qual a apache criou o **Maven**.

O Maven é uma ferramenta de integração de projetos. É responsável por gerenciar dependências, controlar versão de artefatos, gerar relatórios de produtividade, garantir execução de testes, manter nível de qualidade do código dentre outras. Muitas pessoas confundem o maven como sendo uma alternativa para o Ant, ou para o **Ivy** o que não é verdade. O Maven inclusive disponibiliza a funcionalidade de rodar arquivos do Ant durante o build.

Voltando ao nosso problema citado anteriormente, com o Maven consiguimos isolar as bibliotecas usadas no projeto em um ‘**repositório**‘ compartilhado pela equipe, ou por toda internet no caso do repositório central do Maven. Dessa forma não nos preocupamos com duplicidade de dependências entre módulos do projeto e nem em disponibilidade das mesmas no repositório de código. Quanto a versão das dependências, estas ficam centralizadas em arquivos de configuração dos projetos de forma explícita e hierarquisada pelos módulos (POM). Com isso o Maven consegue se encarregar de fazer as devidas substituições de bibliotecas e identificar possíveis falhas no grafo de dependências.

O que torna o Maven muito poderoso é a facilidade que ele fornece para se trabalhar com vários módulos de um mesmo sistema e sua extensibilidade para novas funcionalidades com o uso de ‘**plugins**‘. Existem plugins de geração de código, de integração com plataformas de teste e inclusive suporte a IDEs como Eclipse e NetBeans. Isso torna o projeto muito mais flexível dentro da equipe, pois cada desenvolvedor pode escolher a IDE com que vai trabalhar sem se preocupar em atrapalhar o resto da equipe. Um outro plugin que será visto no post sobre Flex + Maven, é o **flexmojos**. Esse plugin é responável por integrar o Flex aos projetos do Maven de maneira transparente, executando o build da mesma forma que nos projetos Java. O plugin também disponibiliza a funcionalidade de preparar o projeto Flex para ser importado no FlexBuilder.

Vamos começar a entender melhor o Maven estudando seu funcionamento.

## Fases do build

Algo interessante sobre este post é que o que será abordado é sempre referente a ‘um pouco’, ou ‘um pouco melhor’. Isso porque o maven é uma ferramenta bastante complexa dependendo do nível de customização e da necessidade que se tenha. A primeira parte do post cobre as fases executadas durante o build.

Uma fase nada mais é do que um estágio onde são executadas algumas regras sobre o projeto e se obtem algum resultado no final. Por exemplo, a fase de testes roda os testes da aplicação e obtém um ‘OK’ ou um ‘FAIL’, no segundo caso o build é interrompido. Tais fases são executadas dentro do ‘**lifecycle**‘ do build. O ‘**lifecycle**‘ é composto de ‘**goals**‘ e são estes:

- Validate
- Compile
- Test
- Package
- Verify
- Install
- Deploy

Se você está lembrado do post anterior, usamos o comando do maven com os goals ‘**clean**‘ ‘**install**‘. Note que o goal ‘**clean**‘ não está presente no lifecycle do build, por isso tivemos que invocá-lo na linha de comando além do goal ‘**install**‘. O ‘**clean**‘ apenas apaga a pasta **target**, que como visto no post anterior contém os ‘**.class**‘ e os artefatos gerados pelo build. Os goals do lifecycle são executados nessa ordem mostrada acima. Um pouco sobre cada goal:

- **validate**: valida que os ‘**poms**‘ dos projetos involvidos estão corretamente escritos e que todas as informações necessárias para o build estão presentes;
- **compile**: compila todos os códigos do projeto, inclusive os códigos das classes de teste;
- **test**: roda os testes que estão em ‘**src/test/java**‘, e certifica-se que todos estão passando, caso contrário o build é interrompido;
- **package**: usa o código compilado e testado que está em ‘**src/main/java**‘ e cria um arquivo reusável, por exemplo jar;
- **integration-test**: nesta fase serão rodados os testes que necessitam do jar do projeto ‘**deploiado**‘ para serem rodados;
- **verify**: roda as verificações necessárias para se certificar que os pacotes gerados estão corretos e passam nos critérios de qualidade;
- **install**: copia o arquivo gerado para o repositório local para que esteja disponível localmente para outros projetos;
- **deploy**: copia o arquivo gerado para um repositório na rede ou remoto, para que esteja disponível para outros desenvolvedores.

Ainda lembrando o exemplo anterior, em um dos comandos executados possuia um goal que não está no lifecycle e que não foi citado: ‘**eclipse:eclipse**‘. Neste caso não se trata de um goal, mas sim da execução de um ‘**plugin**‘. ‘**Plugins**‘ no maven alteram o lifecycle adicionando novos goals em determinadas fases. No caso do ‘**eclipse:eclipse**‘, estamos invocando o método ‘**eclipse**‘ no plugin ‘**eclipse**‘, para isso usamos os ‘**:**‘. Esse plugin é responsável por criar os arquivos que o Eclipse entende e dessa forma consiga importar o projeto. Não está no objetivo deste post tratar de plugins, por isso se você se interessar pelo assunto existe uma boa documentação no site do próprio maven. Plugins do maven são chamados de ‘**Mojos**‘. Trataremos de ‘**plugins**‘ em um post futuro.

## POM

Project Object Model – se trata do ponto de partida para o maven executar seu lifecycle. Nada mais é do que um arquivo XML que descreve propriedades e características do projeto, como por exemplo a versão do compilador que será usada (exemplo no post anterior):

```

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>br.com.dclick</groupId>
    <artifactId>aprendendo-maven</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>aprendendo-maven</name>
    <url>http://maven.apache.org</url>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <compilerVersion>1.6</compilerVersion>
                    <source>1.6</source>
                    <target>1.6</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Repare que todas as propriedades que foram escolhidas no momento em que geramos o projeto estão definidas aqui, por isso se for preciso editar alguma delas, podemos fazer direto aqui no XML. Lembrando que para as modificações surtirem efeito, temos que executar ‘**$ mvn install**‘ e ‘**$ mvn eclipse:eclipse**‘ para refletir os resultados no Eclipse.
Além de guardar as informações mínimas necessárias para se executar o build, no pom definimos também os plugins que serão utilizados, os repositórios de artefatos que serão buscados, os relatórios que serão gerados, o pom que servirá como ‘**parent**‘, as dependências do projeto, dentre outras coisas.

Nesse post trataremos das dependências, as outras funcionalidades serão abordadas em outros posts.

Para definir um pom que servirá como ‘**parent**‘ de seu projeto, basta adicionar no pom de seu projeto o ‘**groupId**‘, ‘**artifactId**‘ e ‘**version**‘ do parent:

```
<parent>
    <groupId>br.com.dclick.framework</groupId>
    <artifactId>tec-versions</artifactId>
    <version>1.0.4-SNAPSHOT</version>
</parent>
```

Repare que está definida uma tag que ainda não foi mencionada: ‘**scope**‘. ‘**Scope**‘ explicita o escopo em que o artefato definido será usado, nesse caso estamos dizendo para o maven adicionar a dependência do JUnit apenas no momento em que os testes forem executados, garantindo assim que a dependência não seja adicionada no artefato que será gerado. Alguns outros escopos:

- **compile**: escopo padrão do maven para o momento em que o código é compilado e vai junto com o artefato;
- **provided**: adicionado no momento da compilação, mas não vai junto com o artefato. Dessa forma você está dizendo que a dependência virá de maneira transitiva de alguma outra dependência;
- **runtime**: não inclui no artefato, pois estará disponível em tempo de execução;
- **test**: inclui apenas no escopo de testes;
- **system**: não inclui no artefato, pois estará disponível no ambiente;
- **import**: incluirá TODAS as dependências do ‘**depencyManagement**‘ que está definido no pom ‘parent’.

Não falamos ainda de ‘**depencyManagement**‘, mas será tratado mais a frente.

Toda dependência no maven é transitiva, ou seja, se alguma dependência precisa de uma outra não especificada no seu projeto, o seu projeto passará a depender dela também, e será incluida no artefato gerado. A única exigência do maven é que as dependências não sejam cíclicas, e não existe restrição quanto ao número de níveis de dependências.

## Dependency Management

O dependency management é responsável por controlar as versões das dependências definidas no projeto. Por exemplo podemos definir a dependência do JUnit no dependency management, e apenas referenciar a dependência na tag depencies do projeto:

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.8.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

Uma boa prática é definir todas as versões das dependências em um dependency management do pom parent do seu projeto. Dessa forma todo projeto novo que surgir, basta herdar desse parent que as versões já estarão compartilhadas e continuarão centralizadas em um único projeto. Mesmo com a dependência definida no management, os projetos ainda podem definir suas próprias versões de artefatos e o maven dará prioridade para os mais próximos do projeto.

Nos próximos posts veremos como usar alguns plugins úteis para o ambiente de desenvolvimento. Também falaremos de outras funcionalidades como por exemplo geração de relatórios, ‘**release**‘ de projetos, entre outras.
