# Lab Web 03 - Funcionalidades não CRUD

Neste roteiro, usaremos como base o que foi desenvolvido nos **Lab Web** anteriores, adicionando funcionalidades que não obedecem à lógica de CRUD (não são cadastros). Uma rápida discussão sobre nem toda funcionalidade de um sistema de informação ser CRUD foi feita no roteiro anterior. Como os deamis, este roteiro pode ser feito com sua IDE favorita.

Sugere-se ainda continuar aproveitando estes roteiros de laboratório para desenvolver parte do trabalho prático da disciplina. Desta forma, os roteiros podem ser feitos em times, compartilhando o código no repositório criado no GitHub para o projeto e dividindo o trabalho entre os diferentes membros do time.


## Implementação de uma funcionalidade não CRUD

1. Pense em funcionalidades que não se encaixam na lógica de CRUD que você pretende incluir no seu sistema. Por exemplo, no [Oldenburg](https://gitlab.labes.inf.ufes.br/labes/oldenburg) o envio de artigos por parte dos estudantes não se encaixa bem como CRUD e deve ser pensada e desenvolvida uma interface com usuário mais adequada (mostrar ao estudante apenas Workshops ativos no momento, permitir escolher um Workshop e enviar as informações para este), bem como uma classe de serviço apenas para os cenários específicos deste caso de uso (ex.: submeter um artigo, ver a submissão, retirar uma submissão);

2. Implemente a(s) classe(s) de domínio e o(s) respectivo(s) DAO(s) necessário(s) para implementar a funcionalidade escolhida. Se necessário, vide [Tutorial do JButler](https://gitlab.labes.inf.ufes.br/labes/jbutler/-/wikis/tutorials/JButler-CRUD-Tutorial#implement-the-domain-and-persistence-classes). Para o envio de artigos no Oldenburg, foi projetada uma nova classe, `Submission`, conforme projeto do Oldenburg no Visual Paradigm, disponibilizado juntamente com este roteiro;

3. Na página do _template_, adicione um novo link no menu para sua funcionalidade, apontando para uma página a ser construída em seguida. No exemplo do Oldenburg, o link aponta para `core/submitPaper/index`;

4. Crie a nova página Web e vá satisfazendo as dependências, caminhando pela arquitetura da aplicação em direção ao _backend_ até onde for necessário. No exemplo do Oldenburg:

      1. A página Web precisa de um controlador que entregue a lista de Workshops ativos (cuja data de submissão não tenham expirado). Crie o controlador como um bean nomeado (`@Named`) e defina seu escopo;
      2. O controlador precisa de uma classe de serviço que responda ao cenário "Ver Workshops Ativos". Crie a classe de serviço como um EJB;
      3. A classe de serviço precisa que o DAO de Workshop recupere os objetos cujas datas de submissão sejam hoje ou no futuro (ou seja, não expiraram). Adicione este novo método no DAO;
      4. Volte em direção à página, implementando serviço, controle e, finalmente, adicionando os componentes gráficos (ex.: um [Carousel](https://www.primefaces.org/showcase/ui/data/carousel.xhtml)) que mostrarão os dados dos Workshops ativos e um botão para escolher o Workshop ao qual submeter o artigo;
      5. Para implementar a ação do botão, siga novamente em direção ao _backend_ como no início e assim por diante, até a implementação completa do caso de uso.

5. Visite o [repositório do Oldenburg no LabES](https://gitlab.labes.inf.ufes.br/labes/oldenburg) e veja como a funcionalidade de envio de artigos foi implementada:

      - [Visão](https://gitlab.labes.inf.ufes.br/labes/oldenburg/-/tree/main/src/main/webapp/core/submitPaper?ref_type=heads): `/core/submitPaper`;
      - [Controle](https://gitlab.labes.inf.ufes.br/labes/oldenburg/-/blob/main/src/main/java/br/ufes/inf/labes/oldenburg/core/controller/SubmitPaperController.java?ref_type=heads): `SubmitPaperController`;
      - [Aplicação](https://gitlab.labes.inf.ufes.br/labes/oldenburg/-/blob/main/src/main/java/br/ufes/inf/labes/oldenburg/core/application/SubmitPaperServiceBean.java?ref_type=heads): `SubmitPaperService`;
      - [Domínio](https://gitlab.labes.inf.ufes.br/labes/oldenburg/-/blob/main/src/main/java/br/ufes/inf/labes/oldenburg/core/domain/Submission.java?ref_type=heads): `Submission`.
