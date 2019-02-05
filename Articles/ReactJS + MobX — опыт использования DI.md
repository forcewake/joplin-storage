ReactJS + MobX — опыт использования DI

Мне кажется, настало время поделится подходом для написания ReactJS App,

я не претендую на уникальность.

**Первый абзац можно пропустить**. Я занимаюсь web разработкой уже давно, но последние четыре года я плотно сижу на ReactJS и меня все устраивает, в моей жизни был redux, но примерно два года назад я познакомился с MobX, буквально пару месяцев назад я попытался вернуться на redux, но я не смог, было ощущение что я что-то делаю лишнее, может вообще что то не верное, на эту тему переведено уже много байт на серверах, статья не о крутости одного перед другим, это всего лишь попытка поделится своими наработками, может кому-то реально зайдет этот подход, и так к сути.

**Задачи которые мы будем решать:**

*   подключение di для компонентов
*   серверный рендеринг с асинхронной загрузкой данных

  
Структуру проекта можно посмотреть на [Гитхабе](https://github.com/kalyuk/react-mobx-di). Поэтому я пропущу то, как написать примитивное приложение и в статье будут только основные моменты

Введем такие понятия как: модель данных, сервис, стор.

Заведем простую модель

**TodoModel.ts**

    import { observable, action } from 'mobx';
    export class TodoModel {
      @observable public id: number;
      @observable public text: string = '';
      @observable public isCompleted: boolean = false;
    
      @action
      public set = (key: 'text' | 'isCompleted', value: any): void => {
        this[key] = value;
      };
    }
    

  

то что вы видите экшен set, в моделе это больше исключение, чем хороший тон, обычно в проекте есть базовая модель с примитивными хелперами и от нее просто наследуюсь, в моделях вообще по хорошему не должно быть экшенов

теперь нам нужно научится работать с этой моделью, заведем сервис

**TodoService.ts**

    import { Service, Inject } from 'typedi';
    import { plainToClass, classToClass } from 'class-transformer';
    import { DataStorage } from '../storage/DataStorage';
    import { action } from 'mobx';
    import { TodoModel } from '../models/TodoModel';
    
    const responseMock = {
      items: [
        {
          id: 1,
          isCompleted: false,
          text: 'Item 1'
        },
        {
          id: 2,
          isCompleted: true,
          text: 'Item 2'
        }
      ]
    };
    
    @Service('TodoService')
    export class TodoService {
      @Inject('DataStorage')
      public dataStorage: DataStorage;
    
      @action
      public load = async () => {
        await new Promise(resolve => setTimeout(resolve, 300));
        this.dataStorage.todos = plainToClass(TodoModel, responseMock.items);
      };
    
      @action
      public save(todo: TodoModel): void {
        if (todo.id) {
          const idx = this.dataStorage.todos.findIndex(item => todo.id === item.id);
          this.dataStorage.todos[idx] = classToClass(todo);
        } else {
          const todos = this.dataStorage.todos.slice();
          todo.id = Math.floor(Math.random() * Math.floor(100000));
          todos.push(todo);
          this.dataStorage.todos = todos;
        }
        this.clearTodo();
      }
    
      @action
      public edit(todo: TodoModel): void {
        this.dataStorage.todo = classToClass(todo);
      }
    
      @action
      public clearTodo(): void {
        this.dataStorage.todo = new TodoModel();
      }
    }
    

  

В нашем сервисе есть ссылка на

**DataStorage.ts**

    import { Service } from 'typedi';
    import { observable } from 'mobx';
    import { TodoModel } from '../models/TodoModel';
    
    @Service('DataStorage')
    export class DataStorage {
      @observable public todos: TodoModel[] = [];
    
      @observable public todo: TodoModel = new TodoModel();
    }
    

  

в этом сторе мы будем хранить состояние нашего приложения, таких сторов может быть много, но как показала практика, нет смысла разбивать на много мелких сторов. В сторах также как и в моделях не должно быть экшенов

у нас уже почти все готово, осталось это все подключить к нашему приложению, для этого немного подтюним injector от mobx-react

**DI**

    import { inject } from 'mobx-react';
    
    export function DI(...classNames: string[]) {
      return (target: any) => {
        return inject((props: any) => {
          const data: any = {};
          classNames.forEach(className => {
            const name = className.charAt(0).toLowerCase() + className.slice(1);
            data[name] = props.container.get(className);
          });
          data.container = props.container;
          return data;
        })(target);
      };
    }
    
    

  

и заведем контейнер для нашего DI

**browser.tsx**

    import 'reflect-metadata';
    import * as React from 'react';
    import { hydrate } from 'react-dom';
    import { renderRoutes } from 'react-router-config';
    import { Provider } from 'mobx-react';
    import { BrowserRouter } from 'react-router-dom';
    import { Container } from 'typedi';
    import '../application';
    import { routes } from '../application/route';
    
    hydrate(
      <Provider container={Container}>
        <BrowserRouter>{renderRoutes(routes)}</BrowserRouter>
      </Provider>,
      document.getElementById('root')
    );
    

  

Для браузера у нас всегда один контейнер, а вот для серверного рендера нужно смотреть, лучше для каждого запроса организовать свой контейнер

**server.tsx**

    import * as express from 'express';
    import * as React from 'react';
    import { Container } from 'typedi';
    
    import '../application';
    
    import * as mustacheExpress from 'mustache-express';
    import * as path from 'path';
    import { renderToString } from 'react-dom/server';
    import { StaticRouter } from 'react-router';
    import { Provider } from 'mobx-react';
    import * as uuid from 'uuid';
    import { renderRoutes, matchRoutes } from 'react-router-config';
    import { routes } from '../application/route';
    
    const app = express();
    const ROOT_PATH = process.env.ROOT_PATH;
    
    const currentPath = path.join(ROOT_PATH, 'dist', 'server');
    const publicPath = path.join(ROOT_PATH, 'dist', 'public');
    
    app.engine('html', mustacheExpress());
    app.set('view engine', 'html');
    app.set('views', currentPath + '/views');
    
    app.use(express.static(publicPath));
    
    app.get('/favicon.ico', (req, res) => res.status(500).end());
    
    app.get('*', async (request, response) => {
      const context: any = {};
      const id = uuid.v4();
      const container = Container.of(id);
    
      const branch = matchRoutes(routes, request.url);
    
      const promises = branch.map(({ route, match }: any) => {
        return route.component && route.component.loadData ? route.component.loadData(container, match) : Promise.resolve(null);
      });
    
      await Promise.all(promises);
    
      const markup = renderToString(
        <Provider container={container}>
          <StaticRouter location={request.url} context={context}>
            {renderRoutes(routes)}
          </StaticRouter>
        </Provider>
      );
    
      Container.remove(id);
      if (context.url) {
        return response.redirect(
          context.location.pathname + context.location.search
        );
      }
    
      return response.render('index', { markup });
    });
    
    app.listen(2016, () => {
      
      console.info("application started at 2016 port");
    });
    
    

  

Серверный рендер, это на самом деле тонкая штука, с одной стороны охото пропустить через него все, **но у него всего одна бизнес задача, отдать контент ботам**, таким образом лучше вообще поставить проверку на что то подобное «авторизировался ли пользователь хоть раз на сайте», и скипать серверный рендер с созданием контейнеров на сервере.

ну и теперь к нашим компонентам

**MainRoute.tsx**

    import * as React from 'react';
    import { TodoService } from '../service/TodoService';
    import { observer } from 'mobx-react';
    import { DI } from '../annotation/DI';
    import { DataStorage } from '../storage/DataStorage';
    import { Todo } from '../component/todo';
    import { Form } from '../component/form/Form';
    import { ContainerInstance } from 'typedi';
    
    interface IProps {
      todoService?: TodoService;
      dataStorage?: DataStorage;
    }
    
    @DI('TodoService', 'DataStorage')
    @observer
    export class MainRoute extends React.Component<IProps> {
      public static async loadData(container: ContainerInstance) {
        const todoService: TodoService = container.get('TodoService');
        await todoService.load();
      }
    
      public componentDidMount() {
        this.props.todoService.load();
      }
    
      public render() {
        return (
          <div>
            <Form />
            <ul>
              {this.props.dataStorage.items.map(item => (
                <li key={item.id} ><Todo model={item} /></li>
              ))}
            </ul>
          </div>
        );
      }
    }
    
    

  

тут получается все очень логично и красиво, наша вьюха «render» для отрисовки берет данные из нашего стора, хуки компонента говорят в какой момент времени нам стоит загрузить данные.

**Todo.tsx**

    import * as React from 'react';
    import { TodoModel } from '../../models/TodoModel';
    import { TodoService } from '../../service/TodoService';
    import { DI } from '../../annotation/DI';
    import { observer } from 'mobx-react';
    
    interface IProps {
      model: TodoModel;
      todoService?: TodoService;
    }
    
    @DI('TodoService')
    @observer
    export class Todo extends React.Component<IProps> {
      public render() {
        const { model, todoService } = this.props;
        return (
          <>
            <input
              type='checkbox'
              checked={model.isCompleted}
              onChange={e => model.set('isCompleted', e.target.checked)}
            />
            <h4>{model.text}</h4>
            <button type='button' onClick={() => todoService.edit(model)}>Edit</button>
          </>
        );
      }
    }
    
    

  

**Form.tsx**

    
    import * as React from 'react';
    import { observer } from 'mobx-react';
    import { DI } from '../../annotation/DI';
    import { TodoService } from '../../service';
    import { DataStorage } from '../../storage';
    import { TextField } from '../text-field';
    
    interface IProps {
      todoService?: TodoService;
      dataStorage?: DataStorage;
    }
    @DI('TodoService', 'DataStorage')
    @observer
    export class Form extends React.Component<IProps> {
      public handleSave = (e: any) => {
        e.preventDefault();
        this.props.todoService.save(this.props.dataStorage.todo);
      };
    
      public handleClear = () => {
        this.props.todoService.clearTodo();
      };
      public render() {
        const { dataStorage } = this.props;
    
        return (
          <form onSubmit={this.handleSave}>
            <TextField name='text' model={dataStorage.todo} />
            <button>{dataStorage.todo.id ? 'Save' : 'Create'}</button>
            <button type='button' onClick={this.handleClear}>
              Clear
            </button>
          </form>
        );
      }
    }
    

  

На мой взгляд работать с формами намного удобнее через модели/dtoшки, можно использовать обычные нативные формы, и обновлять модель данных и все кто ее слушают будут обновляться моментально.

Вот как-то так я использую вот эту связку библиотек: react, class-transformer, mobx, typedi

Такой подход мы сейчас используем в проде, это очень большие проекты, с едиными общими компонентами и сервисами

Если этот подход будет интересен расскажу как в этом же ключе мы делаем валидацию моделей перед отправкой на сервер, как обрабатываем серверные ошибки и как мы синхронизируем наше состояние между табами браузера.  
На самом деле все очень бонально: «class-validator», «localStorage + window.addEventListener('storage')»

Спасибо что дочитали :)

[Пример](https://github.com/kalyuk/react-mobx-di)
