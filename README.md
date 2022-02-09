# Design Patterns

## Sobre

Anotações relacionadas aos meus estudos sobre design patters em javascript com React e Next.js. Os exemplos em sua maioria fora extraídos do [patterns](patterns.dev)

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
