## React Hooks

- 无状态组件 Function
  - 无法使用生命周期函数
  - 无法使用state
- react Hooks 诞生就是为了使得无状态组件里面可以使用生命周期还有state

#### 例子讲解

- hooks与无状态组件的对比

  - ```
    
    class Example extends React.Component {
        state = {
            count: 0;
        }
        render() {
            return(
            	<div>
                    <p>你点击了{this.state.count}次</p>
                    <button onClick={() => this.setState(this.state.count + 1)}> 点我 </button>
                </div>
            )
        }
    }
    
    
    import { useState } from 'react';     //导入userState的hook
    
    function Example() {
      // 声明一个名为“count”的新状态变量
      const [count, setCount] = useState(0);        //运用了数组结构
      return (
          <div>
              <p>你点击了{count}次</p>
              <button onClick={() => setCount(count + 1)}> 点我 </button>
          </div>
      );
    }
    ```

- 跟state的不同是

  - 我们使用setState的时候， 是更新我们修改的那一个

    - ```
      state = {
          name: "joker",
          age: 18
      }
      this.setstate({
          age: 20
      }, () => { console.log(this.state)})
      
      {
          name: "joker",
          age: 20
      }
      ```

  - 使用useState   **替换老的状态返回新的状态**

    - ```
      function Example() {
          const [ person, setPerson ] = useState({name: 'joker', age: 18});
          
          return(
           	<p> {person} times</p>   
          	<button onClick={() =>  setPerson({age: 20})} > 点我 </button>
          )
      }
      
      //一开始显示  {name: 'joker', age: 18}
      //点击之后变为  {age: 20}
      ```

    - 

- 添加多个状态的理解

  - ```
    
    function Example() {
      // 声明一个名为“count”的新状态变量
      const [count, setCount] = useState(0);
      const [name, setName] = useState("name");
      const [person, setPerson] = useState({name: "joker", age: 18})
      
      return (
          <div> </div>
      );
    }
    ```

  - useState保证多个状态的独立， 我们设置状态的时候都是调用useState， 每一次值传入一个key值，怎么保证一一对应

    - react规定必须把hooks写在函数的最外层

    - ```
      const show = true
      function Example() {
      
        const [count, setCount] = useState(0);
        
        if(show){
       	 const [name, setName] = useState("name");
           show = false
        }
        
        const [person, setPerson] = useState({name: "joker", age: 18})
        
        return (
            <div> </div>
        );
      }
      
      
      
      //第一次执行
      	useState(0)  将count初始化为0
      	useState("name")  将name初始化为name
      	useState({name: "joker", age: 18})  将person初始化为{name: "joker", age: 18}
      	
      //第二次执行
      	useState(0)  将count初始化为0
      	//   useState("name") 没有执行
      	useState({name: "joker", age: 18})  读取到name， 报错
      	
      ```

### 副作用钩子 useEffect     **每一次组件的更新都会调用， ==  componentDidMount 和 componentDidUpdate**

- ```
  import { useState, useEffect } from 'react';
  
  function Example() {
    const [count, setCount] = useState(0);
  
    // 与 componentDidMount 和 componentDidUpdate 类似:
    useEffect(() => {
      document.title = `You clicked ${count} times`;
    });
  
    return (
      <div>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}> Click me </button>
      </div>
    );
  }
  ```

- 每一次更新都会执行， 选择特定的条件下更新

  - ```
    import { useState, useEffect } from 'react';
    
    function Example() {
      const [count, setCount] = useState(0);
    
      useEffect(() => {
        document.title = `You clicked ${count} times`;
      }, [count]);    //表示在count变化的时候执行
    
      useEffect(() => {
        document.title = `You clicked ${count} times`;
      }, []);    //相当于compomentDidMount
    
      return (
        <div>
          <p>You clicked {count} times</p>
          <button onClick={() => setCount(count + 1)}> Click me </button>
         </div>
      );
    }
    ```

- 规则

  - 不能再 **if语句， 循环**里面使用useEffect

    - ```
      //错误例子
      if(true) {
          useEffect(() => {})
      }
      
      //正确
      useEffect(() => {
      	if（true){
              //...
      	}
      })
      ```