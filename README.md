## Introduction to Mustache

# 1. Visão Geral
Neste artigo, vamos nos concentrar nos modelos Mustache e usar uma de suas APIs Java para produzir conteúdo HTML dinâmico.

Mustache é um mecanismo de modelo sem lógica para a criação de conteúdo dinâmico como HTML, arquivos de configuração, entre outras coisas.

# 2. Introdução
Simplificando, o mecanismo é classificado como sem lógica porque não possui construções que suportam instruções if-else e loops for.

Os modelos do Mustache consistem em nomes de tag cercados por {{}} (que se assemelham a bigodes - daí o nome) e são apoiados por um objeto de modelo que contém os dados do modelo.

# 3. Dependência Maven
A compilação e execução dos modelos são suportadas por vários idiomas - tanto do lado do cliente quanto do lado do servidor.

Para poder processar os templates do Java, fazemos uso de sua biblioteca Java que pode ser adicionada como uma dependência Maven.

Java 8+:
```
<dependency>
    <groupId>com.github.spullara.mustache.java</groupId>
    <artifactId>compiler</artifactId>
    <version>0.9.4</version>
</dependency>
```

Java 6/7:
```
<dependency>
    <groupId>com.github.spullara.mustache.java</groupId>
    <artifactId>compiler</artifactId>
    <version>0.8.18</version>
</dependency>
```

Podemos verificar as versões mais recentes da biblioteca no Repositório Central Maven.

# 4. Uso

Vejamos um cenário simples que mostra como:

- Escreva um modelo simples;
- Compilar o template usando a API Java;
- Execute-o fornecendo os dados necessários.

### 4.1. Um modelo simples de bigode
Vamos criar um modelo simples para exibir os detalhes de uma tarefa:

```
<h2>{{title}}</h2>
<small>Created on {{createdOn}}</small>
<p>{{text}}</p>
```

No modelo acima, os campos entre chaves ({{}}) podem ser:

- métodos e propriedades de uma classe Java;
- chaves de um objeto de mapa.

### 4.2. Compilando o modelo de bigode
Podemos compilar o modelo conforme mostrado abaixo:

```
MustacheFactory mf = new DefaultMustacheFactory();
Mustache m = mf.compile("todo.mustache");
```

MustacheFactory procura o modelo fornecido no caminho de classe. Em nosso exemplo, colocamos todo.mustache em src/main/resources.

### 4.3. Executando o modelo Mustache
Os dados fornecidos para o modelo serão uma instância da classe Todo, cuja definição é:

```
public class Todo {
    private String title;
    private String text;
    private boolean done;
    private Date createdOn;
    private Date completedOn;
    
    // constructors, getters and setters
}
```

O modelo compilado pode ser executado para obter HTML conforme mostrado abaixo:

```
Todo todo = new Todo("Todo 1", "Description");
StringWriter writer = new StringWriter();
m.execute(writer, todo).flush();
String html = writer.toString();
```

# 5. Seções e iterações do bigode
Vamos agora dar uma olhada em como listar os todos. Para iterar os dados de uma lista, usamos as seções Mustache.

Uma seção é um bloco de código que se repete uma ou mais vezes, dependendo do valor da chave no contexto atual.

É algo como:

```
{{#todo}}
<!-- Other code -->
{{/todo}}
```

Uma seção começa com uma cerquilha (#) e termina com uma barra (/), onde cada um dos sinais é seguido pela chave cujo valor é usado como base para renderizar a seção.

A seguir estão os cenários que podem ocorrer dependendo do valor da chave:

### 5.1. Seção com lista não vazia ou valor não falso
Vamos criar um template todo-section.mustache que usa uma seção:

```
{{#todo}}
<h2>{{title}}</h2>
<small>Created on {{createdOn}}</small>
<p>{{text}}</p>
{{/todo}}
```

Vejamos este modelo em ação:

```
@Test
public void givenTodoObject_whenGetHtml_thenSuccess() 
  throws IOException {
 
    Todo todo = new Todo("Todo 1", "Todo description");
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todo.mustache");
    Map<String, Object> context = new HashMap<>();
    context.put("todo", todo);
 
    String expected = "<h2>Todo 1</h2>";
    assertThat(executeTemplate(m, todo)).contains(expected);
}
```

Vamos criar outro template todos.mustache para listar os todos:

```
{{#todos}}
<h2>{{title}}</h2>
{{/todos}}
```

E crie uma lista de todos usando-o:

```
@Test
public void givenTodoList_whenGetHtml_thenSuccess() 
  throws IOException {
 
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todos.mustache");
 
    List<Todo> todos = Arrays.asList(
      new Todo("Todo 1", "Todo description"),
      new Todo("Todo 2", "Todo description another"),
      new Todo("Todo 3", "Todo description another")
    );
    Map<String, Object> context = new HashMap<>();
    context.put("todos", todos);
 
    assertThat(executeTemplate(m, context))
      .contains("<h2>Todo 1</h2>")
      .contains("<h2>Todo 2</h2>")
      .contains("<h2>Todo 3</h2>");
}
```

### 5.2. Seção com lista vazia ou valor falso ou nulo
Vamos testar todo-section.mustache com um valor nulo:

```
@Test
public void givenNullTodoObject_whenGetHtml_thenEmptyHtml() 
  throws IOException {
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todo-section.mustache");
    Map<String, Object> context = new HashMap<>();
    assertThat(executeTemplate(m, context)).isEmpty();
}
```

E da mesma forma, teste todos.mustache com uma lista vazia:

```
@Test
public void givenEmptyList_whenGetHtml_thenEmptyHtml() 
  throws IOException {
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todos.mustache");
 
    Map<String, Object> context = new HashMap<>();
    assertThat(executeTemplate(m, context)).isEmpty();;
}
```

# 6. Seções invertidas

Seções invertidas são aquelas que são renderizadas apenas uma vez com base na inexistência da chave ou valor falso ou nulo ou uma lista vazia. Em outras palavras, eles são renderizados quando uma seção não é renderizada.

Eles começam com um acento circunflexo (^) e terminam com uma barra (/) conforme mostrado abaixo:

```
{{#todos}}
<h2>{{title}}</h2>
{{/todos}}
{{^todos}}
<p>No todos!</p>
{{/todos}}
```

O modelo acima quando fornecido com uma lista vazia:

```
@Test
public void givenEmptyList_whenGetHtmlUsingInvertedSection_thenHtml() 
  throws IOException {
 
    Mustache m = MustacheUtil.getMustacheFactory()
      .compile("todos-inverted-section.mustache");
  
    Map<String, Object> context = new HashMap<>();
    assertThat(executeTemplate(m, context).trim())
      .isEqualTo("<p>No todos!</p>");
}
```

# 7. Lambdas
Os valores das chaves de uma seção do mustache podem ser uma função ou uma expressão lambda. Nesse caso, a expressão lambda completa é chamada passando o texto dentro da seção como um parâmetro para a expressão lambda.

Vejamos um template todos-lambda.mustache:

```
{{#todos}}
<h2>{{title}}{{#handleDone}}{{doneSince}}{{/handleDone}}</h2>
{{/todos}}
```

A chave handleDone é resolvida em uma expressão lambda Java 8 conforme mostrado abaixo:

```
public Function<Object, Object> handleDone() {
    return (obj) -> done ? 
      String.format("<small>Done %s minutes ago<small>", obj) : "";
}
```

O HTML gerado pela execução do modelo acima é:

```
<h2>Todo 1</h2>
<h2>Todo 2</h2>
<h2>Todo 3<small>Done 5 minutes ago<small></h2>
```

# 8. Conclusão
Neste artigo introdutório, vimos como criar modelos de mustache com seções, seções invertidas e lambdas. E usamos a API Java para compilar e executar os modelos, fornecendo dados relevantes.

Existem alguns recursos mais avançados do Mustache que valem a pena explorar, como:

- Fornecer um callable como um valor que resulta em uma avaliação simultânea;
- Usando DecoratedCollection para obter o primeiro, o último e o índice dos elementos da coleção;
- Invert API que dá os dados dados o texto e o modelo.

