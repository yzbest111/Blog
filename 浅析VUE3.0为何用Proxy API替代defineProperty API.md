## 浅析VUE3.0为何用Proxy API替代defineProperty API

众所周知，VUE3.0用proxy代替了Object.defineProperty来实现数据的响应式。接下来我们来使用这两个API，实现简单的数据响应，剖析其优劣。

#### **1. Object.defineProperty**

> Object.defineProperty可以直接在一个对象上定义一个新的属性，或者修改该对象上的某个属性，并返回一个对象

##### 为什么能实现响应式？
  * get函数
  当访问某属性时，会触发此函数的调用。执行时不传入任何参数，该函数的返回值会作为该属性的值。
  
  * set函数
  当某属性值被修改时，会调用此函数。该函数接受一个参数，即被赋予的新值。
  
  下面通过代码展示，利用Object.defineProperty，简单实现一个响应式函数：defineReactive
  
  ```javascript
    function defineReactive(obj, key, val) {
      Object.defineProperty(obj, key, {
        get() {
          console.log(`get ${key}: ${val}`)
          return val
        },
        set(newVal) {
          if (newVal !== val) {
            console.log(`set the new value: ${val}`)
            val = newVal
          }
            
        }
      })
    }
  ```
  
  调用defineReactive：
  ```javascript
    const obj = {}
    defineReactive(obj, 'name', '')
    obj.name = 'yuzheng'
    console.log(obj.name)
  ```
  
  若对象存在多个key，则需要进行遍历
  ```javascript
    function observe(obj) {
      if (typeof obj !== 'object' || obj == null) return
      Object.keys(obj).forEach((key) => {
        defineReactive(obj, key, obj[key])
      })
    }
  ```
  
  若存在嵌套对象的情况，还需要在defineReactive中进行递归
  ```javascript
    function defineReactive(obj, key, val) {
      observe(val)
      Object.defineProperty(obj, key, {
        get() {
          console.log(`get ${key}: ${val}`)
          return val
        },
        set(newVal) {
          if (newVal !== val) {
            console.log(`set the new value: ${val}`)
            observe(newVal) // 新值是对象的情况
            val = newVal
          }
            
        }
      })
    }
  ```
  
  如果对一个对象进行属性的删除和新增的情况时，则无法被Object.defineProperty劫持到
  ```javascript
    const obj = {
      foo: 'foo',
      bar: 'bar'
    }
    observe(obj)
    delete obj.foo // 无法被劫持到
    obj.name = 'yuzheng' // 无法被劫持到
  ```
  
  同样，对一个数组进行操作的时候，依然无法被劫持到
  ```javascript
    const arr = [1, 2, 3]
    arr.forEach((item, index) => {
      defineReactive(arr, index, val)
    })
    
    arr.push(4) // 无法被劫持到
    arr.unshift() // 无法被劫持到
  ```
  
  综上所述，在vue2中使用Object.defineProperty对某些情况无法进行数据劫持实现响应式，其不足之处如下：
  * 检测不到对象的删除和添加
  * 检测不到数组的行为
  * 需要对每个属性进行遍历监听，若存在嵌套的情况，需要进行深层监听，损伤性能
  
  #### **2. Proxy**
  > Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。
  
  通过Proxy实现数据响应式
  ```javascript
    function observe(obj) {
      if (typeof obj !== 'object' || obj == null) return
      
      // 通过new Proxy构造一个代理对象并返回，其形式为new Proxy(obj, handler)
      const observed = new Proxy(obj, {
        get(target, key, receiver) {
          const result = Reflect.get(target, key, receiver)  // Reflect作为操作对象统一接口，提供了诸多方法
          console.log(`获取属性${key}`)
          return result
        },
        set(target, key, val, receiver) {
          conset result = Reflect.set(target, key, val, receiver)
          console.log(`设置新属性${key}: ${val}`)
          return result
        },
        deleteProperty(target, key) {
          const result = Reflect.deleteProperty(target, key)
          console.log(`删除${key}`)
          return result
        }
      })
      
      return observed // 返回代理对象
    }
  ```
  
  利用observe函数对数据进行测试：
  ```javascript
    const person = observe({ name: 'yuzheng' })
    person.name // yuzheng
    person.age = 28 
    person.gender = 'Male'
    delete person.gender
  ```
  
  对于其他情况，这里就不做测试了：
  
    * 嵌套对象的监听劫持
    * 监听数组的行为
    
  Proxy兼容IE9以上，关于Proxy的具体用法可参考[阮一峰 ECMAScript 6 (ES6) 标准入门教程 第三版](https://www.bookstack.cn/read/es6-3rd/spilt.1.docs-proxy.md)
  
  
  ### 参考文献：
  [阮一峰 ECMAScript 6 (ES6) 标准入门教程 第三版](https://www.bookstack.cn/read/es6-3rd/spilt.1.docs-proxy.md)
  
