###在使用react的，不可避免的组件间的消息传递

####1：父组件向子组件传值（通常的是父组件向子组件传递props,子组件得到props后进行对应的处理）
App.js(父组件)

      import React,{ Component } from "react";
      import Sub from "./SubComponent.js";
      export deafault class APP extents Component{
      reader(){
      return (`<div>
      <Sub title = "今年过节不收礼" />
      </div>`)
      }
      }
   
SubComponents.js(子组件)

        import React,{ Component } from "react";
        const Sub=(props)=>{
         return(
          <h1>{props.title}</h1>
       )
         }
####2：子组件向父组件传值( 父组件将一个函数作为 props 传递给子组件，子组件调用该回调函数，便可以向父组件通信。)
SubComponents.js

     import React from 'react'
     const Sub=(props)=>{
      const cb=(msg)=>{
       return()=>{
          prop.callback(msg)
       }
     }
      return(
        <div>
            <button onClick = { cb("我们通信把") }>点击我</button>
        </div>
    )
      }
    export default Sub;
App.js  
    import Sub from "./SubComponent.js";
    import React,{ Component } from "react";
    callback(msg){console.log(msg)}
    render(){
     return(
     <div>
            <Sub callback={this.callback.bind(this)}> </Sub>
        </div>
      )
    }
####3：跨级组件之间的通信
####所谓跨级组件通信，就是父组件向子组件的子组件通信，向更深层的子组件通信。跨级组件通信可以采用下面两种方式：

      1: 中间组件层层传递 props
      2: 使用 context 对象

    对于第一种方式，如果父组件结构较深，那么中间的每一层组件都要去传递 props，增加了复杂度，并且这些 props 并不是这些中间组件自己所需要的。不过这种方式也是可行的，当组件层次在三 层以内可以采用这种方式，当组件嵌套过深时，采用这种方式就需 要斟酌了。
	使用 context 是另一种可行的方式，context 相当于一个全局变量，是一个大容器，我们可以把要通信的内容放在这个容器中，这样一来，不管嵌套有多深，都可以随意取用。
	使用 context 也很简单，需要满足两个条件：
	
	1:上级组件要声明自己支持 context，并提供一个函数来返回相应的 context 对象
	2:子组件要声明自己需要使用 context


APP.JS

      import React, { Component } from 'react';
	  import PropTypes from "prop-types";
	  import Sub from "./Sub";
	  import "./App.css";
     export default calss App extents Component{
    //父组件声明自己支持content
     static childContentTypes={
       color:Proptypes.string
       callback:propsTypes.func
     }
    //父组件提供一个函数，用来返回相应的content 对象
    getchildContent(){
     return{
      color:"red"
      callback:this.callback.bind(this)
     }
    }
    callback(msg){
        console.log(msg)
    }
    render(){
        return(
            <div>
                <Sub></Sub>
            </div>
        );
    }
     }

Sub.js

	import React from "react";
	import SubSub from "./SubSub";
	
	const Sub = (props) =>{
	    return(
	        <div>
	            <SubSub />
	        </div>
	    );
	}
	
	export default Sub;
SubSub.js：

	import React,{ Component } from "react";
	import PropTypes from "prop-types";
	
	export default class SubSub extends Component{
	    // 子组件声明自己需要使用 context
	    static contextTypes = {
	        color:PropTypes.string,
	        callback:PropTypes.func,
	    }
	    render(){
	        const style = { color:this.context.color }
	        const cb = (msg) => {
	            return () => {
	                this.context.callback(msg);
	            }
	        }
	        return(
	            <div style = { style }>
	                SUBSUB
	                <button onClick = { cb("我胡汉三又回来了！") }>点击我</button>
	            </div>
	        );
	    }
	}



			如果是父组件向子组件单向通信，可以使用变量，如果子组件想向父组件通信，同样可以由父组件提供一个回调函数，供子组件调用，回传参数。
			在使用 context 时，有两点需要注意：
			
			父组件需要声明自己支持 context，并提供 context 中属性的 PropTypes
			子组件需要声明自己需要使用 context，并提供其需要使用的 context 属性的 PropTypes
			父组件需提供一个 getChildContext 函数，以返回一个初始的 context 对象



     如果组件中使用构造函数（constructor），还需要在构造函数中传入第二个参数 context，并在 super 调用父类构造函数是传入 context，否则会造成组件中无法使用 context。
			...
			constructor(props,context){
			  super(props,context);
			}



	import React, { Component } from 'react';
	import PropTypes from "prop-types";
	import Sub from "./Sub";
	import "./App.css";
	
	export default class App extends Component{
    constructor(props) {
        super(props);
        this.state = {
            color:"red"
        };
    }
    // 父组件声明自己支持 context
    static childContextTypes = {
        color:PropTypes.string,
        callback:PropTypes.func,
    }

    // 父组件提供一个函数，用来返回相应的 context 对象
    getChildContext(){
        return{
            color:this.state.color,
            callback:this.callback.bind(this)
        }
    }

    // 在此回调中修改父组件的 state
    callback(color){
        this.setState({
            color,
        })
    }

    render(){
        return(
            <div>
                <Sub></Sub>
            </div>
        );
    }
} 






在子组件的 cb 方法中，传入相应的颜色参数，就可以改变 context 对象了，进而影响到子组件：

	...
	return(
	    <div style = { style }>
	        SUBSUB
	        <button onClick = { cb("blue") }>点击我</button>
	    </div>
	);
	...
	context 同样可以应在无状态组件上，只需将 context 作为第二个参数传入：
		
		
		import React,{ Component } from "react";
		import PropTypes from "prop-types";
		const SubSub = (props,context) => {
	    const style = { color:context.color }
	    const cb = (msg) => {
	        return () => {
	            context.callback(msg);
	        }
	    }
	
	    return(
	        <div style = { style }>
	            SUBSUB
	            <button onClick = { cb("我胡汉三又回来了！") }>点击我</button>
	        </div>
	    );
	}
	
	SubSub.contextTypes = {
	    color:PropTypes.string,
	    callback:PropTypes.func,
	}
	export default SubSub;
				

####4：非嵌套组件之间通信（非嵌套组件，就是没有任何包含关系的组件，包括兄弟组件以及不在同一个父级中的非兄弟组件。对于非嵌套组件，可以采用下面两种方式：利用二者共同父组件的 context 对象进行通信使用自定义事件的方式）

App.js

	 	  1：我们需要使用一个 events 包：（npm install events --save）
		  2:新建一个 ev.js，引入 events 包，并向外提供一个事件2对象，供通信时使用：
		
		import { EventEmitter } from "events";
		export default new EventEmitter();
		
		
		import React, { Component } from 'react';
		
		import Foo from "./Foo";
		import Boo from "./Boo";
		
		import "./App.css";
		
		export default class App extends Component{
		    render(){
		        return(
		            <div>
		                <Foo />
		                <Boo />
		            </div>
		        );
		    }
		} 



Foo.js：
		import React,{ Component } from "react";
		import emitter from "./ev"
		export default class Foo extends Component{
		    constructor(props) {
		        super(props);
		        this.state = {
		            msg:null,
		        };
		    }
		    componentDidMount(){
		        // 声明一个自定义事件
		        // 在组件装载完成以后
		        this.eventEmitter = emitter.addListener("callMe",(msg)=>{
		            this.setState({
		                msg
		            })
		        });
		    }
		    // 组件销毁前移除事件监听
		    componentWillUnmount(){
		        emitter.removeListener(this.eventEmitter);
		    }
		    render(){
		        return(
		            <div>
		                { this.state.msg }
		                我是非嵌套 1 号
		            </div>
		        );
		    }
			}
Boo.js：
		import React,{ Component } from "react";
		import emitter from "./ev"
		
		export default class Boo extends Component{
		    render(){
		        const cb = (msg) => {
		            return () => {
		                // 触发自定义事件
		                emitter.emit("callMe","Hello")
		            }
		        }
		        return(
		            <div>
		                我是非嵌套 2 号
		                <button onClick = { cb("blue") }>点击我</button>
		            </div>
		        );
		    }
		}
			



