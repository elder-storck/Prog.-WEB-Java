# Lab Web 02 - Funcionalidades CRUD

Neste roteiro, usaremos como base o que foi desenvolvido no **Lab Web 01 - Base da Aplicação**, adicionando funcionalidades de cadastro, ou CRUD (_Create, Retrieve, Update, Delete_). O roteiro em si é pequeno, pois nos referiremos às instruções do [tutorial de CRUD do JButler](https://gitlab.labes.inf.ufes.br/labes/jbutler/-/wikis/tutorials/JButler-CRUD-Tutorial). O roteiro pode ser feito com sua IDE favorita.

Sugere-se continuar aproveitando estes roteiros de laboratório para desenvolver parte do trabalho prático da disciplina. Desta forma, os roteiros podem ser feitos em times, compartilhando o código no repositório criado no GitHub para o projeto e dividindo o trabalho entre os diferentes membros do time.


## Implementação do CRUD básico

1. Pense nas funcionalidades CRUD que você pretende incluir no seu sistema. Por exemplo, no [Oldenburg](https://gitlab.labes.inf.ufes.br/labes/oldenburg) já foi feito um cadastro de Workshops (como resultado do [tutorial de CRUD do JButler](https://gitlab.labes.inf.ufes.br/labes/jbutler/-/wikis/tutorials/JButler-CRUD-Tutorial)) e agora, após o **Lab Web 01 - Base da Aplicação**, precisaremos também de um cadastro de usuários;

      > A classe `Usuario` e o seu DAO já foram criados no roteiro de laboratório anterior. Pode-se pensar, no entanto, em cadastros de objetos de classes que ainda não existam. No Oldenburg, por exemplo, poderíamos querer um cadastro de áreas de pesquisa para que cada estudante possa indicar os assuntos que prefere revisar e também indicar a área de pesquisa dos artigos submetidos. A criação de novas classes de domínio e seus DAOs segue as instruções do roteiro anterior.
      > 
      > Também é preciso refletir neste momento quais funcionalidades são CRUD e quais não são. Novamente usando Oldenburg como exemplo, os cadastros de usuário e de assuntos seriam algo que um administrador precisaria fazer e, portanto, se encaixam bem como CRUDs. Já o envio de artigos por parte dos estudantes não se encaixa bem como CRUD e deve ser pensada e desenvolvida uma interface com usuário mais adequada (mostrar ao estudante apenas Workshops ativos no momento, permitir escolher um Workshop e enviar as informações para este), bem como uma classe de serviço apenas para os cenários específicos deste caso de uso (ex.: submeter um artigo, ver a submissão, retirar uma submissão). Neste caso, o JButler pode continuar ajudando com a classe de domínio e o DAO apenas.

2. Implemente os CRUDs planejados conforme as instruções do [tutorial de CRUD do JButler](https://gitlab.labes.inf.ufes.br/labes/jbutler/-/wikis/tutorials/JButler-CRUD-Tutorial), seção **Implement the CRUD (application, controller and view)**;

      > Dicas: tenha o `toString()` implementado na classe de domínio objeto do CRUD para as mensagens ficarem mais apropriadas. Para o campo papel (cujo tipo é enumerado), sugere-se ter no tipo enumerado um campo que armazene uma chave, exemplo:
      >
      > ```java
      > public enum Role {
      >   ADMIN("admin"), PROFESSOR("professor"), STUDENT("student");
      > 
      >   private final String key;
      > 
      >   Role(String key) {
      >     this.key = key;
      >   }
      > 
      >   public String getKey() {
      >     return key;
      >   }
      > }
      > ```
      > 
      > Você pode então configurar textos no _resource bundle_ colocando as chaves usadas no tipo enumerado como parte da chave da mensagem:
      > 
      > ```
      > manageUsers.field.role.admin = Administrator
      > manageUsers.field.role.professor = Professor
      > manageUsers.field.role.student = Student
      > ```
      > 
      > E então na hora de mostrar este dado na página XHTML, concatenar o prefixo `manageUsers.field.role.` com a chave obtida no objeto do tipo enumerado:
      > 
      > ```xhtml
      > <p:column headerText="#{msgsCore['manageUsers.field.role']}" sortBy="#{entity.role}" filterBy="#{entity.role}" filterStyle="display: none">
      >   <h:outputText value="#{msgsCore['manageUsers.field.role.'.concat(entity.role.key)]}" />
      > </p:column>
      >
      > <p:selectOneRadio id="roleField" value="#{manageUsersController.selectedEntity.role}" unselectable="true" required="true">
      >   <f:selectItems value="#{manageUsersController.roles}" var="obj" itemLabel="#{msgsCore['manageUsers.field.role.'.concat(obj.key)]}" itemValue="#{obj}" />
      > </p:selectOneRadio>
      > ```
      > 
      > O primeiro trecho acima (`<p:column />`) mostra o papel do usuário na tabela de listagem de usuários. O segundo (`<p:selectOneRadio />`) é um campo do tipo botão de rádio a ser usado no formulário de cadastro. Para que o botão de rádio liste as opções (os diferentes papeis disponíveis), adicione ao controlador:
      > 
      > ```java
      >   private final List<Role> roles = List.of(Role.values());
      >   public List<Role> getRoles() {
      >     return roles;
      >   }
      > ```
      > 


## Inclusão de regras de validação

3. Os CRUDs básicos já incluem validações simples como, por exemplo, se um campo é obrigatório, se pode ter até N caracteres, se é um número ou data válida, etc. Pense nas validações mais complexas que você precisará implementar. Por exemplo, no Oldenburg:

      - Não podem haver 2 workshops com a mesma sigla;
      - O prazo de revisão de um workshop deve ser posterior ao prazo de submissão;
      - Não se pode excluir um workshop de um ano anterior;
      - Não podem haver 2 usuários com o mesmo email;
      - Não se pode excluir um usuário administrador.

4. Seguindo o [tutorial de validação de CRUDs do JButler](https://gitlab.labes.inf.ufes.br/labes/jbutler/-/wikis/tutorials/JButler-CRUD-Validation), implemente as validações desejadas.

      > Algumas validações (ex.: não ter 2 objetos com o mesmo valor para um determinado atributo) podem envolver criação de consultas ao banco de dados, como no próprio exemplo do tutorial de validação com JButler. Aprenderemos mais sobre como fazer estas consultas nas próximas aulas, portanto talvez seja um desafio muito grande implementar algumas das validações planejadas no passo anterior. Tente implementar alguma parecida com a que é exemplificada no tutorial.

