# vue开发记录

vue文件里的样式、JS怎么分享独立文件

VUE ELEMENTUI怎么做表单验证

vue界面怎么传值，返回怎么回到之前列表

element :model是什么意思from model

vue的data，我动态添加一个，能不能双向绑定？

vue怎么在模板变量里直接调用一个方法

jwt

elementui table表格里怎么自定义列

element 弹出框只有一个文本和按钮，最简单的方式怎么做
 this.$prompt('请输入邮箱', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/,
          inputErrorMessage: '邮箱格式不正确'
        }).then(({ value }) => {
          this.$message({
            type: 'success',
            message: '你的邮箱是: ' + value
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          });       

@click里面(e)=>{可以写方法主体，在主体里要引用全局的方法需要用e.view指向外面,e.view.alert()}

vue中的this.$options.data()可以拿到原始空数据

list={ page:1, pagesize:10, data[], total:0 }