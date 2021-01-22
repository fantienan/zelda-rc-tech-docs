# 重构forestar@v5.0

### forestar api

- 去掉 `require` 语法
- 去掉 `jquery` 语法
- 使用 `ES6+`重构语法

### react forestar ui

- 替换原来`forestar@v5.0 api`中的`jquery ui`
- 只服务与`forestar api`

### react forestar provider

- 存储地图及地图工具的实例供项目使用

### example

#### 项目中使用 react components、forestar provider：

> react 根节点注入 forestar provider，这样在其他页面才能拿到 forestar 实例们。

```tsx
import ReactDOM from "react-dom";
import { ForestarProvider } from "zelda-rc";

ReactDOM.render(
  <ConfigProvider locale={zhCN}>
    <ForestarProvider>
      <App />
    </ForestarProvider>
  </ConfigProvider>,
  document.getElementById("root")
);
```

> 获取 forestar 实例们。

```tsx
import React, { useContext } from "react";
// 平台组件库
import { Map, Compartment, ForestarContext } from "zelda-rc";

const Example: React.FC = () => {
  // 拿到地图实例和工具实例们
  const context = useContext(ForestarContext);
  return (
    <div>
      <Map />
      <Compartment />
    </div>
  );
};
```

#### react components

> Map 组件

```tsx
import { ForestarContext } from "zelda-rc"

export const store = new Store({
  forestarMap: new ForestarMap()
  ...
})
const Map: React.FC<IMap> = (props) => {
  ...
  const context = useContext(ForestarContext);
  // 地图初始化
  const init = () => {...};
  useEffect(() => {
    init();
    // 向forestar provider挂载实例
    context.setForestarMap(store.forestarMap);
    ...
  }, []);
  ...
  return ...
};
export default Map
```

> Compartment 组件

```tsx
import { ForestarContext } from "zelda-rc"

const Compartment: FC<ICompartment> = (props) => {
	...
	const context = useContext(ForestarContext)
  ...
  const drawFinishHandler = () => {
	}
	useEffect(() => {
		...
		// 初始化addTool实例并注册回调事件
		context.setTool().useDrawFinishHandler(drawFinishHandler)
	}, [])
	...
	return ...
}
export default Compartment
```

#### forestar privider

```tsx
/**
 * 平台向业务层提供获取forestar api各个实例的store
 * **/
import React, { createContext } from "react";
...

export interface IStore {
  addTool?: any; // 区划工具条实例
  forestarMap?: IForestarMap;
  distLocation?: IDistLocation;
  pointQuery?: IPointQuery;
  setAddTool(): void;
  setForestarMap(value: IForestarMap): void;
  setDistLocation(value: IDistLocation): void;
  setPointQuery(value: IPointQuery): void;
}
const DRWA_FINISH_EVENT_NAME = "drawFinishHandler"
class Store implements IStore {
  private register: Map<string, (params: any) => void> = new Map()
  forestarMap: IStore["forestarMap"] = undefined;
  addTool?: any = undefined;
  ...
  // 挂载工具实例
  setTool() {
    // 利用微任务队列处理初始化时实例挂载顺序
    let runner = new Promise(resolve => {
        const chain = Promise.resolve()
        chain.then(this.setAddTool.bind(this))
        ...
        resolve(true)
    })
    runner.then(() => console.log("初始化addTool完毕"))
    return this
  }
  // 挂载地图实例
  setForestarMap(value: IForestarMap) {
    this.forestarMap = value;
  }
  ...
  private setAddTool() {
    this.addTool = new AddTool({
      option: {
        drawFinishHandler: this.drawFinishHandler.bind(this),
      },
      map: this.forestarMap
    })
    return this
  }
  // 区划绘图完毕的回调
  private drawFinishHandler(params: IDrawFinishParams) {
    const cb = this.register.get(DRWA_FINISH_EVENT_NAME)
    typeof cb === "function" && cb(params as IDrawFinishParams)
  }
  useDrawFinishHandler(cb: () => void) {
    this.register.set(DRWA_FINISH_EVENT_NAME, cb)
  }
  ...
}

const store = new Store();
export const ForestarContext = createContext(store);
const Provider: React.FC = (props) => {
  return (
    <ForestarContext.Provider value={store}>
      {props.children}
    </ForestarContext.Provider>
  );
};
export default Provider;
```

> 流程图

![重构forestar@v5.0架构图](images/重构forestar@v5.0.jpg)
