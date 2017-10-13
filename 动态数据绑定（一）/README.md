## 动态数据绑定（一）

[这里是任务描述](./task.md)
***
监听对象属性的变化，有两种方式：

* **采用ES5中的defineProperty，设置get和set函数，重新定义读取和赋值的方式**。

* **采用ES6中的proxy对目标对象进行"拦截"**。

####方式一：ES5的defineProperty
Object.defineProperty(obj, prop, descriptor)定义对象的属性。

Object.defineProperty可设置的属性如下：

**configurable**：能否使用delete、能否修改属性特性、能否修改访问器属性，false为不可重新定义。
//一般情况下，configurable是设置为true，如果设置为false，这个属性除了writable之外都不能修改了，writable也只能从true改为false而不能反过来。

**enumerable**：对象属性是否可通过for-in循环遍历或在Object.keys中列举。

**writable**：对象属性是否可修改，false为不可修改。

**value**：对象属性的默认值，默认值为undefined。

注意： configurable， enumerable， writable特性默认值根据对象定义方法不同而不同。

    // 直接在对象上定义属性，这些特性默认值为true
    var obj = {};
    obj.name = 'season';
    console.log(Object.getOwnPropertyDescriptor(obj, 'name'));
    // Object {value: "season", writable: true, enumerable: true, configurable: true}

    // 调用Object.defineProperty()方法，不指定值的时候，默认为false
    var obj = {};
    obj.defineProperty(obj, 'name', {
      value: 'season'
    });
    console.log(Object.getOwnPropertyDescriptor(obj, 'name'))
    // Object {value: "season", writable: false, enumerable: false, configurable: false}

    // 在本例中，可以定义configurable、enumerable，默认为false。 但是如果定义了set或get方法中的任何一个，就不能再设置writable，即使false也不可以。


接着获取对象属性：

* 使用Object.keys(obj)，该方法返回一个数组，数组里是obj可被枚举的所有属性，接着对数组进行forEach遍历。
* 使用for in获取所有属性，接着用obj.hasOwnProperty(key)对属性进行判断过滤。

接着就可以自定义get和set函数啦！

在这过程中犯了一个低级错误：
get函数return data[key]，导致get函数返回时又触发了get函数，陷入死循环。

最终代码：

    function Observer(data) {
      this.data = data;
      this.makeObserver(data);
    }

    Observer.prototype.makeObserver = function(data) {
      for(var i in data) {
        if(data.hasOwnProperty(i)) {
          this.getset(i, data[i]);
        }
      }
    }

    Observer.prototype.getset = function(i, value) {
      let val = value;
      let that = this;
      Object.defineProperty(this.data, i, {
        configurable: true,
        enumerable: true,
        get: function () {
          console.log('你访问了' + i);
          return val;
        },
        set: function (newval) {
          console.log('你设置了' + i + ',新的值为' + newval);
          val = newval;
        }
      })
    }

####方式二：ES6的拦截器proxy
> Proxy可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

最终代码：

    function Observer (data) {
      return new Proxy(data, {
        get: function (target, propKey) {
          if (propKey in target) {
            console.log('你访问了' + propKey);
            return target[propKey];
          } else {
            console.log("Property \"" + propKey + "\" does not exist.");
          }
        },
        set: function (target, propKey, value) {
          console.log('你设置了' + propKey + ',新的值为' + value);
          target[propKey] = value;
        }
      })
    }
