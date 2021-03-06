# 你不知道的JavaScript
* 作用域和闭包 
   * 作用域是什么
      * 编译原理 
      * 作用域
   * 词法作用域
   * 函数作用域和块作用域
   * 提升
   * 作用域闭包		
* this和对象原型
> 作用域 负责收集并维护由所有声明的标识符（变量）组成的一系列的查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限
<p>遍历嵌套作用域链的规则：引擎从当前的执行作用域开始查找变量，如果找不到，就向上一级继续查找，当到达最外层的全局作用域时，不管找没找到，都结束查找</p>
> 编译器 负责语法分析和代码生成

> 引擎 负责整个JavaScript程序的编译和执行过程
<p>LHS和RHS 含义是“赋值操作的左侧或者右侧”，具体的辨别可以参见以下代码</p>
<pre>function foo(a){
    var b = a;
    return a + b;
    }
    var c = foo(2);
    </pre>
    <p>分析：执行`c = foo(a)`时，需要找到foo的源头，令b = a时需要找到a的源头，return a + b时需要分别找到a 和 b的源头，一共4次RHS</p>
    <p>不成功的RHS引用会导致抛出ReferenceError异常，不成功的LHS引用会导致自动隐式地创建一个全局变量（非严格模式），该变量使用LHS引用的目标作为标识符，或者在严格模式下抛出ReferenceError</p>
   
  **词法作用域**就是定义在词法阶段的作用域，换句话说，词法作用域就是由你在写代码是将变量和块作用域写在哪里来决定的
  <pre>function foo(a){
  var b = a*2;  
  function bar(c){
  console.log(a,b,c);
  }
  bar(b*3);
  }
  foo(2);//2,4,12
  </pre>
  <p>以上代码中有三个逐级嵌套的作用域，最里面的bar()的作用域（只有一个标识符c），以及外层的foo()作用域（包含着a,b,以及bar标识符），还有最外层的全局作用域（只有一个foo标识符）</p>
查找:作用域旗袍的结构和互相之间的位置关系给引擎提供了足够的位置信息，引擎可以借助这些信息来查找标识符的位置。
遮蔽效应：作用域查找会在找到第一个匹配的标识符时停止，在多层嵌套的作用域中可以定义同名的标识符，反正内部的总会遮蔽掉外部的，就近找到了要查找的标识符引擎就不会再向外部索取。被这种效应遮蔽的全局变量还是可以借助window.a（此处假设a是要查找的全局变量）找到，但是如果被遮蔽的不是全局变量，如下：
  <pre>
  var a = 2;
  function foo(){
      var a = 4;
      function bar(){
          var a = 10;
          console.log(a);//10
          console.log(window.a);//2
      }
  }
  </pre>
  以上代码中值为4的a，你是找不着滴~~~~~~~~~~~~~ 
<h2>欺骗词法</h2>
 <strong>eval()</strong>
  eval()函数可以接受一个字符串作为参数，并将其中的内容视为好像在书写时就存在于程序中这个位置的代码，它修改词法作用域环境的范例如下：
  <pre>
  function foo(str,a){
    eval(str);//欺骗！
    console.log(a,b);
    }
    var b = 10;
    foo("var b = 12",1);//运行结果为1，12，很明显程序被eval这个小婊砸欺骗了，明明我们要的是10
 </pre>
 <p>总之呢，尽量避免使用eval吧</p>
 <strong>with</strong>
  <p>先看两段代码</p>
  <pre>
  var obj = {
  a:1,
  b:2,
  c:3
  };
  //重新赋值
  with(obj){
  a=3;
  b=4;
  c=5;
  }
  </pre>
  <pre>
  function foo(obj){
  with(obj){
    a=2;
      }
    }
    var o1 = {
      a:3
     };
      var o2 = {
      b:3
     };
     foo(o1);
     console.log(o1.a);
     foo(o2);
     console.log(o2.a);//undefined,本来o2就没有a这个属性
     console.log(a);//2，此时a=2被泄漏到全局作用域上了
     </pre>
     <p>o2的作用域、foo()的作用域和全局作用域中都找不到标识符a，因此当a=2执行时，自动创建了一个全局变量。with声明实际上是根据你传递给它的对象凭空创建了一个全新的词法作用域</p>
     <p><strong>总之不要使用eval 和 with</strong></p>
  
  
