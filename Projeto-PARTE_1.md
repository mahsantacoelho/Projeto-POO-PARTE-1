# C06 — NP2 | Parte 1 — Produtos e Estoque

**Responsabilidade:** integrante 1. **Linguagem:** Java. **Entregue:** quatro classes de domínio, quatro classes de exceção compartilhadas e um programa separado para testes.

Esta entrega foi construída a partir do UML do grupo (`ControleEstoque_UML.puml`) e **não implementa sozinha o sistema final**. As outras partes implementarão movimentações, persistência, interface de menu, comunicação e threads.

## 1. Estrutura de pastas

```text
src/
  estoque/
    modelo/
      Produto.java
      ProdutoPerecivel.java
      ProdutoNaoPerecivel.java
      Estoque.java
    excecoes/
      EstoqueException.java
      ProdutoNaoEncontradoException.java
      ProdutoDuplicadoException.java
      EstoqueInsuficienteException.java
    testes/
      TesteParte1.java
```

Cada classe pública tem arquivo `.java` próprio, e as pastas correspondem aos pacotes declarados na primeira linha. **Não retire os `package` sem combinar com o grupo.** As demais partes devem utilizar as mesmas estruturas e importar esses pacotes.

## 2. Classes e decisões

### `Produto` — classe abstrata

**Atributos privados**: `codigo`, `nome`, `quantidade`, `estoqueMinimo`. Os três primeiros dados de identidade/regra imutáveis após criação (`codigo`, `nome`, `estoqueMinimo`) são `final`; a `quantidade` pode mudar, mas apenas por métodos que validam a alteração.

**Construtor:** `Produto(int codigo, String nome, int quantidade, int estoqueMinimo)`.

**Métodos públicos:**

- `int getCodigo()`;
- `String getNome()`;
- `int getQuantidade()`;
- `int getEstoqueMinimo()`;
- `void aumentarQuantidade(int qtd)`;
- `void diminuirQuantidade(int qtd)`;
- `boolean estaAbaixoDoMinimo()`;
- `abstract boolean verificarCondicao()`.

**Regras:** código positivo; nome não nulo/não vazio e sem `;` ou quebras de linha (compatível com CSV simples); quantidades e estoque mínimo não negativos; entradas/saídas estritamente positivas; saída não pode exceder o saldo; entrada não pode causar overflow de `int`. `estaAbaixoDoMinimo()` testa `quantidade < estoqueMinimo` (igualdade NÃO dispara alerta).

**POO:** abstração, construtor, atributos, encapsulamento, modificadores de acesso, métodos de instância, método abstrato, herança pela superclasse e `static` no método auxiliar privado. `synchronized` nos acessos à quantidade protege a leitura/alteração de uma quantidade individual com a thread monitora.

### `ProdutoPerecivel` — subclasse concreta

Atributo extra `dataValidade: LocalDate` (obrigatório). Construtor com os quatro campos comuns e a validade. Implementa `getDataValidade()` e `verificarCondicao()`: **true** se a validade é hoje ou uma data futura; **false** se anterior a hoje. `@Override` mostra a sobrescrita do método abstrato. `super(...)` invoca o construtor da classe pai.

### `ProdutoNaoPerecivel` — subclasse concreta

Construtor com os quatro campos comuns. `verificarCondicao()` devolve **true**, porque esta versão não possui data de vencimento cadastrada para esse tipo. **Não significa inspeção física ou garantia de qualidade.**

### `Estoque` — classe que agrupa os produtos

Campo privado `Map<Integer, Produto> produtos`, armazenado como `LinkedHashMap` para manter ordem de cadastro. Seus métodos públicos são:

- `Estoque()`;
- `void adicionarProduto(Produto produto)`;
- `Produto buscarProduto(int codigo)`;
- `List<Produto> listarProdutos()`;
- `void aumentarQuantidade(int codigo, int qtd)`;
- `void diminuirQuantidade(int codigo, int qtd)`.

Ao cadastrar código repetido, lança `ProdutoDuplicadoException`. Ao buscar código não cadastrado, lança `ProdutoNaoEncontradoException`. `listarProdutos()` devolve uma **lista nova**, porém com **as referências originais**: a pessoa 4 pode consultar os mesmos objetos pelo monitor. `aumentarQuantidade` e `diminuirQuantidade` localizam e **delegam** ao próprio objeto `Produto`.

**Relacionamento UML**: `Estoque o-- "0..*" Produto` (agregação). Produtos podem ser criados fora do estoque; há uma referência, não uma herança. Os acessos ao `Map` são sincronizados; a alteração da quantidade é sincronizada pelo `Produto`. Uma lista é uma cópia das referências, não um snapshot atômico do saldo de todos os produtos.

### Exceções compartilhadas

- `EstoqueException extends RuntimeException`: base comum.
- `ProdutoNaoEncontradoException extends EstoqueException`.
- `ProdutoDuplicadoException extends EstoqueException`.
- `EstoqueInsuficienteException extends EstoqueException`.

Outros valores inválidos geram `IllegalArgumentException`; validade nula gera `NullPointerException` pela validação `Objects.requireNonNull`. **Não criem versões duplicadas dessas classes na parte 2.**

## 3. Exatamente onde está a POO

| Conceito | Código | Como explicar |
|---|---|---|
| Classe | `class Estoque` | Molde para criar objetos com dados e operações. |
| Objeto/instância | `new Estoque()`, `new ProdutoPerecivel(...)` em `TesteParte1` | `new` cria uma instância concreta. |
| Atributo | `private int quantidade` em `Produto` | Estado pertencente a cada produto. |
| Método | `aumentarQuantidade()` em `Produto` | Comportamento que modifica o estado do objeto. |
| Encapsulamento | atributos `private`, getters e métodos de alteração | Impede modificação externa sem validação. |
| Construtor | `Produto(...)` | Valida e inicializa um objeto no momento da criação. |
| `this` | `this.codigo = codigo` | Diferencia o atributo do parâmetro. |
| `super` | `super(...)` nas subclasses | Executa o construtor da classe pai. |
| Herança | `extends Produto` | Subclasses reutilizam dados e métodos da superclasse. |
| Classe abstrata | `abstract class Produto` | Não é instanciada diretamente. |
| Método abstrato | `abstract boolean verificarCondicao()` | Obriga as subclasses a fornecer implementação. |
| Sobrescrita | `@Override` nas subclasses | Substitui a implementação do mesmo método. |
| Polimorfismo | `Produto p = new ProdutoPerecivel(...)`; `p.verificarCondicao()` no teste | A chamada se resolve conforme o tipo real do objeto. |
| Agregação | `Map<Integer, Produto>` dentro de `Estoque` | Estoque tem referências a produtos independentes. |
| Coleções | `Map`, `LinkedHashMap`, `List`, `ArrayList` | Guarda uma quantidade variável de produtos com busca pelo código. |
| `static` | `validarQuantidadePositiva` | Rotina de validação que não depende de uma instância. |
| Exceções | `throw new ProdutoDuplicadoException(...)` | Comunica que uma regra de negócio foi violada. |
| Concorrência | `synchronized` em quantidade e mapa | Coordena acessos de threads aos objetos compartilhados. |

**Não afirmar que**: `Produto` é uma interface; `Estoque` herda de `Produto`; `synchronized` sozinho cria uma thread; `Map` é uma classe concreta; uma lista copiada duplica os objetos; `verificarCondicao()` verifica qualidade física de não perecíveis. Essas afirmações estariam erradas.

## 4. Contrato para a Pessoa 2

A Pessoa 2 deve importar:

```java
import estoque.modelo.Estoque;
import estoque.modelo.Produto;
import estoque.excecoes.EstoqueInsuficienteException;
import estoque.excecoes.ProdutoNaoEncontradoException;
```

Implementação de `Entrada.executar(Estoque estoque)`:

```java
estoque.aumentarQuantidade(getCodigoProduto(), getQuantidade());
```

Implementação de `Saida.executar(Estoque estoque)`:

```java
estoque.diminuirQuantidade(getCodigoProduto(), getQuantidade());
```

**IMPORTANTE:** `Entrada`/`Saida` não devem escrever diretamente `produto.quantidade` (o atributo é `private`). O serviço de movimentações será responsável por persistir os saldos e o histórico; **esta parte não grava arquivos**. O serviço deve decidir como reagir a falhas de persistência, pois dois CSVs não garantem transação atômica.

## 5. Contrato para a Pessoa 3

Ao ler `produtos.csv`, construir cada `ProdutoPerecivel` ou `ProdutoNaoPerecivel` conforme a coluna `TIPO`, e inserir usando `Estoque.adicionarProduto`. Para salvar, percorrer `estoque.listarProdutos()` e usar `instanceof ProdutoPerecivel` para obter `getDataValidade()` (alternativamente outro desenho acordado com o grupo). Datas ISO `LocalDate.parse` / `toString`.

**Formato acordado:** `TIPO;CODIGO;NOME;QUANTIDADE;MINIMO;VALIDADE`. Tipo `P` para perecível, `N` para não perecível; validade vazia para `N`. Esta parte rejeita `;` e quebras de linha em nomes para manter o CSV simples.

### Exemplo de polimorfismo que a `Main` pode exibir

```java
for (Produto p : estoque.listarProdutos()) {
    System.out.println(p.getNome() + " => " + p.verificarCondicao());
}
```

## 6. Contrato para a Pessoa 4

`MonitorEstoque` recebe uma instância de `Estoque` e chama:

```java
for (Produto p : estoque.listarProdutos()) {
    if (p.estaAbaixoDoMinimo()) {
        notificador.notificar("Produto " + p.getNome() + " abaixo do minimo!");
    }
}
```

`listarProdutos()` pode ser chamado de outra thread; os métodos de quantidade também são sincronizados. **O monitor apenas consulta e avisa**, sem alterar o saldo. O encerramento da thread é tarefa da `Main` com `encerrar`, `interrupt` e `join` conforme o UML do grupo.

## 7. Como compilar e executar os testes

Na raiz desta pasta, em um terminal com o JDK instalado:

**Windows PowerShell**:

```powershell
New-Item -ItemType Directory -Force bin | Out-Null
javac -encoding UTF-8 -d bin src/estoque/excecoes/*.java src/estoque/modelo/*.java src/estoque/testes/*.java
java -cp bin estoque.testes.TesteParte1
```

**Linux/macOS (bash)**:

```bash
mkdir -p bin
javac -encoding UTF-8 -d bin $(find src -name '*.java')
java -cp bin estoque.testes.TesteParte1
```

Resultado esperado: diversas linhas `[OK]` e `TODOS OS TESTES DA PARTE 1 PASSARAM.`

`TesteParte1` é um **testador auxiliar**, não substitui a `Main` que a Pessoa 3 criará. Os testes não salvam dados em disco nem inicializam a thread: essas são responsabilidades futuras.

## 8. Perguntas que o professor pode fazer

**1. Onde está a herança?** Nas duas subclasses que escrevem `extends Produto`; reutilizam getters e operações da classe pai.

**2. Onde está o polimorfismo?** No teste, as variáveis do tipo `Produto` apontam a objetos das subclasses. A chamada `verificarCondicao()` usa o comportamento da subclasse real.

**3. Por que `Produto` é abstrata?** Porque não queremos cadastrar uma instância genérica sem definir a regra específica de condição; exige uma subclasse concreta.

**4. Por que os atributos são `private`?** Para impedir alterações diretas que desrespeitem regras como saldo não negativo e código único.

**5. `Estoque` e `Produto` têm herança?** Não. Um estoque tem produtos; representamos agregação, não uma relação “é um”.

**6. `Map` e `ArrayList` são iguais?** Não. O `Map` relaciona chave e valor (código -> produto); a `List` é usada para devolver uma sequência de produtos.

**7. O que é `@Override`?** Indica que o método da subclasse está sobrescrevendo um método declarado na superclasse; o compilador verifica a assinatura.

**8. Onde está a interface?** Nesta parte é usado o tipo de interface da biblioteca `Map`. As interfaces autorais obrigatórias (`RepositorioDados` e `Notificavel`) pertencem às Pessoas 3 e 4.

**9. Essa parte possui thread?** Não: contém sincronização para compartilhamento seguro do estado. A thread propriamente dita será implementada pela Pessoa 4.

**10. Onde está o banco ou os arquivos?** Não estão nesta parte; a pessoa 3 implementará a persistência definida no UML.

## 9. Checklist de integração

- [x] Classes públicas seguem os nomes e assinaturas do UML.
- [x] Herança, sobrescrita, método abstrato e polimorfismo demonstráveis.
- [x] Cadastro duplicado e busca inexistente geram as exceções acordadas.
- [x] Saída nunca deixa saldo negativo; entrada evita overflow.
- [x] Testes Java compilados e executados com sucesso (na criação do módulo).
- [ ] Pessoa 2 integra movimentos usando `aumentarQuantidade`/`diminuirQuantidade`.
- [ ] Pessoa 3 implementa salvamento e carregamento e a `Main`.
- [ ] Pessoa 4 implementa o monitor e suas notificações.
- [ ] Grupo executa testes de ponta a ponta, revisa e atualiza o UML.
- [ ] Grupo declara de forma verdadeira o uso de IA no README principal.

**Declaração de IA para o README principal:** informem que o ChatGPT foi utilizado no planejamento do UML, na elaboração do módulo de Produtos e Estoque, nos comentários e na sugestão de testes, e quais partes os integrantes revisaram, alteraram e validaram. O professor exige transparência.
