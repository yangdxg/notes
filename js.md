### var、let、const区别

#### var

var声明变量,这个变量属于当前的函数作用域,如果声明是在函数外的顶层,这个变量就属于全局作用域,默认值是undefined

```
var a =1;//变量a为全局变量
function foo(){
	var a = 2;//a为函数foo的局部变量
}
```

如果函数内省略var直接写a=2,a就是全局变量

var存在提升,只要下面定义了,上面使用也不会报错,为默认值undefined

#### let

- let的变量具有块级作用域的特征

- 在同一块级作用域,不能重复声明变量

- 不存在提升

  ```
      for (var i = 0; i < 10; i++) {
          setTimeout(function(){
              console.log(i);
          },100)
      };
      打印出10个10
  ```

  ```
      for (let i = 0; i < 10; i++) {
          setTimeout(function(){
              console.log(i);
          },100)
      };
      打印出0-9
  ```

#### const

除了let的上述特点外,还有一特点,不能修改为final

