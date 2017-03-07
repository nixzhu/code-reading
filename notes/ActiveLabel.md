# ActiveLabel

- 项目地址：[ActiveLabel.swift](https://github.com/optonaut/ActiveLabel.swift)
- 项目规模：小型
- 阅读难度系数：3/5

## 功能描述

一个可替换 UILabel 的控件，支持检测和点击 Hashtag、Mention 以及 URL，并且支持利用正则表达式自定义识别模式。另外，支持只高亮某些类型，且可缩短显示URL（例如太长会显示为`https://github.com/nix...`）。

## 实现分析

`@IBDesignable open class ActiveLabel: UILabel `，ActiveLabel 是 UILabel 的子类，且支持在 Storyboard 中定制。因此还有许多 IBInspectable 的属性，如`@IBInspectable open var mentionColor: UIColor`，不表。

因为是 UILabel 的子类，因此重载了如`text`、`font`等许多属性，并在设置后`updateTextStorage()`

### 组织架构

- 主类 ActiveLabel
- enum ActiveType 定义可检测的类型；内部使用 enum ActiveElement 关联类型对应的字符串；注意 ActiveType: Hashable 的实现，已定义的case用负数
- struct ActiveBuilder 根据解析的信息，生成 ElementTuple 等，用于后续使用，值得一读
- struct RegexParser 预定义的正则表达式，缓存 NSRegularExpression，用于从文本中提取匹配的模式

### 初始化

``` swift
lazy var textStorage = NSTextStorage()
lazy var layoutManager = NSLayoutManager()
lazy var textContainer = NSTextContainer()
func setupLabel() {
    textStorage.addLayoutManager(layoutManager)
    layoutManager.addTextContainer(textContainer)
    textContainer.lineFragmentPadding = 0
    textContainer.lineBreakMode = lineBreakMode
    textContainer.maximumNumberOfLines = numberOfLines
    isUserInteractionEnabled = true
}
```

基本的 TextKit 用法，不表。

### 文字高亮

``` swift
lazy var activeElements = [ActiveType: [ElementTuple]]()
func updateTextStorage(parseText: Bool = true) {
    // 取到 attributedText 并根据设置（如 textAlignment、lineSpacing 等）`addLineBreak()`
    // 如果必要，`parseTextAndExtractActiveElements()`：根据需要，检测必要的类型，构造 activeElements（区别处理URL）
    // 添加链接属性，`addLinkAttribute()`：先设置基本字体和颜色，然后根据 activeElements 的信息，设置对应位置的文字的字体和颜色。
    // 将属性字符串存储到 textStorage，设置 text 并呼叫重新绘制
}
```

### 点击检测

因为 isUserInteractionEnabled 为 true，所以直接重载了 touchesBegan、touchesMoved、touchesCancelled、touchesEnded，并调用 `onTouch()` 方法。

``` swift
func onTouch(_ touch: UITouch) -> Bool {
    // 获取点击位置
    // 用`element(at:)`或者对应位置的元素：利用 layoutManager 的 glyphIndex 找到对应位置的 index，然后比对 activeElements 中的位置信息即可
    // 如有必要，更新一下文字外观 updateAttributesWhenSelected，记录被点中的元素 selectedElement
    // 在触摸结束时，判断 selectedElement，调用对应的方法，然后调用delegate方法，事件就能被用户处理了
}
```

## 小结

通过阅读此项目代码，可以了解到如何子类化 UILabel、操作属性字符串的一些实践、如何使用原始触摸方法、以及正则表达式的使用等。