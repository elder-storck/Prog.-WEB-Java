# Lab Web 04 - Autorização de acesso

Neste roteiro, usaremos como base o que foi desenvolvido nos **Lab Web** anteriores, adicionando controle de autorização de acesso, ou seja, permitir o uso de determinadas funcionalidades apenas aos usuários que possuam o papel correspondente.

Sugere-se ainda continuar aproveitando estes roteiros de laboratório para desenvolver parte do trabalho prático da disciplina. Desta forma, os roteiros podem ser feitos em times, compartilhando o código no repositório criado no GitHub para o projeto e dividindo o trabalho entre os diferentes membros do time.


## Análise da implementação da autenticação

1. Os mecanismos básicos de segurança são a autenticação (verificar se o(a) usuário(a) é quem ele(a) diz ser) e a autorização (verificar se o(a) usuário(a) tem acesso a um determinado recurso). Reveja no **Lab Web 01**, a partir do passo 9, como foi implementada a autenticação utilizando o Jakarta Security:

      ```java
      @CustomFormAuthenticationMechanismDefinition(
      loginToContinue = @LoginToContinue(loginPage = "/login.xhtml", useForwardToLogin = false,
            errorPage = ""))
      @DatabaseIdentityStoreDefinition(dataSourceLookup = "java:app/datasources/oldenburg",
      callerQuery = "select password from User where email = ?",
      groupsQuery = "select role from User where email = ?")
      @FacesConfig
      @ApplicationScoped
      public class AppConfig {
      }
      ```

      > As anotações na classe `AppConfig` acima indicam que os usuários serão identificados a partir dos atributos `email` e `password` da classe `User` e que o seu papel no sistema será identificado pelo atributo `role` nesta mesma classe.

2. Verifique como o atributo `role` em `User` foi mapeado ao banco de dados, pois o que for retornado na consulta `groupsQuery` acima é o que deve ser usado na configuração de autorização. Por padrão, tipos enumerados são armazenados como números inteiros (o "índice" daquele valor no conjunto de valores do tipo), porém para fins de autorização é melhor usar o "nome" do membro do tipo enumerado (ex: usar ADMIN ao invés de 0, PROFESSOR ao invés de 1 e assim por diante). Uma anotação do JPA faz com que o nome seja armazenado no banco de dados ao invés do índice:

      ```java
      @Enumerated(EnumType.STRING)
      private Role role;

      ```


## Implementação da autorização

3. Para cada funcionalidade já desenvolvida, avalie qual papel um(a) usuário(a) teria que ter para poder acessá-la. No exemplo do Oldenburg, os cadastros de usuários e workshops seriam acessíveis apenas a administradores, enquanto o envio de artigos apenas a estudantes;

4. Adicione às classes de serviço a anotação `@RolesAllowed`, que recebe como argumento uma string ou um vetor de strings com os nomes dos papeis que podem ter acesso ao elemento que ela está anotando. Ela pode anotar um método específico ou uma classe inteira (e, portanto, se aplica a todos os métodos da classe). No exemplo do Oldenburg, foram modificadas as seguintes classes:

      ```java
      @Stateless
      @RolesAllowed("ADMIN")
      public class ManageUsersServiceBean extends CrudServiceImpl<User> implements ManageUsersService {
            // ...
      }
      ```

      ```java
      @Stateless
      @RolesAllowed("ADMIN")
      public class ManageWorkshopsServiceBean extends CrudServiceImpl<Workshop> implements ManageWorkshopsService {
            // ...
      }
      ```

      ```java
      @RolesAllowed("STUDENT")
      public class SubmitPaperServiceBean implements SubmitPaperService {
            // ...
      }
      ```

5. Faça a implantação e teste a autorização. Por exemplo, entre com um usuário ADMIN e verifique se tem acesso aos serviços marcados com este papel e não tem acesso aos marcados com papeis diferentes. Entre com outro papel e tente acessar os serviços que são apenas do ADMIN. Ao tentar acessar algo que não é permitido, a exceção `jakarta.ejb.EJBAccessException` é lançada, levando a um ERRO 500 na página Web;

6. Caso alguma de suas classes seja um serviço de CRUD do JButler (i.e., subclasse de `CrudServiceImpl`), como os exemplos de `ManageUsersServiceBean` e `ManageWorkshopsServiceBean` acima, repare que você ainda consegue abrir a funcionalidade e ver os elementos cadastrados, mas talvez não consiga criar ou editar elementos. Para corrigir essa situação e receber o erro assim que tentar abrir a funcionalidade sem o papel adequado, sobrescreva o método `authorize()`, como no exemplo abaixo:

      ```java
      @Stateless
      @RolesAllowed("ADMIN")
      public class ManageUsersServiceBean extends CrudServiceImpl<User> implements ManageUsersService {
            // ...

            @Override
            public void authorize() { }
      }
      ```

      > A anotação `@RolesAllowed`, quando usada no nível da classe, se aplica aos métodos implementados naquela classe, porém não aos métodos herdados de superclasses. Por esse motivo que, mesmo após a inclusão da anotação de autorização, o acesso ainda é concedido a parte das funcionalidades do CRUD, a saber, as partes que não foram sobrescritas pela classe (ex.: se você tiver sobrescrito `validateCreate()`, ao tentar criar um novo objeto o erro de acesso não autorizado irá se apresentar). O JButler incluiu este método `authorize()` em `CrudServiceImpl` vazio e ele é chamado por todas as operações de CRUD pelo controlador, de modo que, ao sobrescrevê-lo (também vazio) como no exemplo acima, nenhuma operação de CRUD (nem mesmo a listagem) vai funcionar, pois este método agora está sob o controle de autorização de `@RolesAllowed`. Importante observar que esta é uma característica do JButler, não sendo necessário fazer isso nos serviços que implementam seus próprios métodos de lógica de negócio como, por exemplo, `SubmitPaperServiceBean`.

7. Teste novamente a autorização e certifique-se que usuários que não possuam acesso a determinadas funcionalidades de fato estejam recebendo erros ao tentar acessá-las.


## Melhorias na interface com o usuário

8. Idealmente, não deveríamos oferecer uma opção no menu de uma funcionalidade que o usuário não tem acesso. Portanto, faça com que o seu _template_ que inclui os menus nas páginas verifique junto ao controlador que fez o login qual o papel do usuário que está atualmente autenticado. No exemplo do Oldenburg, em `src/main/webapp/WEB-INF/templates/template.xhtml`:

      ```html
      <!-- ... -->

      <ui:define name="menu">
      <ul class="sidebar-menu">
            <!-- Menu entries. -->
            <li><p:link outcome="/index">
                  <i class="fa fa-home"></i>
                  <span><h:outputText value="#{msgs['menu.home']}" /></span>
            </p:link></li>
            <h:panelGroup rendered="#{sessionController.admin}">
                  <li><p:link outcome="/core/manageWorkshops/index">
                        <i class="fa fa-calendar-days"></i>
                        <span><h:outputText value="#{msgsCore['menu.core.manageWorkshops']}" /></span>
                  </p:link></li>
                  <li><p:link outcome="/core/manageUsers/index">
                        <i class="fa fa-users"></i>
                        <span><h:outputText value="#{msgsCore['menu.core.manageUsers']}" /></span>
                  </p:link></li>
            </h:panelGroup>
            <h:panelGroup rendered="#{sessionController.student}">
                  <li><p:link outcome="/core/submitPaper/index">
                        <i class="fa fa-paper-plane"></i>
                        <span><h:outputText value="#{msgsCore['menu.core.submitPaper']}" /></span>
                  </p:link></li>
            </h:panelGroup>
      </ul>
      </ui:define>

      <!-- ... -->
      ```

      > A _tag_ `<h:panelGroup rendered="#{sessionController.admin}">` chama o método `isAdmin()` (ou `getAdmin()`) no controlador `SessionController` e exibe seu conteúdo apenas se o método retornar `true`.

9. Teste o sistema e veja se cada usuário vê apenas as funcionalidades para as quais tem acesso.

As demais funcionalidades do sistema que você tem desenvolvido para o trabalho prático da disciplina já podem ser desenvolvidas levando em conta as questões de autorização trazidas por este roteiro.
