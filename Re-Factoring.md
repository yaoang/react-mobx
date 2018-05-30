# 代码重构之多状态处理 code-refactoring

> Author: Ang

在代码里面，经常会有很多状态判断，用以区分不同的情况，这个时候，我们经常会写很多的if-else。举个栗子：

```javascript
function validate() {
    if(!this.state.password) {
        this.setState({
            error: {
                message: 'Password not inputed!',
            }
        })
        return false
    } else if(!this.state.repeatPassword) {
        this.setState({
            error: {
                message: 'Repeat Password not inputed!',
            }
        })
        return false
    } else if(this.state.repeatPassword !== this.state.password) {
        this.setState({
            error: {
                message: 'Repeat Password not match the password!',
            }
        })
        return false
    }
    
    return true
}
```

这个时候，如果有很多的状态判断，结果就会变成如下情况：

```javascript
function validate() {
    if(condition1) {
       // ...
        return false
    } else if(condition2) {
       // ...
        return false
    } else if(condition3) {
       // ...
        return false
    } else if(condition4) {
       // ...
        return false
    }
    // ...
    else if(conditionN) {
        // ...
        return false
    }
    return true
}
```

一旦条件非常多，这种代码是非常难以理解的。这是一种非常懒得的办法，一个情况就加一个if。到了后期，谁都不明白你写的是什么，包括你自己。很多人都不知道为什么要加这个if，但是不加就会出bug。所以就不停往后面加else if。

假设我们已经写好了上面的代码，那么我们一步步来重构这段代码。

首先，把各种情况分类。

* Password
* Repeat Password
* Compare the passwords

建立一个对象

```javascript
const PasswordValidationPoliciers = {
    "Password": {
        dealWithError: ()=>{
            this.setState({
                error: {
                    message: 'Error 1',
                }
            })  
        },
    },
    "RepeatPassword": {
        dealWithError: ()=>{
            this.setState({
                error: {
                    message: 'Error 2',
                }
            })  
        },
    },
    "ComparePassword": {
        dealWithError: ()=>{
            this.setState({
                error: {
                    message: 'Error 3',
                }
            })  
        },
    },
}
```

那如何找到对应的处理对象？

加个函数来查找：

```javascript
import {trim, equal} from {lodash}

const PasswordValidationPoliciers = {
	getPasswordValidationPolicy: () => {
        if(!trim(this.state.password)) {
            return PasswordValidationPoliciers['Password']
        }
        if(!trim(this.state.repeatPassword)) {
            return PasswordValidationPoliciers['RepeatPassword']
        }
        if(!equal(this.state.repeatPassword, this.state.password)) {
            return PasswordValidationPoliciers['ComparePassword']
        }
	},
    "Password": {
        dealWithError: ()=>{
            // ...
        },
    },
    "RepeatPassword": {
        dealWithError: ()=>{
            // ...
        },
    },
    "ComparePassword": {
        dealWithError: ()=>{
            // ...
        },
    },
}

function validate(){
    const passwordValidationPolicy = PasswordValidationPoliciers.
    	getPasswordValidationPolicy()
    if(passwordValidationPolicy) {
        return passwordValidationPolicy.dealWithError()
    }
    
    return true
}
```

