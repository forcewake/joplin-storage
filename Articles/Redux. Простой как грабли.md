Redux. Простой как грабли

Мне уже доводилось заглядывать в репозиторий библиотеки [redux](https://github.com/reduxjs/redux), но откуда-то появилась мысль углубиться в его реализацию. Своим в некотором роде шокирующим или даже разочаровывающим открытием я хотел бы поделиться с сообществом.

**TL;DR:** базовая логика redux помещается в 7 строк JS кода

О redux вкратце (вольный перевод заголовка на гитхабе):

> Redux — библиотека управления состоянием для приложений, написанных на JavaScript  
> Она помогает писать приложения, которые ведут себя стабильно/предсказуемо, работают на разных окружениях (клиент/сервер/нативный код) и легко тестируемы

Я склонировал репозиторий redux ([https://github.com/reduxjs/redux](https://github.com/reduxjs/redux)), открыл в редакторе папку с исходниками (игнорируя

docs

,

examples

и прочее) и взялся за

~ножницы~

клавишу Delete:

*   **Удалил все комментарии из кода**  
    Каждый метод библиотеки задокументирован с помощью JSDoc весьма подробно  
    
*   **Убрал валидацию и логирование ошибок**  
    В каждом методе жёстко контролируются входные параметры с выведением очень приятных глазу подробных комментариев в консоль  
    
*   **Убрал методы bindActionCreators, subscribe, replaceReducer и observable**  
    … потому что мог. Ну или потому что поленился писать для них примеры. Но без корнер-кейсов они ещё менее интересны, чем то, что ждёт вас впереди  
    

А теперь давайте разберём то, что осталось  

* * *

  

### Пишем redux за 7 строк

Весь базовый функционал redux умещается в малюсенький файлик, ради которого вряд ли кто-нибудь будет создавать github репозиторий :)

    function createStore(reducer, initialState) {
        let state = initialState
        return {
            dispatch: action => { state = reducer(state, action) },
            getState: () => state,
        }
    }
    

Всё. Да, серьёзно, **ВСЁ**  
Так устроен redux. 18 страниц вакансий на HeadHunter с поисковым запросом «redux» — люди, которые надеются, что вы разберетесь в 7 строках кода. Всё остальное — синтаксический сахар.

С этими 7 строками уже можно писать TodoApp. Или что угодно. Но мы быстренько перепишем [TodoApp](https://redux.js.org/basics/example) из документации к redux

    
    function todosReducer(state, action) {
      switch (action.type) {
        case 'ADD_TODO':
          return [
            ...state,
            {
              id: action.id,
              text: action.text,
              completed: false
            }
          ]
        case 'TOGGLE_TODO':
          return state.map(todo => {
            if (todo.id === action.id) {
              return { ...todo, completed: !todo.completed }
            }
            return todo
          })
        default:
          return state
      }
    }
    
    const initialTodos = []
    
    const store = createStore(todosReducer, initialTodos)
    
    
    store.dispatch({
      type: 'ADD_TODO',
      id: 1,
      text: 'Понять насколько redux прост'
    })
    
    store.getState() 
    
    
    store.dispatch({
      type: 'TOGGLE_TODO',
      id: 1
    })
    
    store.getState() 
    
    

Уже на этом этапе я думал бросить микрофон со сцены и уйти, но _show must go on_  
Давайте посмотрим, как устроен метод

### combineReducers

Это метод, который позволяет вместо того, чтобы создавать один огромный reducer для всего состояния приложения сразу, разбивать его на отдельные модули  
Используется он так:

    
    
    function counterReducer(state, action) {
      if (action.type === 'ADD') {
        return state + 1
      } else {
        return state
      }
    }
    
    const reducer = combineReducers({
      todoState: todoReducer,
      counterState: counterReducer
    })
    
    const initialState = {
      todoState: [],
      counterState: 0,
    }
    
    const store = createStore(reducer, initialState)
    

Дальше использовать этот store можно так же, как предыдущий  
Разница моего примера и описанного в той же документации к [TodoApp](https://redux.js.org/basics/example) довольно забавная

В документации используют модный синтаксис из ES6 (7/8/∞)

    const reducer = combineReducers({ todos, counter })
    

и соответственно переименовывают todoReducer в todos и counterReducer в counter. И многие в своём коде делают то же самое. В итоге разницы нет, но для человека, знакомящегося с redux, с первого раза эта штука выглядит магией, потому что ключ части состояния _(state.todos)_ соответствует функции, названной также только по желанию разработчика _(function todos(){})_

Если бы нам нужно было написать такой функционал на нашем micro-redux, мы бы сделали так

    function reducer(state, action) {
      return {
        todoState: todoReducer(state, action),
        counterState: counterReducer(state, action),
      }
    }
    

Этот код плохо масштабируется. Если у нас 2 «под-состояния», нам нужно дважды написать **(state, action)**, а хорошие программисты так не делают_, правда?_

> В следующем примере от вас ожидается, что вы не испугаетесь метода [Object.entries](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/entries) и [Деструктуризации параметров функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

Однако реализация метода combineReducers довольно простая (напоминаю, это если убрать валидацию и вывод ошибок) и самую малость отрефакторить на свой вкус

    function combineReducers(reducersMap) {
      return function combinationReducer(state, action) {
        const nextState = {}
        Object.entries(reducersMap).forEach(([key, reducer]) => {
          nextState[key] = reducer(state[key], action)
        })
        return nextState
      }
    }
    

Мы добавили к нашему детёнышу redux ещё 9 строк и массу удобства.  
Перейдём к ещё одной важной фиче, которая кажется слишком сложной, чтобы пройти мимо неё

### applyMiddleware

middleware в разрезе redux — это какая-то штука, которая слушает все dispatch и при определенных условиях делает _что-то_. Логирует, проигрывает звуки, делает запросы к серверу,… — _что-то_  
В оригинальном коде middleware передаются как дополнительные параметры в createStore, но если не жалеть лишнюю строчку кода, то использование этого функционала выглядит так:

    const createStoreWithMiddleware = applyMiddleware(someMiddleware)(createStore)
    const store = createStoreWithMiddleware(reducer, initialState)
    

При этом реализация метода applyMiddleware, когда ты потратишь 10 минут на ковыряние в чужом коде, сводится к очень простой вещи: createStore возвращает объект с полем «dispatch». dispatch, как мы помним (не помним) из первого листинга кода, — это функция, которая всего лишь применяет редюсер к нашему текущему состоянию (newState = reducer(state, action)).  
Так вот applyMiddleware не более чем переопределяет метод **dispatch**, добавляя перед (или после) обновлением состояния какую-то пользовательскую логику.  
Возьмём, например, самый популярный middleware от создателей redux — [redux-thunk](https://github.com/reduxjs/redux-thunk)  
Его смысл сводится к тому, что можно делать не только

    store.dispatch({type: 'SOME_ACTION_TYPE', some_useful_data: 1 })

но и передавать в store.dispatch сложные функции

    function someStrangeAction() {
      return async function(dispatch, getState) {
        if(getState().counterState % 2) {
           dispatch({
             type: 'ADD',
           })
        }
        await new Promise(resolve => setTimeout(resolve, 1000))
        dispatch({
          type: 'TOGGLE_TODO',
          id: 1
        })
      }
    }
    

И теперь, когда мы выполним команду

    dispatch(someStrangeAction())
    

то:  
— если значение store.getState().counterState не делится на 2, оно увеличится на 1  
— через секунду после вызова нашего метода, todo с id=1 переключит completed true на false или наоборот

Итак, я залез в репозиторий redux-thunk, и сделал то же самое что и с redux — удалил комментарии и параметры, которые расширяют базовый функционал, но не изменяют основной

Послучилось следующее

    const thunk = store => dispatch => action => {
      if (typeof action === 'function') {
        return action(store.dispatch, store.getState)
      }
      return dispatch(action)
    }
    

я понимаю, что конструкция

const thunk = store => dispatch => action

выглядит жутковато, но её тоже просто нужно вызвать пару раз с произвольными параметрами и вы осознаете, что всё не так страшно, это просто функция, возвращающая функцию, возвращающую функцию (ладно, согласен, страшно)

Напомню, оригинальный метод **createStore** выглядел так

    function createStore(reducer, initialState) {
        let state = initialState
        return {
            dispatch: action => { state = reducer(state, action) },
            getState: () => state,
        }
    }
    

То есть он принимал атрибуты (reducer, initialState) и возвращал объект с ключами { dispatch, getState }

Оказалось, что реализовать метод **applyMiddleware** проще, чем понять, как он работает  
Мы берём уже реализованный метод **createStore** и переопределяем его возвращаемое значение

    function applyMiddleware(middleware) {
      return function createStoreWithMiddleware(createStore) {
        return (reducer, state) => {
          const store = createStore(reducer, state)
    
          return {
            dispatch: action => middleware(store)(store.dispatch)(action),
            getState: store.getState,
          }
        }
      }
    }

  

### Вывод

Под капотом redux содержатся очень простые логические операции. Операции на уровне «Если бензин в цилиндре загорается, давление увеличивается». А вот то, сможете ли вы построить на этих понятиях болид Формулы 1 — уже решайте сами  
P.S.  
Для добавления в мой «micro-redux» упрощённого метода store.subscribe потребовалось 8 строк кода. А вам?