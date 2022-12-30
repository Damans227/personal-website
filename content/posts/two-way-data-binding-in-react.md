---
title: "Two Way Data Binding in React"
date: 2022-12-30T13:57:49-05:00
draft: false
tags:
  - JavaScript
  - React
---

Two data binding means:

- The data we changed in the view has updated the state.
- The data in the state has updated the view.

Following code block implements the two way binding in react:

```javascript
class UserInput extends React.Component{

  state = {
      name:"reactgo"
  }
  handleChange = (e) =>{
    this.setState({
        name: e.target.value
    })
  }

   render(){
    return(
      <div>
       <h1>{this.state.name}</h1>
       <input type="text"
         onChange={this.handleChange}
         value={this.state.name} />
      </div>
      )
   }
}
```

