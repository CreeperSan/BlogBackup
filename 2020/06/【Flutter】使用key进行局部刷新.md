# 【Flutter】使用key进行局部刷新

### 前言

总所周知，在Flutter中数据和视图绑定在一起，需要更新数据的话可以使用 `setState()` 方法进行更新，十分方便。但是如果使用不当，`setState` 刷新时可能会全局刷新，不仅会带来性能问题，还可能会引发一些其他的问题。比如下面的例子

![](img/01.gif)

### 问题原因

此界面的代码如下

```dart
class TestPage extends StatefulWidget{
  String _inputText = '';

  @override
  State<StatefulWidget> createState() {
    return CallDownSMSTemplateState();
  }

}

class CallDownSMSTemplateState extends State<TestPage>{

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(0xFFF2F5F7),
      appBar: AppBar(
        title: Text('局部更新'),
      ),
      body: Column(
        children: <Widget>[
          Text(_getDisplayString()),
          TextField(
            controller: TextEditingController(
              text: widget._inputText
            ),
            onChanged: (value){
              widget._inputText = value;
              setState(() {});
            },
          ),
        ],
      ),
    );
  }

  String _getDisplayString(){
    return '你输入的文本是：${widget._inputText}';
  }

}
```

逻辑比较简单，顶部的文本实时更新显示输入框的文本。当输入框输入的内容发生变化的时候，通过 `setState` 方法刷新界面。

但是我们可以看到，当输入框内容发生变化的时候，顶部的文本虽然有发生变化，但是输入框的光标也自动跑到了最前面。

究其原因，就是在调用 `setState` 时，连文本输入框也进行了刷新。光标的位置因此也发生了改变

因此，进行局部刷新或许是更好的实现方法。

### 解决方法

要实现局部刷新，可以利用 `GlobalKey` 进行改造，我们把需要更新的 `Text` 取出来，因为 `GlobalKey` 需要使用到 `State` ，而 `Text` 是继承自 `StatelessWidget` ，没有对应的 `State` 。

改造完成后代码如下

```dart
class TestPage extends StatefulWidget{
  String _inputText = '';

  @override
  State<StatefulWidget> createState() {
    return CallDownSMSTemplateState();
  }

}

class CallDownSMSTemplateState extends State<TestPage>{
  GlobalKey<StatefulTextState> key = GlobalKey();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(0xFFF2F5F7),
      appBar: AppBar(
        title: Text('局部更新'),
      ),
      body: Column(
        children: <Widget>[
          StatefulText(key),
          TextField(
            controller: TextEditingController(
              text: widget._inputText
            ),
            onChanged: (value){
              widget._inputText = value;
              key.currentState.setText(_getDisplayString());
            },
          ),
        ],
      ),
    );
  }

  String _getDisplayString(){
    return '你输入的文本是：${widget._inputText}';
  }

}

class StatefulText extends StatefulWidget{

  StatefulText(Key key):super(key: key);

  @override
  State<StatefulWidget> createState() {
    return StatefulTextState();
  }

}

class StatefulTextState extends State<StatefulText>{
  String _text = '';

  @override
  Widget build(BuildContext context) {
    return Text(_text);
  }

  void setText(String text){
    setState(() {
      _text = text;
    });
  }

}
```

运行可以看出，输入框光标位置也没有出现重置的情况

![](img/02.gif)

### 参考资料

[Flutter性能优化之局部刷新 - 刺客的幻影](https://www.jianshu.com/p/23a2e8a96a79)