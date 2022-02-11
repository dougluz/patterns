# Design Patterns

## Sobre

Anotações relacionadas aos meus estudos sobre design patterns em javascript com React e Next.js. Os exemplos em sua maioria fora extraídos do [patterns](http://patterns.dev)

### Singleton

*Singletons* são classes que podem ser instanciadas apenas umas vez e podem ser acessadas globalmente. A única instância pode ser compartilhada por toda a aplicação, o que torna eles ótimos para gerênciar o estado de uma aplicação.

Pode economizar muita memória por apenas criar uma instância do objeto, mas devem ser evitadas em *javascript*.

Podemos criar um *counter* usando *singleton* da seguinte forma:

```js
let count = 0;

const counter = {
  increment() {
    return ++count;
  },
  decrement() {
    return --count;
  }
};

Object.freeze(counter);
export { counter };
```

Como objetos são passados por referência, os componentes que importassem o *counter* estariam usando a mesma referência para o *counter*.

## Proxy Pattern

Com um objeto *proxy* é possível obter mais controle sobre as interações com certos objetos, quando acessando ou setando um valor.

O proxy faz o papel de intermediario, é como se em vez de conversar diretamente com alguém, você utilizasse uma terceira pessoa como intermediaria. Em *Javascript* em vez de interagirmos diretamente com o objeto, interagimos com o objeto *Proxy*.

Por exemplo, em um objeto com que representa *John Doe*.

```js
const person = {
    name: "John Doe",
    age: 40,
    nationality: "American"
}
```

Em vez de interagirmos diretamente com esse objeto, podemos interagir por meio de um *Proxy*. Para criar um *Proxy* podemos fazer o seguinte

```js
const personProxy = new Proxy(person, {})
```

O segundo argumento do proxy é um objeto que representa o nosso *handler*. Nesse objeto podemos definir comportamentos específicos baseados em cada tipo de interação. Os mais comuns são `get` e `set`.

- `get`: É chamado quando estamos tentando **acessar** uma propriedade;

- `set`: É chamado quando estamos tentando **modificar** uma propriedade.

Para adicionarmos *handlers* ao nosso *proxy* para quando modificarmos as propriedades do nosso objeto podemos utilizar os dois metodos citados acima, passando eles para o nosso objeto que representa o *handler*. No exemplo abaixo queremos que a aplicação loge no console toda vez que alterarmos o objeto ou acessarmos alguma propriedade dele.

```js
const personProxy = new Proxy(person, {
    get: (obj, prop) => {
        console.log(`The value of ${prop} is ${obj[prop]}`)
    },
    set: (obj, prop, value) => {
        console.log(`Changed ${prop} from ${obj[prop]} to ${value}`)
    }
})
```

*Proxys* são ótimos para criarmos **validações**. Podemos, por exemplo, validar se o objeto `person` não está recebendo um nome vazio ou uma *String* na propriedade `age`.

```js
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    if (!obj[prop]) {
      console.log(
        `Hmm.. this property doesn't seem to exist on the target object`
      );
    } else {
      console.log(`The value of ${prop} is ${obj[prop]}`);
    }
  },
  set: (obj, prop, value) => {
    if (prop === "age" && typeof value !== "number") {
      console.log(`Sorry, you can only pass numeric values for age.`);
    } else if (prop === "name" && value.length < 2) {
      console.log(`You need to provide a valid name.`);
    } else {
      console.log(`Changed ${prop} from ${obj[prop]} to ${value}.`);
      obj[prop] = value;
    }
  }
});
```

#### Reflect

O *Javascript* nos fornece uma objeto chamado `Reflect` que torna mais facil manipular o objeto desejado quando trabalhando com *proxies*.

No exemplo acima tentamos modificar e acessar propriedades no objeto desejado com o *proxy* diretamente acessando ou setando o valor com `bracket notation`. Em vez disso, podemos utilizar o objeto `Reflect`.

Em vez de acessarmos a propriedade através de `prop[obj]` ou setar os valores através de `prop[obj] = value`, nos podemos acessar ou modificar as propriedades utilizando o `Reflect.get()` e o `Reflect.set()`. Os metodos recebem os mesmos argumentos que os metodos no objeto manipulador.

```js
const personProxy = new Proxy(person, {
  get: (obj, prop) => {
    console.log(`The value of ${prop} is ${Reflect.get(obj, prop)}`);
  },
  set: (obj, prop, value) => {
    console.log(`Changed ${prop} from ${obj[prop]} to ${value}`);
    Reflect.set(obj, prop, value);
  }
});
```

*Proxies* são uma forma poderosa de adicionar controle sob o comportamento de um objeto. Um *proxy* pode ter varios casos de uso: pode ajudar com validação, formatação, notificações ou *debugging*.

Porém fazer o objeto de *Proxy* executar operações pesadas em cada *handler* pode facilmente afetar a performance da sua aplicação negativamente. É melhor não usar *proxies* para código que necessita de muita performance.

## Provider Pattern

Em alguns casos se faz necessário compartilhar os dados com vários (ou todos) componentes em uma aplicação. Apesar de podermos passar esses dados por meio de  *props*, isso pode se tornar muito complexo se quase todos os componentes da aplicação precisarem de acesso ao valor.

Podemos acabar com um problema chamado *prop drilling*, que é o caso quando passamos *props* por vários níveis na arvore de componentes. Refatorar esse código pode se tornar muito trabalhoso.

Com o *provider pattern* nós podemos tornar os dados disponíveis em multiplos componentes. Em vez de passarmos as props pra todos os componentes, podemos envolver nossos componentes em um *Provider*. O *Provider* é um *high order component* que é disponibilizado para nós por meio do objeto *Context*. É possível criar um contexto usando o método `createContext` que o *React* disponibiliza.

O provider recebe uma *prop* chamada `value` que contem os dados que queremos disponibilizar para os componentes que estão dentro do *Provider*.

```jsx
const DataContext = React.createContext()

function App() {
  const data = { ... }

  return (
    <div>
      <DataContext.Provider value={data}>
        <SideBar />
        <Content />
      </DataContext.Provider>
    </div>
  )
}
```

Cada componente pode acessar o objeto `data` por meio do hook `useContext`. Esse *hook* recebe o contexto que desejamos tornar disponível para aquele componente. O `useContext` é utilizado da seguinte forma:

```jsx
function Header() {
  const { data } = React.useContext(DataContext);
  return <div>{data.title}</div>;
}
```

### Hooks

Podemos criar um *hook* para prover o contexto para nossos componentes. Em vez de termos que importar o `useContext` em todos os componentes que usarão o estado, podemos fazer algo da seguinte forma:

```js
function useThemeContext() {
  const theme = useContext(ThemeContext);
  if (!theme) {
    throw new Error("useThemeContext must be used within ThemeProvider"); 
  }
  return theme;
}
```



### Pros

Torna possível passar dados para muitos componentes sem ter que manualmente passar eles por meio de props.

Reduz o risco de acidentalmente inserir *bugs* quando refatorando o código.

Torna desnecessário lidar com *prop-drilling*, que pode ser um anti-pattern.

Manter algo tipo um estado global fica facil com esse *pattern*.

### Cons

Em alguns casos pode ser problematico e resultar em problemas de performance. Todos os componentes renderizam novamente em cada alteração do estado.
