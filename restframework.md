====== Haftware REST Framework ======

O Haftware REST Framework é uma coletânea de classes Java visando facilitar e acelerar o desenvolvimento de APIs REST confiáveis e estáveis.

===== Começando um novo projeto =====

Toda a configuração do framework é feita através de uma única classe, que normalmente tem o nome do projeto e deve extender a superclasse ''RestApi''. Essa classe deve ter a anotação ''@WebServlet(urlPatterns = "/api/*")'':

<code java>
@WebServlet(urlPatterns = "/api/*")
public class nome_do_projeto extends RestApi {
</code>

Ao extender a classe //RestApi//, será necessário implementar o método ''void configura()'' que é o método que faz a configuração do Framework quando ele começa a rodar.

O método ''configura()'' faz uso de diversas funções para configurar o framework, as principais estão descritas abaixo:

^ Descrição      ^ Função       ^
|''setNomeProjeto(String nome)'' | Define o nome do projeto. |
|''setBaseUrl(String url)'' | Define a URL base de acesso ao projeto. |
|''addResourceClass(String path, Class classe)'' | Adiciona uma nova classe de recursos no path indicado. |
|''useResourceFile(String nome)'' | Adiciona um novo arquivo de recursos com o nome indicado. |
|''setAuthManager(Class classe)'' | Define a classe de autenticação a ser usada na autenticação automática do framework. |
|''addFiltersClass(Class classe)'' | Adiciona uma classe de filtros de métodos para ser usada. |

===== Classes de recursos =====

Uma classe de recursos é uma classe que contém os métodos da API REST referentes à uma tarefa específica (Autenticação, por exemplo), quando uma classe de recurso é adicionado ao projeto, o Framework faz uma varredura nessa classe em busca de todos os métodos que contenham a anotação ''@Path'', preparando-os para atenderem as chamadas à API.

__Idealmente nunca utilizar variáveis nas classes de recurso para evitar problemas com o multi-threading automático do framework.__

O Framework REST da Haftware suporta ilimitadas classes de recursos, o projeto pode ter quantas classes de recursos forem necessárias.
   * No método de configuração(''configura()''), as classes de recursos são adicionadas com o método:
      ''addResourceClass(String url, Class c)'', onde 'url' é o caminho base de acesso aos métodos da classe
      
==== Definindo os métodos ====
Após criar uma classe de recurso e adicioná-la ao framework com a função ''addResourceClass'', podemos começar a adicionar os métodos daquela classe, cada método a ser adicionado deve seguir as seguintes regras:
  * Possuir a anotação ''@Path("caminho-do-metodo")''
  * Ter um único parâmetro de entrada extendo a MethodInput(ver [[haftware:restframework#a_methodinput|MethodInput]])
  * Não ter saída(ser do tipo ''void'')

> Um método somente com os requisitos acima, será disponibilizado com um método do tipo ''GET'', necessitando autenticação e não transacional

Opcionalmente, podem/devem ser usadas outras anotações para indicar ao Framework o comportamento do método:
^ Anotação      ^ Função       ^
| ''POST'' | Indica que o método é chamado através do método HTTP POST |
| ''GET''  | Indica que o método é chamado através do método HTTP GET (padrão)|
| ''SemAuth'' | Indica que o método pode ser chamado sem estar autenticado (ver [[haftware:restframework#autenticacao_automatica|Autenticação automática]])|
| ''Transacional'' | Indica que o framework deve abrir uma transação no banco de dados para o método, permitindo à ele alterar registros no Banco de Dados |
**Exemplo:**
<code java>
@Path("logout")
@POST
public void logout(InputVazia inp){
    // ....
}
</code>

Com isso feito, o método já será aceito pelo framework e estará operando, ao chamá-lo do cliente, será retornada a resposta padrão do framework:
<code json>
{
  status: "OK"
}
</code>

===== A MethodInput =====
Todos o métodos da API recebem uma ''MethodInput'', essa classe é a interface do método com todos os recursos do framework.

Opcionalmente, a MethodInput pode se auto-validar, garantindo que, quando o método recebê-la, já estará tudo OK.

A classe ''MethodInput'' é uma superclasse das inputs dos métodos, não podendo ser diretamente colocado como párametro de entrada do método, para cada método POST que receber algum dado de entrada, a class ''MethodInput'' deve ser extendida, criar uma Input é muito simples:

<code java>
public class MinhaInput extends MethodInput {
    String titulo;
    String descricao;
}
</code>

  * **Para métodos POST que não recebem nenhuma entrada, deve ser colocada na entrada a classe ''InputVazia'', já presente no framework**

Ao criar uma classe filha da ''MethodInput'', o compilador dará um erro, pois devem ser adicionadas 3 funções na classe, ficando assim:

<code java>
public class MinhaInput extends MethodInput {
    String titulo;
    String descricao;

    @Override
    protected void filtraCampos() {
    }

    @Override
    protected void validaCampos(HwResponse res) {
    }
    
    @Override
    protected String getMethodNamespace() {
    }
}
</code>
A função de cada método está descrita abaixo:
==== filtraCampos ====
Esse método serve para filtrar os campos da input que foram recebidos do front-end mas não devem ser considerados.

> **Por exemplo:** no cadastro de usuário, a input terá um campo do tipo ''Usuario'', nessa classe devem ser recebidos somente o ''nome'' e ''senha''. Porém, por algum motivo, foi recebido o campo ''dataCadastro'', esse campo pode ser limpo na função ''filtraCampos'', simplesmente com a linha \\ ''usuario.dataCadastro = null''.
==== validaCampos ====
Nesse método deve ser feita a validação dos campos recebidos na input, colocando os erro na HwResponse recebida:
<code java>
public class MinhaInput extends MethodInput {
    String titulo;
    String descricao;
    ...
    @Override
    protected void validaCampos(HwResponse res) {
        if(titulo == null)
            res.addErroValidacaoCampo("titulo", CodigoErroCampo.NAO_INFORMADO, "Esse campo é obrigatório");
    }
    ...
}
</code>
> Caso esse método adicione qualquer erro na HwResponse recebida, o framework automaticamente cancelará o request à API, retornando o erro ao front-end

==== getMethodNamespace ====
Esse método é utilizado para diferenciar campos de diferentes telas em aplicações //React//, basicamente o framework colocara o nome retornado por essa função antes de todos os nomes de campos com erros de validação retornados na resposta, por exemplo:
<code java>
...
@Override
protected String getMethodNamespace () {
    return "cadastro"
}
...
</code>
Fará com que campos validados sejam retornados como ''cadastro.<nome-do-campo>''
> Para aplicações que não sejam //React//, pode-se retornar ''null'' nessa função.
===== Haftware Response =====

A Haftware Response (''HwResponse'') é a estrutura padrão de uma resposta do servidor REST, tendo o conteúdo, o status, os erros e, opcionalmente, uma mensagem de alerta e um redirect.

Como.
===== Acesso ao banco =====
===== Autenticação automática =====
===== Filtros de métodos =====
===== Tasks agendadas =====
===== Arquivos de recursos =====
===== O Gerenciador de arquivos =====
===== O Gerenciador de websockets =====
===== O Gerenciador de pagamentos =====
===== Thread-Safety =====
