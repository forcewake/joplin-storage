Работа с callbacks в React / Блог компании HeadHunter

За время свой работы, я периодически сталкивался с тем, что разработчики не всегда четко представляют, каким образом работает механизм передачи данных через props, в частности колбеков, и почему их PureComponents обновляется так часто.

Поэтому в данной статье мы разберемся, как передаются callbacks в React, а также обсудим особенности работы event handlers.

  

### TL;DR

  

1.  Не мешайте JSX и бизнес-логику — это усложнит восприятие кода.
2.  Для небольших оптимизаций кешируйте функции-обработчики в виде classProperties для классов или с помощью useCallback для функций — тогда чистые компоненты не будут перерендериваться постоянно. Особенно кеширование колбеков особенно может пригодиться, чтобы при их передаче в PureComponent не произошло ненужных updating циклов.
3.  Не забывайте о том, что в колбек вам попадает не настоящее событие, а Syntetic event. Если вы выйдете из текущей функции, то вы не сможете обращаться к полям этого события. Кешируйте нужные вам поля, если у вас есть замыкания с асинхронностью.

  

### Часть 1\. Event handlers, кеширование и восприятие кода

React представляет достаточно удобный способ добавления обработчиков событий для html элементов.

Это одна из базовых вещей, с которым знакомится любой разработчик, когда начинает писать на React:

  

    class MyComponent extends Component {
      render() {
        return <button onClick={() => console.log('Hello world!')}>Click me</button>;
      }
    }

Достаточно просто? Из этого кода сразу становится понятно, что произойдет, когда пользователь кликнет на кнопку.

Но что делать, если кода в обработчике становится все больше и больше?

Предположим, что по кнопке мы должны подгрузить и отфильтровать всех, кто не входит в определенную команду (`user.team === 'search-team'`), затем отсортировать их по возрасту.

  

    class MyComponent extends Component {
      constructor(props) {
        super(props);
        this.state = { users: [] };
      }
      render() {
        return (
          <div>
            <ul>
              {this.state.users.map(user => (
                <li>{user.name}</li>
              ))}
            </ul>
            <button
              onClick={() => {
                console.log('Hello world!');
                window
                  .fetch('/usersList')
                  .then(result => result.json())
                  .then(data => {
                    const users = data
                      .filter(user => user.team === 'search-team')
                      .sort((a, b) => {
                        if (a.age > b.age) {
                          return 1;
                        }
                        if (a.age < b.age) {
                          return -1;
                        }
                        return 0;
                      });
                    this.setState({
                      users: users,
                    });
                  });
              }}
            >
              Load users
            </button>
          </div>
        );
      }
    }

В этом коде достаточно сложно разобраться. Код бизнес-логики смешивается с версткой, которую увидит пользователь.

Самый простой способ от этого избавиться: вынести функцию на уровень методов класса:

  

    class MyComponent extends Component {
      fetchUsers() {
        
      }
      render() {
        return (
          <div>
            <ul>
              {this.state.users.map(user => (
                <li>{user.name}</li>
              ))}
            </ul>
            <button onClick={() => this.fetchUsers()}>Load users</button>
          </div>
        );
      }
    }

Здесь мы вынесли бизнес-логику из JSX кода в отдельное поле в нашем классе. Чтобы this был доступен внутри функции, мы определили callback таким образом: `onClick={() => this.fetchUsers()}`

Кроме того, при описании класса мы можем объявить поле как стрелочную функцию:

  

    class MyComponent extends Component {
      fetchUsers = () => {
        
      };
      render() {
        return (
          <div>
            <ul>
              {this.state.users.map(user => (
                <li>{user.name}</li>
              ))}
            </ul>
            <button onClick={this.fetchUsers}>Load users</button>
          </div>
        );
      }
    }

Это позволит нам объявлять колбек как `onClick={this.fetchUsers}`

**В чем разница этих двух способов?**

`onClick={this.fetchUsers}` — Здесь при каждом вызове функции render в props к `button` будет передаваться всегда одна и та же ссылка.

В случае с `onClick={() => this.fetchUsers()}` при каждом вызове функции render JavaScript инициализирует новую функцию `() => this.fetchUsers()` и сетит ее в `onClick` prop. Это значит, что `nextProp.onClick` и `prop.onClick` у `button` в этом случае всегда будут не равны, и даже если компонент будет помечен как чистый, он будет перерендерен.

**Чем это грозит при разработке?**

В большинстве случаев визуально вы не заметите просадки по производительности, потому что Virtual DOM, который будет сгенерирован компонентом, не будет отличаться от предыдущего, и никаких изменений в вашем DOM происходить не будет.

Однако, если вы рендерите большие списки компонентов или таблицы, то на большом объеме данных можно будет заметить "тормоза".

**Почему понимание, как передается функция в колбек, важно?**

Зачастую в twitter или на stackoverflow можно встретить такие советы:

"Если у вас есть проблемы с производительностью React приложения, попробуйте заменить наследование с Component на PureComponent. Также не забывайте о том, что для Component вы всегда можете определить shouldComponentUpdate, чтобы избавиться от ненужных updating циклов".

Если мы определяем компонент как Pure — это означает, что у него уже существует функция `shouldComponentUpdate`, которая делает shallowEqual между props и nextProps.

Передавая каждый раз такому компоненту новую функцию-колбек, мы теряем все преимущества и оптимизации `PureComponent`.

Давайте посмотрим на примере.  
Создадим компонент Input, который также будет выводить информацию, сколько раз он был обновлен:

  

    class Input extends PureComponent {
      renderedCount = 0;
      render() {
        this.renderedCount++;
        return (
          <div>
            <input onChange={this.props.onChange} />
            <p>Input component was rerendered {this.renderedCount} times</p>
          </div>
        );
      }
    }

Создадим два компонента, которые будут рендерить внутри себя Input:

  

    class A extends Component {
      state = { value: '' };
      onChange = e => {
        this.setState({ value: e.target.value });
      };
      render() {
        return (
          <div>
            <Input onChange={this.onChange} />
            <p>The value is: {this.state.value} </p>
          </div>
        );
      }
    }

И второй:

  

    class B extends Component {
      state = { value: '' };
      onChange(e) {
        this.setState({ value: e.target.value });
      }
      render() {
        return (
          <div>
            <Input onChange={e => this.onChange(e)} />
            <p>The value is: {this.state.value} </p>
          </div>
        );
      }
    }

Попробовать пример руками можно здесь: [https://codesandbox.io/s/2vwz6kjjkr](https://codesandbox.io/s/2vwz6kjjkr)  
Этот пример наглядно демонстрирует, как можно потерять все преимущества PureComponent, если передавать в PureComponent каждый раз новую функцию-колбек.

  

### Часть 2\. Использование Event handlers в компонентах-функциях

В новой версии React (16.8) был анонсирован механизм [React hooks](https://reactjs.org/docs/hooks-intro.html), позволяющий писать полноценные функциональные компоненты, с четким lifecycle, которые могут покрыть практически все юзкейсы, которые до текущего момента покрывали только классы.

Модифицируем пример с Input component так, чтобы все компоненты были представлены функцией и работали с React-hooks.

Input должен сохранять внутри себя информацию о том, сколько раз он был изменен. Если в случае с классами мы использовали поле в нашем инстансе, доступ к которому был реализован через this, то в случае с функцией мы не сможем объявить переменную через this.  
React предоставляет хук useRef, с помощью которого можно сохранять ссылку на HtmlElement в DOM дереве, но также он интересен тем, что его можно использовать для обычных данных, которые нужны нашему компоненту:

  

    import React, { useRef } from 'react';
    
    export default function Input({ onChange }) {
      const componentRerenderedTimes = useRef(0);
      componentRerenderedTimes.current++;
    
      return (
        <>
          <input onChange={onChange} />
          <p>Input component was rerendered {componentRerenderedTimes.current} times</p>
        </>
      );
    }

Также нам необходимо, чтобы компонент был "чистым", то есть обновлялся только в случае, если props, которые были переданы в компонент, изменились.  
Для этого существуют разные библиотеки, которые предоставляют HOC, но лучше воспользоваться функцией memo, которая уже встроена в React, так как она работает быстрее и эффективнее:

  

    import React, { useRef, memo } from 'react';
    
    export default memo(function Input({ onChange }) {
      const componentRerenderedTimes = useRef(0);
      componentRerenderedTimes.current++;
    
      return (
        <>
          <input onChange={onChange} />
          <p>Input component was rerendered {componentRerenderedTimes.current} times</p>
        </>
      );
    });

Компонент Input готов, теперь перепишем компоненты A и B.  
В случае с компонентом B это сделать легко:

  

    import React, { useState } from 'react';
    function B() {
      const [value, setValue] = useState('');
    
      return (
        <div>
          <Input onChange={e => setValue(e.target.value)} />
          <p>The value is: {value} </p>
        </div>
      );
    }

Здесь мы воспользовались `useState` hook, который позволяет сохранять и работать с state компонента, в случае если компонент представлен функцией.

Каким образом мы можем закешировать функцию-колбек? Мы не можем вынести ее из компонента, так как в этом случае она будет общая для разных инстансов компонента.  
Для подобных задач в React есть набор кеширующих и мемоизирующих hooks, из которых больше всего нам подойдет `useCallback` [https://reactjs.org/docs/hooks-reference.html](https://reactjs.org/docs/hooks-reference.html)

Добавим в компонент `A` этот hook:

  

    import React, { useState, useCallback } from 'react';
    function A() {
      const [value, setValue] = useState('');
    
      const onChange = useCallback(e => setValue(e.target.value), []);
    
      return (
        <div>
          <Input onChange={onChange} />
          <p>The value is: {value} </p>
        </div>
      );
    }

Мы закешировали функцию, а значит компонент Input не будет обновляться каждый раз.

**Как работает `useCallback` hook?**

Этот hook возвращает закешированную функцию (то есть ссылка не изменяется от рендера к рендеру).  
Помимо функции, которую нужно кешировать, в нее передан второй аргумент — пустой массив.  
Этот массив позволяет передать список полей, при изменении которых необходимо изменить функцию, т.е. вернуть новую ссылку.

Наглядно разницу между обычным способом передачи функции в колбек и `useCallback` можно посмотреть здесь: [https://codesandbox.io/s/0y7wm3pp1w](https://codesandbox.io/s/0y7wm3pp1w)

**Зачем нужен массив?**

Предположим, нам нужно закешировать функцию, которая зависит от какого-то значения через замыкание:

  

    import React, { useCallback } from 'react';
    import ReactDOM from 'react-dom';
    
    import './styles.css';
    
    function App({ a, text }) {
      const onClick = useCallback(e => alert(a), [
        
      ]);
    
      return <button onClick={onClick}>{text}</button>;
    }
    const rootElement = document.getElementById('root');
    ReactDOM.render(<App text={'Click me'} a={1} />, rootElement);

Здесь компонент App зависит от prop `a`. Если запустить пример, то все будет работать корректно до того момента, как мы добавим в конец:

  

    setTimeout(() => ReactDOM.render(<App text={'Next A'} a={2} />, rootElement), 5000);

После срабатывания таймаута при клике на кнопку в alert будет выведено `1`. Так происходит, потому что мы сохранили предыдущую функцию, которая замкнула `a` переменную. И так как `a` — переменная, которая в нашем случае является value type, а value type является неизменяемым, мы получили данную ошибку. Если мы уберем комментарий `/*a*/`, то код будет работать корректно. React при втором рендере проверит, что данные, переданные в массиве отличны и вернет новую функцию.

Попробовать этот пример самому можно здесь: [https://codesandbox.io/s/6vo8jny1ln](https://codesandbox.io/s/6vo8jny1ln)

В React представлено немало функций, которые позволяют мемоизировать данные, например `useRef`, `useCallback` и `useMemo`.  
Если последний нужен для мемоизирования значения функции, и они с `useCallback` достаточно похожи друг на друга, то `useRef` позволяет кешировать не только ссылки на DOM элементы, но и выступать в качестве instance field.

На первый взгляд, его можно использовать для кеширования функций, потому что `useRef` также кеширует данные между отдельными обновлениями компонента.  
Однако, использовать `useRef` для кеширования функций нежелательно. Если наша функция использует замыкание, то в любом рендере замкнутое значение может измениться, а наша кешированная функция будет работать со старым значением. Это означает, что нам нужно будет написать логику обновления фукнций либо просто воспользоваться `useCallback`, в котором это реализовано за счет механизма зависимостей.

[https://codesandbox.io/s/p70pprpvvx](https://codesandbox.io/s/p70pprpvvx) здесь можно посмотреть мемоизацию функций с правильным `useCallback`, с неправильным и с `useRef`.

  

### Часть 3\. Syntetic events

Мы уже разобрались, как использовать event handlers и как корректно работать с замыканиями в колбеках, но в React есть еще одно очень важное отличие при работе с ними:

Обратим внимание: сейчас `Input`, с которым мы работали выше, абсолютно синхронен, но в некоторых случаях может понадобиться, чтобы колбек происходил с задержкой, по паттерну [debounce](https://learn.javascript.ru/task/debounce) или [throttling](https://learn.javascript.ru/task/throttle). Так, debounce, например, очень удобно использовать для инпутов поисковой строки — поиск произойдет только тогда, когда пользователь перестанет вводить символы.

Создадим компонент, который внутри себя вызывает изменение состояния:

  

    function SearchInput() {
      const [value, setValue] = useState('');
    
      const timerHandler = useRef();
    
      return (
        <>
          <input
            defaultValue={value}
            onChange={e => {
              clearTimeout(timerHandler.current);
              timerHandler.current = setTimeout(() => {
                setValue(e.target.value);
              }, 300); // wait, if user is still writing his query
            }}
          />
          <p>Search value is {value}</p>
        </>
      );
    }

Этот код не будет работать. Дело в том, что React внутри себя проксирует события, и в наш onChange колбек попадает так называемый Syntetic Event, который после нашей функции будет "очищен" (поля будут засечены в null). React это делает из соображений производительности, чтобы использовать один объект, а не создавать каждый раз новый.

Если нам нужно взять value, как в данном примере, то достаточно закешировать нужные поля ДО выхода из функции:

  

    function SearchInput() {
      const [value, setValue] = useState('');
    
      const timerHandler = useRef();
    
      return (
        <>
          <input
            defaultValue={value}
            onChange={e => {
              clearTimeout(timerHandler.current);
              const pendingValue = e.target.value; // cached!
              timerHandler.current = setTimeout(() => {
                setValue(pendingValue);
              }, 300); // wait, if user is still writing his query
            }}
          />
          <p>Search value is {value}</p>
        </>
      );
    }

Посмотреть пример можно здесь: [https://codesandbox.io/s/oj6p8opq0z](https://codesandbox.io/s/oj6p8opq0z)

В очень редких случаях появляется необходимость сохранить весь инстанс события. Для этого можно вызывать `event.persist()`, что уберет  
данный инстанс Syntetic event из event-pool реактовских событий.

  

### Заключение:

React event handlers очень удобны, так как они:

  

1.  Автоматизируют подписку и отписку (при unmount компонента);
2.  Упрощают восприятие кода, большую часть подписок легко отследить в JSX коде.

Но вместе с тем, при разработке приложений, можно столкнуться с некоторыми трудностями:

  

1.  Переопределение колбеков в props;
2.  Syntetic events, которые очищаются после выполнения текущей функции.

Переопределение колбеков обычно не заметно, так как не меняется vDOM, но стоит помнить, если вы вводите оптимизации, заменяя компоненты на Pure через наследование от `PureComponent` или используя `memo`, то стоит озаботиться их кешированием, иначе польза от введения PureComponents или memo не будет заметна. Для кеширования вы можете использовать или classProperties (при работе с классом) или `useCallback` hook (при работе с функциями).

Для правильной асинхронной работы, в случае если вам нужны данные из события, также кешируйте нужные вам поля.