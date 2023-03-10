# 带有 React 的 Spring 安全登录页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-login-react>

## 1.概观

[React](https://web.archive.org/web/20220707143845/https://reactjs.org/ "https://reactjs.org/") 是由脸书构建的基于组件的 JavaScript 库。使用 React，我们可以轻松地构建复杂的 web 应用程序。在本文中，我们将让 Spring Security 与 React 登录页面协同工作。

我们将利用前面例子中已有的 Spring 安全配置。所以我们将在之前的一篇文章的基础上，用 Spring Security 创建一个[表单登录。](/web/20220707143845/https://www.baeldung.com/spring-security-login)

## 2.设置 React

首先，**使用命令行工具 [create-react-app](https://web.archive.org/web/20220707143845/https://github.com/facebook/create-react-app "https://github.com/facebook/create-react-app") 通过执行命令"`create-react-app react”`创建一个应用程序**。

在`react/package.json`中，我们将有如下配置:

```java
{
    "name": "react",
    "version": "0.1.0",
    "private": true,
    "dependencies": {
        "react": "^16.4.1",
        "react-dom": "^16.4.1",
        "react-scripts": "1.1.4"
    },
    "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "test": "react-scripts test --env=jsdom",
        "eject": "react-scripts eject"
    }
}
```

然后，我们将**使用[前端-Maven-插件](https://web.archive.org/web/20220707143845/https://github.com/eirslett/frontend-maven-plugin "https://github.com/eirslett/frontend-maven-plugin")来帮助用 Maven:** 构建我们的 React 项目

```java
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.6</version>
    <configuration>
        <nodeVersion>v8.11.3</nodeVersion>
        <npmVersion>6.1.0</npmVersion>
        <workingDirectory>src/main/webapp/WEB-INF/view/react</workingDirectory>
    </configuration>
    <executions>
        <execution>
            <id>install node and npm</id>
            <goals>
                <goal>install-node-and-npm</goal>
            </goals>
        </execution>
        <execution>
            <id>npm install</id>
            <goals>
                <goal>npm</goal>
            </goals>
        </execution>
        <execution>
            <id>npm run build</id>
            <goals>
                <goal>npm</goal>
            </goals>
            <configuration>
                <arguments>run build</arguments>
            </configuration>
        </execution>
    </executions>
</plugin>
```

插件的最新版本可以在[这里](https://web.archive.org/web/20220707143845/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.github.eirslett%22%20AND%20a%3A%22frontend-maven-plugin%22 "https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.github.eirslett%22%20AND%20a%3A%22frontend-maven-plugin%22")找到。

当我们运行`mvn compile`时，这个插件将下载`node`和`npm`，安装所有节点模块依赖项，并为我们构建 react 项目。

这里我们需要解释几个配置属性。我们指定了`node`和`npm`的版本，这样插件就知道下载哪个版本了。

我们的 React 登录页面在 Spring 中将作为静态页面，所以我们使用“`src/main/` webapp `/WEB-INF/view/react`”作为`npm`的工作目录。

## 3.Spring 安全配置

在我们深入 React 组件之前，我们更新 Spring 配置以服务 React 应用程序的静态资源:

```java
@EnableWebMvc
@Configuration
public class MvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(
      ResourceHandlerRegistry registry) {

        registry.addResourceHandler("/static/**")
          .addResourceLocations("/WEB-INF/view/react/build/static/");
        registry.addResourceHandler("/*.js")
          .addResourceLocations("/WEB-INF/view/react/build/");
        registry.addResourceHandler("/*.json")
          .addResourceLocations("/WEB-INF/view/react/build/");
        registry.addResourceHandler("/*.ico")
          .addResourceLocations("/WEB-INF/view/react/build/");
        registry.addResourceHandler("/index.html")
          .addResourceLocations("/WEB-INF/view/react/build/index.html");
    }
}
```

**注意，我们添加登录页面`“index.html”`作为静态资源**，而不是动态服务的 JSP。

接下来，我们更新 Spring 安全配置以允许访问这些静态资源。

与我们在[之前的表单登录](https://web.archive.org/web/20220707143845/http://wwwbaeldung.com/spring-security-login "http://wwwbaeldung.com/spring-security-login")文章中使用的`“login.jsp”`不同，这里我们使用`“index.html”`作为我们的`Login`页面:

```java
@Configuration
@EnableWebSecurity
@Profile("!https")
public class SecSecurityConfig 
  extends WebSecurityConfigurerAdapter {

    //...

    @Override
    protected void configure(final HttpSecurity http) 
      throws Exception {
        http.csrf().disable().authorizeRequests()
          //...
          .antMatchers(
            HttpMethod.GET,
            "/index*", "/static/**", "/*.js", "/*.json", "/*.ico")
            .permitAll()
          .anyRequest().authenticated()
          .and()
          .formLogin().loginPage("/index.html")
          .loginProcessingUrl("/perform_login")
          .defaultSuccessUrl("/homepage.html",true)
          .failureUrl("/index.html?error=true")
          //...
    }
}
```

从上面的片段中我们可以看到，当我们将表单数据发送到“`/perform_login`”时，如果凭证匹配成功，Spring 会将我们重定向到“`/homepage.html`”，否则重定向到“`/index.html?error=true`”。

## 4.反应组分

现在让我们在 React 上动手吧。我们将使用组件构建和管理表单登录。

**注意，我们将使用 ES6 (ECMAScript 2015)语法来构建我们的应用程序。**

### 4.1.投入

让我们从支持`react/src/Input.js`中登录表单的`<input />`元素的`Input`组件开始:

```java
import React, { Component } from 'react'
import PropTypes from 'prop-types'

class Input extends Component {
    constructor(props){
        super(props)
        this.state = {
            value: props.value? props.value : '',
            className: props.className? props.className : '',
            error: false
        }
    }

    //...

    render () {
        const {handleError, ...opts} = this.props
        this.handleError = handleError
        return (
          <input {...opts} value={this.state.value}
            onChange={this.inputChange} className={this.state.className} /> 
        )
    }
}

Input.propTypes = {
  name: PropTypes.string,
  placeholder: PropTypes.string,
  type: PropTypes.string,
  className: PropTypes.string,
  value: PropTypes.string,
  handleError: PropTypes.func
}

export default Input
```

如上所述，我们将`<input />`元素包装到一个 React 控制的组件中，以便能够管理其状态并执行字段验证。

React 提供了一种使用 [`PropTypes`](https://web.archive.org/web/20220707143845/https://reactjs.org/docs/typechecking-with-proptypes.html) 来验证类型的方法。具体来说，我们使用`Input.propTypes = {…}`来验证用户传入的属性类型。

**注意`PropType`验证只对开发有效。`PropType`验证是检查我们对组件所做的所有假设是否得到满足。**

拥有它总比被生产中的随机故障吓到要好。

### 4.2.形式

接下来，我们将在文件`Form.js`中构建一个通用表单组件，该组件组合了我们的`Input`组件的多个实例，我们可以基于这些实例来创建登录表单。

在`Form`组件中，我们获取 HTML `<input/>`元素的属性，并从中创建`Input`组件。

然后将`Input`组件和验证错误信息插入`Form:`

```java
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import Input from './Input'

class Form extends Component {

    //...

    render() {
        const inputs = this.props.inputs.map(
          ({name, placeholder, type, value, className}, index) => (
            <Input key={index} name={name} placeholder={placeholder} type={type} value={value}
              className={type==='submit'? className : ''} handleError={this.handleError} />
          )
        )
        const errors = this.renderError()
        return (
            <form {...this.props} onSubmit={this.handleSubmit} ref={fm => {this.form=fm}} >
              {inputs}
              {errors}
            </form>
        )
    }
}

Form.propTypes = {
  name: PropTypes.string,
  action: PropTypes.string,
  method: PropTypes.string,
  inputs: PropTypes.array,
  error: PropTypes.string
}

export default Form
```

现在，让我们看看如何管理字段验证错误和登录错误:

```java
class Form extends Component {

    constructor(props) {
        super(props)
        if(props.error) {
            this.state = {
              failure: 'wrong username or password!',
              errcount: 0
            }
        } else {
            this.state = { errcount: 0 }
        }
    }

    handleError = (field, errmsg) => {
        if(!field) return

        if(errmsg) {
            this.setState((prevState) => ({
                failure: '',
                errcount: prevState.errcount + 1, 
                errmsgs: {...prevState.errmsgs, [field]: errmsg}
            }))
        } else {
            this.setState((prevState) => ({
                failure: '',
                errcount: prevState.errcount===1? 0 : prevState.errcount-1,
                errmsgs: {...prevState.errmsgs, [field]: ''}
            }))
        }
    }

    renderError = () => {
        if(this.state.errcount || this.state.failure) {
            const errmsg = this.state.failure 
              || Object.values(this.state.errmsgs).find(v=>v)
            return <div className="error">{errmsg}</div>
        }
    }

    //...

}
```

在这个代码片段中，我们定义了`handleError`函数来管理表单的错误状态。回想一下，我们还用它进行了`Input`字段验证。**实际上，`handleError()`是在`render()`函数**中作为回调传递给`Input Components`的。

我们使用`renderError()`来构造错误消息元素。注意，`Form's`构造函数消耗了一个`error`属性。此属性指示登录操作是否失败。

然后是表单提交处理程序:

```java
class Form extends Component {

    //...

    handleSubmit = (event) => {
        event.preventDefault()
        if(!this.state.errcount) {
            const data = new FormData(this.form)
            fetch(this.form.action, {
              method: this.form.method,
              body: new URLSearchParams(data)
            })
            .then(v => {
                if(v.redirected) window.location = v.url
            })
            .catch(e => console.warn(e))
        }
    }
}
```

我们将所有表单字段包装到`FormData`中，并使用 [`fetch` API](https://web.archive.org/web/20220707143845/https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 将其发送到服务器。

别忘了我们的登录表单带有一个`successUrl`和`failureUrl`，这意味着无论请求是否成功，响应都需要重定向。

这就是为什么我们需要在响应回调中处理重定向。

### 4.3.表单渲染

既然我们已经设置了所有需要的组件，我们可以继续把它们放在 DOM 中。基本 HTML 结构如下(在`react/public/index.html`下找到):

```java
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- ... -->
  </head>
  <body>

    <div id="root">
      <div id="container"></div>
    </div>

  </body>
</html>
```

最后，我们将表单呈现到`react/src/index.js`中 id 为`container”` 的`<div/>`中:

```java
import React from 'react'
import ReactDOM from 'react-dom'
import './index.css'
import Form from './Form'

const inputs = [{
  name: "username",
  placeholder: "username",
  type: "text"
},{
  name: "password",
  placeholder: "password",
  type: "password"
},{
  type: "submit",
  value: "Submit",
  className: "btn" 
}]

const props = {
  name: 'loginForm',
  method: 'POST',
  action: '/perform_login',
  inputs: inputs
}

const params = new URLSearchParams(window.location.search)

ReactDOM.render(
  <Form {...props} error={params.get('error')} />,
  document.getElementById('container'))
```

所以我们的表单现在包含两个输入字段:`username`和`password`，以及一个提交按钮。

这里我们向`Form`组件传递一个额外的`error`属性，因为我们希望在重定向到失败 URL: `/index.html?error=true`之后处理登录错误。

![form login error](img/dda022c4ca001f23ffd5f1e3bd9bb99d.png)

现在，我们已经使用 React 构建了一个 Spring 安全登录应用程序。我们需要做的最后一件事是运行`mvn compile`。

在这个过程中，Maven 插件将帮助构建我们的 React 应用程序，并在`src/main/webapp/WEB-INF/view/react/build`中收集构建结果。

## 5.结论

在本文中，我们介绍了如何构建一个 React 登录应用程序，并让它与 Spring 安全后端进行交互。一个更复杂的应用程序会涉及到使用 [React Router](https://web.archive.org/web/20220707143845/https://reacttraining.com/react-router/) 或 [Redux](https://web.archive.org/web/20220707143845/https://redux.js.org/) 的状态转换和路由，但这超出了本文的范围。

和往常一样，完整的实现可以在 GitHub 上找到[。要在本地运行它，在项目根文件夹中执行`mvn jetty:run`，然后我们可以在`http://localhost:8080`访问 React 登录页面。](https://web.archive.org/web/20220707143845/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-react)