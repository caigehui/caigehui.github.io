# 设备巡检列表

介绍如何基于React WxEAP开发一个巡检列表，支持下拉刷新，上拉加载和筛选状态，效果如下

![效果](./实例.PNG)

<a id="1"/>

## 新建工程
通过SVN拷贝模板工程然后删除目录下的`.svn`文件，修改模板工程的名字，即可新建一个工程

> 地址: http://192.168.0.7:8080/svn/wxapp/trunk/code/MobileTemplate

用`Visual Studio Code`打开工程文件夹，`VSCode`会自动创建`.vscode`文件，以后在资源浏览器中，随意打开一个js文件即可打开整个工程


**安装依赖**

在工程目录下运行命令
> npm install

运行程序
> npm start

接下来就是用`smr(service-model-route)三步法`创建一个视图应用

<a id="2"/>

## 创建service

在`services`文件夹下创建tasksList.js
```js
//services/tasksList.js
import { request } from 'react-wxeap-mobile';
import { PAGE_SIZE } from '../constants';

//export一个叫`fetchList`方法, 返回通过`request`请求到的数据
export function fetchList({ selectedIndex, page }) {
  return request(`${API}csmequip/QueryTasks?page=${page}&pageSize=${PAGE_SIZE}&state=${selectedIndex}`);
} 
```


<a id="3"/>

## 创建model
在`models`文件夹下创建`tasksList.js`

```js
import { fetchList } from '../services/tasksList';
import { PAGE_SIZE } from '../constants';


export default {
  namespace: 'tasksList',
  state: {
    selectedIndex: 0,/**当前条件（0-未完成，1-已完成） */
    tasks: []/**当前已经获取到的任务 */
  },
  reducers: {
    save(state, { payload }) {
      return { ...state, ...payload }
    },
  },
  effects: {
    *fetch({ payload: { page, fill } }, { call, put, select }) {
      try {
        /**获取当前条件（未完成，已完成），和已经获取的任务 */
        const { selectedIndex, tasks } = yield select(state => state.tasksList);
        /**根据当前页数和状态获取任务 */
        const { data, err } = yield call(fetchList, { selectedIndex, page })
        if(err) return
        /**组织新的任务列表，如果页数是1，代表刷新列表，否则拼接上原来的数据 */
        let newTasks = (page === 1 ? data.tasks : [...tasks, ...data.tasks]);
        /**保存新的任务 */
        yield put({ type: 'save', payload: { tasks: newTasks }})
        /**填充数据到列表，包括是否加载完成 */
        yield call(fill, newTasks, data.tasks.length < PAGE_SIZE ? true : false)
      } catch (msg) {
        console.warn(msg);
      }
    },
  },
  subscriptions: {
    setup({ dispatch, history }) {
    },
  },
};

```

<a id="4"/>

## 创建route
在`Routes`文件夹下新建一个TasksList.js，路由一般首字母大写
```jsx
import React from 'react';
import {
  ListView,
  routerRedux,
  bind
} from 'react-wxeap';
import { PAGE_SIZE } from '../constants'
import {
  SegmentedControl,
  WingBlank,
  List,
  RefreshControl
} from 'antd-mobile';
const Item = List.Item;
const Brief = Item.Brief;

const styles = {
  container: {
    backgroundColor: '#fff'
  },
  wingBlank: {
    paddingTop: 30,
    paddingBottom: 30
  }
}


class TasksList extends React.Component {

  onFetch = (page, fill) => {
    this.props.dispatch({
      type: 'tasksList/fetch',
      payload: { page, fill }
    })
  }

  onIndexChange = e => {
    this.props.dispatch({ type: 'tasksList/save', payload: { selectedIndex: e.nativeEvent.selectedSegmentIndex } })
    this.listView.reload();
  }

  onItemClick = (rowData) => {
    const { taskId, taskTitle, taskDate, taskEquipNum } = rowData;
    const { dataSource } = this.props;
    let taskIds = "";
    dataSource.map((item, index) => {
      taskIds += item.taskId
      if (index !== dataSource.length - 1) {
        taskIds += ","
      }
    })
    this.props.dispatch(routerRedux.push({
      pathname: '/TaskDetail',
      query: {
        taskId,
        taskTitle,
        taskIds
      }
    }))
  }

  getTaskStateString = (taskState) => {
    switch (taskState) {
      case 0: return '未开始'
      case 1: return '进行中'
      case 2: return '已完成'
      default: return ''
    }
  }

  render() {
    const { dispatch, selectedIndex } = this.props;
    const renderRow = (rowData, sectionID, rowID) =>
      <Item key={rowID} arrow="horizontal" thumb={require('../assets/taskEquip.png')} multipleLine onClick={() => this.onItemClick(rowData)}>
        {rowData.taskTitle} <Brief>{rowData.taskDate + `    ` + this.getTaskStateString(rowData.taskState) + `    ` + rowData.taskEquipNum}</Brief>
      </Item>

    return (
      <div style={styles.container} >
        <WingBlank size="lg" style={styles.wingBlank}>
          <SegmentedControl values={['未完成', '已完成']} selectedIndex={selectedIndex} onChange={this.onIndexChange} />
        </WingBlank>
        <ListView
          ref={o => this.listView = o}
          header="巡检任务列表"
          renderRow={renderRow}
          pageSize={PAGE_SIZE}
          onFetch={this.onFetch} />}
          />}/>
      </div >

    );
  }


}

const mapStateToProps = state => {
  const { list } = state.tasksList;
  return {
    dataSource: list,
    ...state.tasksList
  };
}

export default bind(mapStateToProps)(TasksList);
```

**在入口文件index.js修改配置**
```js
import { MobileApp } from 'react-wxeap';

let routes = [
  {
    name: 'TasksList',
    path: '/',
    model: require('./models/tasksList'),
    component: require('./routes/TasksList')
  },
  {
    name: 'TaskDetail',
    path: '/TaskDetail',
    model: require('./models/taskDetail'),
    component: require('./routes/TaskDetail'),
    createForm: true
  }
];

let options = {
  module: 'wxcsm',
  origin: 'http://192.168.0.92/WxSoft.EAP',
  auth: '/WxLoginIF.aspx?EmpNo=sy&EmpPassword=111111'
}

const app = new MobileApp(routes, options);

app.start();
```

<a id="5"/>

## 调试与打包

使用`chrome`打开调试服务器，打开`chrome`的开发者工具，切换为移动模式，即可调试

运行下面的命令会将应用打包成静态文件，`API`路径会自动转至生产环境

> npm run build

然后在`dist`文件夹下面就会出现打包压缩后的应用，放入`WxEAP`后才能正常运行

